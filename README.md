# ETL with NiFi

NiFi is a data pipeline and transformation tool from the Apache Software Foundation. NiFi supports data pipelines and transformation on the data flowing through the pipeline. It is good at handling pipelines where conversions are needed (e.g. text -> JSON, Kafka -> ElasticSearch). It has a user-friendly interface. It is modeled around processors (think of a processor as a transformation). It is scalable, and for horizontal scaling, you can simply add machines. 
Pro's:
- It can transfer data or map to different systems
- It can deliver data to analytic platforms
- Ability to enrich and prepare for subsequent steps

Cons:
- Not a distributed computation platform (i.e. not ideal for high performance computing)
- It is not for complex event processing
- It is not for doing aggregate operations on large volumes of data (a database may be more ideal)

In Summary, NiFi is great for ETL but not for aggregating data. 

## Platform Concepts
FlowFile
- It is the data itself
- Contains attributes are associated with the data, such as key-value pairs

Processors
- Think of the processor as the trasnformation
- It applies transformations to FlowFiles
- Every transformation creates a new output
- The processor outputs feeds into the next processor input 

Connector
- Can be considered as a queue for the FlowFiles waiting to be processed by a processor
- Can have prioirty rules (i.e. FIFO, LIFO, etc)
- Can have back pressure. i.e. can limit it to 10k FlowFiles and stop the queue until more of them can be processed

## NiFi System Architecture
There will be two containers for this system:
1. MySQL Source - This will be a docker container with a MySQL database
2. NiFi - This will be a container that holds the extract, transform, and load functions.  
Both of these containers will connected via a docker network. 

# Set Up
This project will follow the following workflow as an example. 
## Install NiFi in a Docker container
1. Create a network for two containers (MySQL and NiFi containers):
``` 
docker network create netable
```
2. Create container for NiFi
```
docker run --name nifiable -p 8080:8080 --network netable -d apache/nifi:1.13.2
```
To confirm, navigate to the following URL after creation. This should take you to the NiFi user interface. 
``` 
http://localhost:8080/nifi
```
Of mention, 8080 is a frequently used port. Be aware of port conflicts. Also note that the image for NiFi is rather large and may take some time to download, depending on the internet connection. 
3. Create MySQL Docker container
```
docker run -p 33061:3306 --name mysqlabel -e MYSQL_ROOT_PASSWORD=MyNewPass --network netabel -d mysql:8.0
```

## Create and Initialize Database in a container
1. A sample database will be created, called Education, with a table, Students. Sample data will be inserted accordingly. To accomplish this, connect to the MySQL container via Workbench and execute the following command: 
### Create database
``` SQL
-- DATABASE EDUCATION
DROP DATABASE IF EXISTS `education`;
CREATE DATABASE IF NOT EXISTS `education`;
USE `education`;

-- TABLE STUDENTS

CREATE TABLE `students`
(
    `email` varchar(50)
    `name` varchar(50)
    `city` varchar(50)
    PRIMARY KEY (`email`)
);

-- POPULATE STUDENTS

INSERT INTO `students` VALUES('peter@mit.edu', 'peter parker', 'Cambridge,MA');
```
Confirm entry: 
``` SQL
USE education;

SELECT * FROM students;
```
## Install the driver in the container
In order for NiFi to access the MySQL container it will need a driver. Drivers can be downloaded from [dev.mysql.com](dev.mysql.com). In this case, the driver can be accessed from [this route](https://dev.mysql.com/downloads/connector/j/). Navigate the site accordingly:
- Select Operating System: Platform Independent
- Download .zip file (not the TAR archive)
Unzip the file and move into the appropriate host directory within the terminal window. 
At this point, the driver will need to be copied into the container. To do this, open another terminal window. 
1. Enter the NiFi container:
```
docker exec -it nifiable /bin/bash
```
alternatively, the container can be accessed with /bin/sh instead of /bin/bash
2. Within the NiFi container, navigate to the nifi folder (`cd /opt/nifi`)
3. Make a directory for the drivers and move into the directory:
```
mkdir drivers
```
then move into directory
```
cd drivers
```
At this point we have two terminal windows, one for the host and one for the NiFi container. Return to the host terminal in order to upload the driver to the nifi container with the following command:
```
docker cp ./mysql-connector-java-8.0.25.jar nifi:/opt/nifi/drivers/mysql-connector-java-8.0.25.jar
```
Confirm file transfer by returning the the nifi container within terminal and checking the contents of the /drivers folder that was created in the preceeding steps. 

## Use NiFi to create an ETL pipeline
Now, we bring it all together and work within the NiFi platform to create the ETL pipeline. 
### Create DB connection pull
1. Click on gear icon in the NiFi Flow operator box on the left. 
2. Select control services in the page that appears. 
3. Click on the + icon. 
4. Add a controller service by selecting the `DBCPConnectionPool` option
5. Configure connection by clicking on gear icon next to the controler. Name it `MySQL` and move to the properties tab. Change the connection URL to the db, `jdbc:mysql://mysqlabel:3306` (note that the path is the name of the container as defined in preceeding steps). Next, set the driver class name as `com.mysql.jdbc.Driver`. Next, set the Database Driver Location to the driver location within the Docker container, `/opt/nifi/drivers/mysql-connector-java-8.0.25.jar`. Next, set Database User as `root` and Passowrd as, `MyNewPass`. 
6. Returning to the Controller within the Controler Services menu, click the lightening bolt (right side of connector) to enable the connection. A menu will appear for Enable Controler Services, change the scope to `Service and referencing components`. Select enable and ensure that all checkmarks appear. Upon returning to the Controller selection, a blue lightening bolt with text saying `Enabled` will appear in the status section of the controller. 
7. Returning to the main window, begin adding processors. 
(1) Add `ExecuteSQL`
(2) Add `LogAttribute`
Connect both processors and select `Success` in the For Relationships selection
8. Configure SQL by selecting the ExecuteSQL processor and clicking the gear icon
(1) Automatically Terminate Relationships: `failure`
(2) In the Scheduling tab, run every 30 seconds
(3) In the Properties tab, set Database Connection Pooling Service to `MySQL` (name that was set in preceeding steps)
(4) Enter query in the SQL select query to `SELECT * FROM education.students`
9. Start processor by right click on the processor, then Start

# Additional Notes
1. In addition to pulling data from a database, there are processors for connecting to an excel file and transforming the data into other formats and returning the data as a single file format or into a DB. 
2. NiFi ETL pipelines can be created within various database containers, including Cassandra, Mongo, and Redis, 
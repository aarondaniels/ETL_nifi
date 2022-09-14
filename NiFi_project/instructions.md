# Load data into MySQL using NiFi
This project will use NiFi to perform a data load into a MySQL database. It will use real earthquake data from [USGS](https://earthquake.usgs.gov/earthquakes/search/) and storing it within a database. 

Before begining this project, establishe two containers, one for NiFi and one for MySQL (or database of choice) running within Docker and ensure they are connected within a common Docker network. See the repo [README](https://github.com/aarondaniels/ETL_nifi) for details on this step. 

## Steps for executing ETL project
### Configure database
With the MySQL database established in Docker, configure it using the following commands:
1. Create database
``` sql
CREATE DATABASE IF NOT EXISTS usgs;
USE usgs;
```
2. Generate table
``` sql
create TABLE earthquakes(
    idx int,
    time varchar(100),
    latitude varchar(100),
    longitude varchar(100),
    depth varchar(100),
    mag varchar(100),
    magType varchar(100),
    nst varchar(100),
    gap varchar(100),
    dmin varchar(100),
    rms varchar(100),
    net varchar(100),
    id varchar(100),
    updated varchar(100),
    place varchar(100),
    type varchar(100),
    horizontalError varchar(100),
    depthError varchar(100),
    magError varchar(100),
    magNst varchar(100),
    status varchar(100),
    locationSource varchar(100),
    magSource varchar(100)
);
```
3. Confirm table is empty
``` sql
SELECT * FROM earthquakes
```
### Establish source data
Place the USGS_data.csv file in the NiFi Server in a newly created directory with the path `/opt/nifi/current-nifi/data` via the NiFi terminal. 

Create a directory called current-nifi and create a data subdirectory within it. Open up a second terminal window on the local machine and navigate to the folder where theh USGS_data.csv file is located and execute the following command: 
```
docker cp ./USGS_data.csv nifi:/opt/nifi/current-nifi/data
```

### Establish workflow within NiFi
Open NiFi in a browser and establish a MySQL controller service (see README for details on establishing the MySQL controller service). Also ensure the NiFi driver is installed (See README). Two additional controller services will be established, a reader and a writer. The Reader will read the data from the FlowFiles and the writer will write the data to MySQL commands. 
1. Create a reader: Go to the controller services, select the '+' option and search for `CSVReader`. Open up the configurations and confirm that the screen appears as follows:
![create reader](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img1.png)
2. Createa  writer: Select teh '+' option again and search for `JsonRecordSetwriter`. Open the configurations and confirm that the screen looks as follows: 
![create writer](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img2.png)
3. Seelct the lightening bold symbol to start up each controller and select "Service Only" under the service and reference components menu
4. Set up the data pipeline. This will consist of 5 processors flowing in the order specified below: 
a) `GetFile` processor:
![Get File Configure](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img3.png)
![Get File Details](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img4.png)
b) `SplitText` processor:
![Split Text Configure](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img5.png)
![Split Text Details](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img6.png)
c) `ConvertRecord` processor: 
![Convert Record Configure](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img7.png)
![Convert Record Details](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img8.png)
d) `ConvertJSONToSQL` processor:
![JSON Convert Configure](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img9.png)
![JSON Convert Details](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img10.png)
e) `PutSQL` processor:
![Get File](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img11.png)
![Get File](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img12.png)

With all five processors configured, the connections must be made. There will be a total of 5 connectors. The first four connectors will connect the five processors and the last connector will be a loop connector over the last processor, `PutSQL`, to handle retries. Retries are necessary because databases have settings to assure the data is not being read and written at the same time. If the data table is locked when trying to perform a load, the processor must retry once the lock is removed. 
a) The connector between the `GetFile` and `SplitText` processors will be a `success` relationship:
![connector](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img13.png)
b) The connector between the `SplitText` and `ConvertRecord` processors will be a `splits` relationship:
![Get File](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img14.png)
c) The connector between the `ConvertRecord` and `ConvertJSONToSQL` processors will be a `success` relationship:
![Get File](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img15.png)
d) The connector between the `ConvertJSONToSQL` and `PutSQL` processors will bbe a `sql` relationship:
![Get File](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img16.png)
e) The final connector will stem from the `PutSQL` processor and loop back to itself. This will be a `retry` relationship:
![Get File](https://github.com/aarondaniels/ETL_nifi/blob/main/images/img17.png)

Start each processor, beginning with the `GetFile` processor. Observe teh data propogate down toward the `PutSQL` processor. 

Navigate back to MySQL Workbench and perform the following query:
``` sql
SELECT * FROM earthquakes;
```
There should now be rows of data that have been loaded into the table. 
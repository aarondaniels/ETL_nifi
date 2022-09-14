# Load data into MySQL using NiFi
This project will use NiFi to perform a data load into a MySQL database. It will use real earthquake data from [USGS](https://earthquake.usgs.gov/earthquakes/search/) and storing it within a database. 

Before begining this project, establishe two containers, one for NiFi and one for MySQL (or database of choice) running within Docker and ensure they are connected within a common Docker network. See the repo [README](https://github.com/aarondaniels/ETL_nifi) for details on this step. 


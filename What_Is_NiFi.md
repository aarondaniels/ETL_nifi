
# Apache NiFi
Apache NiFi is an open-source data pipeline and transformation software for automating and managing the flow of data between systems. The tool uses a web-based user interface (UI) to create and manage different data flows. It is highly customizable and allows users to configure and modify their data live at runtime.

## History of Apache NiFi

NiFi was initially developed at the National Security Agency (NSA) in 2006 and was used by the organization for ETL for eight years. In 2014, it was donated to the Apache Software Foundation and, in 2015, NiFi became an official part of the Apache Project Suite. Since then, NiFi has been continually updated and improved and every six to eight weeks, Apache NiFi releases a new update. When using NiFi, it is important to ensure that you are using the most recent version.

## NiFi Architecture

NiFi has a well-thought-out architecture. Once data is fetched from external sources or your local machine, it is represented as a FlowFile inside the NiFi architecture. Next, a flow controller must be created to configure your flow. At this stage, you will need to add different controllers depending on the type of database you are working with. After that, you must add a number of single-function components (in NiFi, these are called processors) that can be utilized to build out a customized NiFi data flow. Finally, in order to start the data flow to perform ETL, you must connect your processors using connectors.

[NiFi architecture]() ***insert image***

Key Components of NiFi Architecture

| Component | Definition |
|-----------|------------|
| Connector |A connector links processors together and defines the relationship between the processors, which controls how data flows. Connectors can also link one processor back to itself to create a loop.|
| Controller | A controller records connections between processes and controls the allocation of threads used by all processes. |
| Event | Events represent the change to a FlowFile while traversing through a NiFi flow. These events are tracked in NiFi repositories. |
| Flow | The NiFi flow is a process that begins once all the processors are connected and running. During this process, the data is updated, inserted, or deleted in a database. |
| FlowFile | A FlowFile is “original data with meta-information attached to it. It allows you to process not only CSV or other record-based data but also pictures, videos, audio, or any other binary data”.|
| Process Group | A process group is composed of all of the processors and their related connectors, and it includes the exchange of data that occurs between these entities through the use of ports. |
| Processor | A processor is “the NiFi component that is responsible for creating, sending, receiving, transforming, routing, splitting, merging, and processing FlowFiles. It is the most important building block available to NiFi users to build their data flows”|
| Server | As you learned in earlier modules, web servers are network-connected computers that serve or send information to client computers on the network. The NiFi user interface (UI) is hosted on a web server that “hosts NiFi’s HTTP-based commands and API”. |

There are a lot of different components to NiFi architecture. To learn more about the key components of NiFi architecture, reference the [Apache NiFi Overview](https://nifi.apache.org/docs/nifi-docs/html/overview.html).

## Why Use Apache NiFi?

There are a number of reasons to use Apache NiFi. NiFi allows you to import data into NiFi from numerous data sources and create FlowFiles to integrate that data into your data flow process. NiFi is user-friendly and customizable; you can edit your data in real time, manage your data flows, and integrate NiFi into your existing infrastructure. NiFi is scalable; you can scale out data in clusters and manage different data flows either separately or as a whole. The NiFi UI “allows users to visualize and monitor performance behavior in a flow bulletin that offers insight and inline documentation”. NiFi also allows users to manage their data flows using existing libraries, Java code, and a number of processors with the ability to listen, fetch, split, aggregate, route, transform, and drag & drop data.

NiFi is a popular tool because it is equipped with a number of useful features. Key features of NiFi are described in the following table:

### Features of NiFi

| Feature | Description |
|---------|-------------|
| Continuous Improvement | NiFi is updated frequently, which ensures that users always have access to the most current and effective system. NiFi also runs quickly to allow for fast processing, testing, and communication of data. |
| Customizable | NiFi gives the power to the user to create a data flow that works best for them. Users can set up the processors, connectors, and controllers that they need to build an effective flow for their data. NiFi uses single-function components that can be built and added on to make data flows as simple or complex as the user needs them to be. |
| Role-Based Security Features | NiFi uses role-based authentication and authorization protocols to ensure the security of data (Taylor 2021). Data also flows through secure methods that encrypt communication to prevent data insecurity. |
| Support | NiFi offers helpful online guides and technical support for users to help with troubleshooting issues and optimizing data flows. |
| User-Friendly | NiFi uses the Java programming language, which is commonly known and allows for easy transfer of data from any device into the NiFi system. NiFi is also easily accessed, even from places with limited connectivity. |

For more information about NiFi, visit the [official NiFi Overview](https://nifi.apache.org/docs.html) website.
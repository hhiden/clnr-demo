# clnr-demo
Initial demo of a data streaming application using the CLNR dataset.

This demo will show a number of containers processing data streams of smart meter readings. The data was generated by the [Customer Led Network Revoultion](http://www.networkrevolution.co.uk/project-data-download/?dl=TC1a.zip#) project. Initially the demo will use the TC1a dataset which has domestic smart meter readings on 30 minute intervals.

## Persistent Producer

The persistent producer module will store expects the messages sent to it to be of type 
Reading. This represents one sample of smart meter containing the customerId, timestamp 
and kWh reading since the last sample.

In the current version the persistent producer also sends the kafka messages once the 
message has been persisted. This will be replaced by Debezium in a future version of the demo.

In order to run this producer you need to deploy a MySQL application into your project.
This can be achieved with the following command

```bash
oc new-app --template=mysql-persistent \
       -p MYSQL_USER=mysql \
       -p MYSQL_PASSWORD=mysql \
       -p MYSQL_DATABASE=reading
```  

Once the MySQL container has started the persistent producer can be deployed using the 
command `mvn clean package fabric8:deploy`. 

There is a script which will send sample data to the application. Set the `PRODUCER_URL` 
environment variable to point to your persistent-producer application and run the `start.sh` script.

```bash
$ oc get routes
NAME                  HOST/PORT                                             PATH      SERVICES              PORT      TERMINATION   WILDCARD
consumer              consumer-hardcoded-test.127.0.0.1.nip.io                        consumer              8080                    None
persistent-producer   persistent-producer-hardcoded-test.127.0.0.1.nip.io             persistent-producer   8080                    None

$ export PRODUCER_URL='persistent-producer-hardcoded-test.127.0.0.1.nip.io'
$ ./start.sh
Sending data to persistent-producer-hardcoded-test.127.0.0.1.nip.io
120,Electricity supply meter,Consumption in period [kWh],03/12/2011 00:00:00,0.067
120,Electricity supply meter,Consumption in period [kWh],03/12/2011 00:30:00,0.067
120,Electricity supply meter,Consumption in period [kWh],03/12/2011 01:00:00,0.066
120,Electricity supply meter,Consumption in period [kWh],03/12/2011 01:30:00,0.066
120,Electricity supply meter,Consumption in period [kWh],03/12/2011 02:00:00,0.039
120,Electricity supply meter,Consumption in period [kWh],03/12/2011 02:30:00,0.028
...
^C
$
```

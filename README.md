# Time Series Data with the IntelⓇ Edge Insights Platform

## Introduction

The Intel Edge Insights Software can be configured to not only take in video information but also to take in sensory information and other time series based related data series.

Information in this configuration is ingested with Telegraf, stored within the Influxdb database, alerts and monitors can be set using Kapacitor, and visualized with Chrongraf.

In this lab, we will configure the Intel insights platform for point data and explore the capabilities that it offers.

## Intel Edge Insights Software Components Overview

The edge insights platform from Intel is a framework for rapidly deploying video analytics solutions. It provides a microservice architecture that distributes the tasks of video intake, filtering, analyzing with deep learning and computer vision, and performing automated actions, alerts, and intelligent monitoring.

The edge insights platform provides triggers/filters and classifiers that can be customized by an organization to fit their custom needs.

The edge insights platform supports both a developer mode and a production mode. The developer mode disables certificate security and allows the developer to concentrate on the functionality of their application without deploying cryptographic certificates.

Let's go through a description of the components of this microarchitecture

Multiple microservices coordinate to provide the overall service of the edge insights platform. a number of these are open source projects such as telegraf, influxdb, chronograf, Kapacitor (collectively called the TICK stack) and Vault, a service that keeps cryptographic secrets in a secure manner.

Here's a list of the microservices:

The data agent
Data analytics
Data bus abstraction
Data ingestion library
The image store
The stream manager
Stream sub library
Telegraph
Video ingestion
Secret storage
Algorithms including triggers / filters and classifiers

## Prerequisites

This article was written for the Intel Edge Insights Software version 1.5. The software is on a quarterly release cadence.

## Setup instructions

Setting the Intel insights platform to process pointed data. In this configuration we will disable the video data ingestion, which can cause a significant load on the CPU.

In order to accomplish this we will do the following things:

- Set a couple of helpful environmental variables so that we don't need to continually re-type directory names
- Shut down the Intel insights platform
- Modify configuration files
- Repackage it
- Relaunch the Intel insights platform
- Setup Chronograf to visualize the data

Let's get started.

## Environmental Variables

Let’s set a variable for the top level of the EIS installations. If you unzipped the software in a different location, be sure to change the variable below.

`export EIS_HOME=/home/eis/Workshop/IEdgeInsights-v1.5LTS`

## Shut down the Intel Edge Insights Software service

The **docker_setup** directory is the location for building a that you should be in if you're going to rebuild the edge insights platform, modify configuration files, or manually start and stop the system.

If Edge in sizes running, the first step is to stop all docker containers related to it.

`cd $EIS_HOME/docker_setup`

`sudo make down`

## Modify the Configuration Files

In order to configure the Intel Edge insights platform for sensor ingestion in point beta mode. We are going to modify the `$EIS_HOME/docker_setup/.env` file.

```
gedit $EIS_HOME/docker_setup/.env
```

Inside the .env file, find the `IEI_SERVICES` variable and set it to `services_all.json`

`IEI_SERVICES=services_all.json`

If you open the `config/services_all.json` file you will notice that two services are specified by name and by docker image.

The first service is telegraf, which is a service with a flexible plug-in architecture for ingesting data and outputting it to Influxdb.

The second service is the data analytics service, which contains the Intel Edge insights datapoint analytics software as well as Kapacitor from the TICK stack. Kapacitor is a monitoring and alerting solution that integrates with multiple reporting out options such as email, executing a script, syslog, Apache Kafka, MQTT, SNMPTrap, Slack Messaging and others. See the documentation for Kapacitor for a complete list.

### Rebuild EIS

`cd $EIS_HOME/docker_setup`

`sudo make build run`

Now the Intel Edge Insights Platform is running in point data mode.

## Tips for working with Docker containers

Getting a Bash Shell Inside a Container
To list all of the running Docker containers type:
`sudo docker ps`

The rightmost column will have a list of all the names of the containers. You’ll need the name of the container for many operations including running a Bash shell in the container.

To get a bash shell inside the ia_telegraf docker container simply run the command.
`sudo docker exec -it ia_telegraf /bin/bash`

Likewise, to get a bash shell inside the ia_data_analytics docker container simply run the command.
`sudo docker exec -it ia_data_analytics /bin/bash`

When you are inside a docker container often running a simple process snapshot command will give a lot of information about the purpose of the particular container. Be sure that you are in a container when running this command.

`ps aux`

# Run a Time Series Workflow

## Configuring Telegraf for PointData Ingestion

The configuration for point data ingestion can be found at: `$EIS_HOME/docker_setup/config/telegraf.conf`. Lets take a look at that file now. Open up the telegraf.conf file and find the section for input MQTT - Note: This is a large file and the needed section is around line 1200, use ctrl-f to search for "[[inputs.mqtt_consumer]]".

```bash
gedit $EIS_HOME/docker_setup/config/telegraf.conf
```

```
[[inputs.mqtt_consumer]]
	servers = ["tcp://$HOST_IP:1883"]
	topics = [
 	"temperature/simulated/0",
 	]
	name_override = "point_data"
	data_format = "json"
   persistent_session = false
   client_id = ""
```

In the following lab we will want to take CPU temperture measurments as part of the dashboard creation. Lets set up Telegraf to do that. Search for "[[inputs.cpu]]" and uncomment the following lines:

```
# Read metrics about cpu usage
 [[inputs.cpu]]
  ## Whether to report per-cpu stats or not
  percpu = true
  ## Whether to report total system cpu stats or not
  totalcpu = true
  ## If true, collect raw CPU time metrics.
  collect_cpu_time = false
  ## If true, compute and report the sum of all non-idle CPU states.
  report_active = false
```

Save the changes.


Inside the dockerfile.yml, The MQTT port is exposed to the host system. This means that with telegraph configured to receive input on the same port that any MQTT client which is sending traffic to the host system can be received and fed into the Intel Edge Insight software.

### Rebuild EIS

Since we changed the telegraf configuration we need to rebuild ouor images. 

```
cd $EIS_HOME/docker_setup/
sudo make build run
```

## Check the docker logs

Uses command to check the logs of the docker telegraph container - You can use this method to troubleshoot issues with data transport.
`sudo docker logs ia_telegraf`

## Start the Simulated Sensor Data

Once Intel EIS is rebuilt and running, go to the \$EIS_HOME/tools/mqtt-temp-sensor directory. From here you can launch MQTT broker as well as a publisher and subscriber.

`$ cd $EIS_HOME/tools/mqtt-temp-sensor`

Build the docker container

`sudo ./build.sh`

Run the broker

`sudo ./broker.sh`

**Open New Terminal**
Run the publisher

```
export EIS_HOME=/home/eis/Workshop/IEdgeInsights-v1.5LTS
cd $EIS_HOME/tools/mqtt-temp-sensor
sudo ./publisher.sh
```

If you wish to see the messages going over MQTT, run the subscriber with the following command:
**Open New Terminal**

```

export EIS_HOME=/home/eis/Workshop/IEdgeInsights-v1.5LTS
cd $EIS_HOME/tools/mqtt-temp-sensor
sudo ./subscriber.sh
```

## Verify the ia_data_agent is Ingesting and Publishing Data

`sudo docker logs -f ia_data_agent`

Below is a snapshot of sample output of the above command.

```
I0608 02:25:57.3029569 StreamManager.go:159] Publishing topic: point_classifier_results
I0608 02:25:58.3029139 StreamManager.go:159] Publishing topic: point_classifier_results
I0608 02:25:59.3025279 StreamManager.go:159] Publishing topic: point_classifier_results
```

## More Information about the DataAgent

Technologies and Components Used
The data agent uses a number of technologies directly and it also bundles several projects as sidecars into its docker image.

Toml - Tom's obvious, minimal language is a specification that is often used for configuration files. This language is used for the configuration files in the TICK stack.
GRPC - a high-performance, open source universal RPC framework

### Security
When the data agent begins it starts by reading the command-line arguments and checking if the program is running on a genuine Intel system.

If it is running on a genuine Intel system it checks to see if the trusted platform module can be enabled
If then checks to see if it is running in developer mode or production mode.

Next it checks to make sure that vault is running and if it is not running the data agent will exit.

It then reads the certificates for influxdb from vault and writes out the SSL secrets that the grpc internal client will use.

The certificate authority certificate is written to /etc/ssl/ca/ca_certificate.pem
Service Initialization

After the security initialization is performed the data agent then launches influxdb and the stream manager.
The data agent and then takes UDP data from influxdb and forward it to the stream manager
The key for each stream is used as the topic to publish to the stream server.

## Exploring the InfluxDB

DescriptionInfluxDB is an open-source time series database developed by InfluxData. It is written in Go and optimized for fast, high-availability storage and retrieval of time series data in fields such as operations monitoring, application metrics, Internet of Things sensor data, and real-time analytics. 

Let's use the InfluxDB command line client to connect to the InfluxDB.

First we need to install the InfluxDB client:

```
sudo apt install influxdb-client
```
Next connect to InfluxDB:

```
influx -username admin -password admin123 -ssl -unsafeSsl
```

### Let's explore the database with some basic commands.

Let's start by listing all the databases.
```
show databases;
```

Use the "datain" database 
```
use datain
```

Next let's view the measurement tables that are being recorded. This is the data that the Intel Edge Insights Software is recording.

```
show measurements
```

Now, we can view the data from the point_classifier. 
```
select * from point_data;
```

Now run through the same commands with the *point_classifier_results* measurements table.

## Configure Kapacitor to Send Alerts

Kapacitor is able to handle alerts using two different methods: Push to Handler and Publish and Subscribe

Both approaches need handlers to be enabled and configured and information such as tokens and passwords need to be stored securely. 

### Push to handler
TICKscript handlers are relevant functions that can be called to handle data and push it to third parties. For example, you can call the log function, the email function, the HTTP out function or a user-defined third party function.

### Publish and Subscribe
Alert topics are similar to topics in MQTT. A topic is a string that groups together alerts. Multiple handlers can subscribe to a topics and be executed for any message published to the topic. Handlers are bound to topics either on the command line or in a YAML or JSON file.

For example: 
```
topic: cpu
id: slack
kind: slack
options:
   channel: '#kapacitor'
```

Save this to a file named slack_cpu_handler.yaml

```
kapacitor define-topic-handler slack_cpu_handler.yaml

```

### List of Handlers

The following handlers are currently supported:

Alerta: Sending alerts to Alerta.

Email: To send alerts by email.

HipChat: Sending alerts to the HipChat service.

Kafka: Sending alerts to an Apache Kafka cluster.

MQTT: Publishing alerts to an MQTT broker.

OpsGenie: Sending alerts to the OpsGenie service.

PagerDuty: Sending alerts to the PagerDuty service.

Pushover: Sending alerts to the Pushover service.

Sensu: Sending alerts to Sensu.

Slack: Sending alerts to Slack.

SNMP Trap: Posting to SNMP traps.

Talk: Sending alerts to the Talk service.

Telegram: Sending alerts to Telegram.

VictorOps: Sending alerts to the VictorOps service.

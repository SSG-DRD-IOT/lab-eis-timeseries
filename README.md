# Time Series Data with the IntelⓇ Edge Insights Platform

## Introduction

The Intel Edge Insights Software can be configured to not only take in video information but also to take in sensory information and other time series based related data series.

Information in this configuration is ingested with Telegraf, stored within the Influxdb database, alerts and monitors can be set using Kapacitor, and visualized with Chrongraf.

In this lab, we will configure the Intel insights platform for point data and explore the capabilities that it offers.

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

`export EIS_HOME=/home/eis/labs/IEdgeInsights`

## Shut down the Intel Edge Insights Software service

The **docker_setup** directory is the location for building a that you should be in if you're going to rebuild the edge insights platform, modify configuration files, or manually start and stop the system.

If Edge in sizes running, the first step is to stop all docker containers related to it.

`cd $EIS_HOME/docker_setup`

`sudo make down`

## Modify the Configuration Files

In order to configure the Intel Edge insights platform for sensor ingestion in point beta mode. We are going to modify the **\$EIS_HOME/docker_setup/.env** file.

Inside the .env file, find the IEI_SERVICES variable and set it to **services_pointdata.json**

**\$EIS_HOME/docker_setup/.env** - IEI_SERVICES=services_pointdata.json

If you open the **config/services_pointdata.json** file you will notice that two services are specified by name and by docker image.

The first service is telegraf, which is a service with a flexible plug-in architecture for ingesting data and outputting it to Influxdb.

The second service is the data analytics service, which contains the Intel Edge insights datapoint analytics software as well as Kapacitor from the TICK stack. Kapacitor is a monitoring and alerting solution that integrates with multiple reporting out options such as email, executing a script, syslog, Apache Kafka, MQTT, SNMPTrap, Slack Messaging and others. See the documentation for Kapacitor for a complete list.

## Modify the IEI_SERVICES environmental variable

First go into the docker_setup up directory and set the IEI_SERVICES variable to services_pointdata.json.

This is an example file for configuring time series data ingestion.

`cd $EIS_HOME/docker_setup`

`sudo make build run`

Now the Intel Edge Insights Platform is running in point data mode.

## Tips for TroubleShooting

Getting a Bash Shell Inside a Container
To list all of the running Docker containers type:
`sudo docker ps`

The rightmost column will have a list of all the names of the containers. You’ll need the name of the container for many operations including running a Bash shell in the container.

To get a bash shell inside the ia_telegraf docker container simply run the command.
`sudo docker exec -it ia_telegraf /bin/bash`

Likewise, to get a bash shell inside the ia_data_analytics docker container simply run the command.
`sudo docker exec -it ia_data_analytics /bin/bash`

## Find the Commands Running Inside the Container

When you are inside a docker container often running a simple process snapshot command will give a lot of information about the purpose of the particular container. Be sure that you are in a container when running this command.

`ps aux`

# Run a Time Series Workflow

## Build and run the sample mqtt temperature sensor

Simple Python temperature sensor simulator

## Configuring Telegraf for PointData Ingestion

To configure the Intel Edge insights Software for point data ingestion for data that is coming over MQTT, open up the telegraf.conf file and find the section for input MQTT.

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

```
cd $EIS_HOME/docker_setup
sudo make build run
```

Now rebuild the EIS containers.

Inside the dockerfile.yml, The MQTT port is exposed to the host system. This means that with telegraph configured to receive input on the same port that any MQTT client which is sending traffic to the host system can be received and fed into the Intel Edge Insight software.

## Check the docker logs

Uses command to check the logs of the docker telegraph container.
`sudo docker logs ia_telegraf`

## Start the Simulated Sensor Data

Once Intel EIS is rebuilt and running, go to the \$EIS_HOME/tools/mqtt-temp-sensor directory. From here you can launch MQTT broker as well as a publisher and subscriber.

$ cd $EIS_HOME/tools/mqtt-temp-sensor
Build the docker container

`./build.sh`

Run the broker

`./broker.sh`

Run the publisher (different terminal window)

`./publisher.sh`

If you wish to see the messages going over MQTT, run the subscriber with the following command:

`./subscriber.sh`

## Verify the ia_data_agent is Ingesting and Publishing Data

`docker logs -f ia_data_agent`

Below is a snapshot of sample output of the above command.

```
I0608 02:25:57.3029569 StreamManager.go:159] Publishing topic: point_classifier_results
I0608 02:25:58.3029139 StreamManager.go:159] Publishing topic: point_classifier_results
I0608 02:25:59.3025279 StreamManager.go:159] Publishing topic: point_classifier_results
```

## Configure Kapacitor to Send Alerts

## Configure Chronograf to Visualize the Data

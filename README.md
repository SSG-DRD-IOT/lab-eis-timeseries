# Time Series Data with the IntelⓇ Edge Insights Platform

## Introduction

The Intel Edge Insights Software can be configured to not only take in video information but also to take in sensory information and other time series based related data series.

Information in this configuration is ingested with Telegraf, stored within the Influxdb database, alerts and monitors can be set using Kapacitor, and visualized with Chrongraf.

In this lab, we will configure the Intel Edge Insights platform for time series and explore the capabilities that it offers.

## Intel Edge Insights Software Time Series Mode

EIS Time Series Analytics component consist of TICK stack except the chronograf and an extra component called InfluxDBConnector. It is responsible for the classification of the point data use case based on the UDF in Kapacitor and publish the classified data to the EIS Message Bus.

Telegraf is a plugin-driven server agent for collecting and sending metrics. Telegraf can collect metrics from a wide range of inputs and write them into a wide array of outputs. It is plugin-driven for both collection and output of data so it is easily extendable.

For more information on Telegraf Kapacitor is an analytics engine and is a part of TICK stack. User can register custom analytics algorithms written in GO/Python (called as User Defined Functions or UDF). Kapacitor also support many built-in functions which can be invoked thorugh Kapacitor's custom script called as TICK script.

Kapacitor read the data from InfluxDB and invoke UDF/TICK scripts. The given example has alogorithm written in GOLANG and TICK scripts name is "point_classifier.tick". Kapacitor is a monitoring and alerting solution that integrates with multiple reporting out options such as email, executing a script, syslog, Apache Kafka, MQTT, SNMPTrap, Slack Messaging and others. See the documentation for Kapacitor for a complete list.


EIS InfluxDBConnector component is responsible for subscription to InfluxDB database and publishing the point data recieved on the subscription server to the EIS Message Bus.

## Prerequisites

This article was written for the Intel Edge Insights Software version 2.0. The software is on a quarterly release cadence.

## Setup instructions

Setting the Intel insights platform to process time series data.

In order to accomplish this we will do the following things:

- Set a couple of helpful environmental variables so that we don't need to continually re-type directory names
- Shut down the Intel Edge Insights Software
- Modify configuration files
- Build and Repackage it
- Relaunch the Intel Edge Insights Software
- Setup Chronograf to visualize the data

Let's get started.

## Environmental Variables

Let’s set a variable for the top level of the EIS installations. If you unzipped the software in a different location, be sure to change the variable below.

`export EIS_HOME=/home/eis/Workshop/IEdgeInsights-2.0`

## Shut down the Intel Edge Insights Software service

The **docker_setup** directory is the location for building a that you should be in if you're going to rebuild the Edge Insights Software, modify configuration files, or manually start and stop the system.

If the software is running, the first step is to stop all docker containers related to it.

`cd $EIS_HOME/docker_setup`

`sudo docker-compose down`

## Modify the Configuration Files

In order to configure the Intel Edge Insights Software for sensor ingestion . We are going to modify the `$EIS_HOME/docker_setup/docker-compose.yml` file.

```
gedit $EIS_HOME/docker_setup/docker-compose.yml
```
### Comment the ia_video_ingestion and ia_video_analytics Sections

Inside the docker-compose.yml search for the ia_video_ingestion and ia_video_analytics. Comment out the entire section. You can setup EIS to run both video and time series data pipelines, but for this lab we will not be using the video services.


### Modify the ia_visualizer in docker-compose.yml
   In ia_visualizer service, uncomment the following lines:
   ```
   SubTopics: "InfluxDBConnector/point_classifier_results"
   point_classifier_results_cfg: "zmq_tcp,127.0.0.1:65016"
   ```
   and comment the following lines:
   ```
   SubTopics: "VideoAnalytics/camera1_stream_results"
   camera1_stream_results_cfg : "zmq_tcp,127.0.0.1:65013"
   ```
   
## Starting the EIS.
   To start the EIS in production mode, provisioning is required. Then you will rebuild the service containers and launch EIS.
   
   ```
   cd docker_setup/provision
   sudo su
   ./provision_eis.sh ../docker-compose.yml
   
   cd ..
   docker-compose up --build -d
   ```

## Run Simulated Temperature Sensor Data

Next we will start a Python program that will publish simulated temperature data over MQTT. 

### Simple Python temperature sensor simulator.

First, go to the tools/mqtt-temp-sensor directory
```
cd ./tools/mqtt-temp-sensor
```


1. Run the broker (Use `docker ps` to see if the broker has started successfully as the container starts in detached mode)
    ```sh
    $ ./broker.sh
    ```
2. Build and run the MQTT publisher docker container
   * For EIS TimeSeries Analytics usecase
     ```sh
     $ ./build.sh && ./publisher.sh
     ```
   * For DiscoveryCreek usecase (publishes data from csv row by row)
     ```sh
     $ ./build.sh && ./publisher_csv.sh
     ```
     Once can also change filename, topic, sampling rate and subsample parameters in the publisher_csv.sh script.

3. If one wish to see the messages going over MQTT, run the
   subscriber with the following command:
   ```sh
   $ ./subscriber.sh
   ```


## Check the docker logs

To verify the output please check the output of below commands
   ```
   docker logs -f ia_influxdbconnector
   docker logs -f ia_visualizer
   ```

   Below is the snapshot of sample output of the ia_influxdbconnector command.
   ```
   I0822 09:03:01.705940       1 pubManager.go:111] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=19.29358085726703,ts=1566464581.6201317 1566464581621377117] 
   I0822 09:03:01.927094       1 pubManager.go:111] Published message: map[data:point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=19.29358085726703,ts=1566464581.6201317 1566464581621377117]
   I0822 09:03:02.704000       1 pubManager.go:111] Published message: map[data:point_data,host=ia_telegraf,topic=temperature/simulated/0 ts=1566464582.6218634,temperature=27.353740759929877 1566464582622771952]
   ```

   Below is the snapshot of sample output of the ia_visualizer command.
   ```
   2019-08-26 10:38:13,869 : INFO : root : [visualize.py] :zmqSubscriber : in line : [402] : Classifier results: {'data': 'point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=18.624896449633443,ts=1566815892.9866698 1566815892987649482\n'}
   2019-08-26 10:38:15,154 : INFO : root : [visualize.py] :zmqSubscriber : in line : [402] : Classifier results: {'data': 'point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=10.84324034762356,ts=1566815893.9882996 1566815893989408936\n'}
   2019-08-26 10:38:15,154 : INFO : root : [visualize.py] :zmqSubscriber : in line : [402] : Classifier results: {'data': 'point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=10.052214661918322,ts=1566815894.990011 1566815894991129870\n'}
   2019-08-26 10:38:16,776 : INFO : root : [visualize.py] :zmqSubscriber : in line : [402] : Classifier results: {'data': 'point_classifier_results,host=ia_telegraf,topic=temperature/simulated/0 temperature=12.555421975490562,ts=1566815895.9918363 1566815895993111771\n'}


# Congratulations

At this point, you have successfully built a time-series pipeline.

Air Liquide Open Source projects
---

# Predictable Farm -  Proof Of Concept - 2017

## Introduction

Predictable Farm is a connected farm system Proof Of Concept that allows to run a small to mid-sized cluster of farms automatically by capturing relevant sensor data and controling physical actuators via a visual flow editor.

This system has been tested during two year on an aquaponic farm to challenge traditional automation systems using PLCs in term of user experience, reliability and pricing.
This Proof Of Concept has never gone to production phase because of the lack of reliability of field device sensors (calibration issues) and actuators (high power relay generating ECM disturbing ethernet PoE channel on our design) but may be adapted to run on more appropriate field devices or just for hobyist projects at home.
During this Proof Of Concept, Air Liquide digital teams developped somes interresting add-ons, that may be usefull as starting point for other projects and are therefore shared on this Github group. You may especially have a look to the way we integrated third party dashboard with [node-red](https://nodered.org/), enalbled [BOOST](https://www.highcharts.com/docs/advanced-chart-features/boost-module) library to help handled several thousand of data on JS chart libraries without any lag,  time series database Cassandra to store all sensors events in real time, or the offline strategy generator to deploy node red flow (recipe) directly into field devices, or very simple device management based on remote firmware update and remote ssh tunnels. 

The following repositories belong to the Predictable Farm Project : 

**Embedded device software** :

  - [predictable.farm.embedded-software.os](https://github.com/airliquide/predictable.farm.embedded-software.os) - yocto Linux on Siemens IOT2040
  - [predictable.farm.embedded-software ](https://github.com/airliquide/predictable.farm.embedded-software) - nodejs set of processes running on Siemens IOT2040 with event orchestration using websockets and sensor connectivity with libMRAA
  - [predictable.farm.embedded-software.offline-strategy-generator](https://github.com/airliquide/predictable.farm.embedded-software.offline-strategy-generator) - nodejs code to decode IBM node-red flows generated in the cloud and published over websocket to the target
  - [predictable.farm.embedded-software.sensor-simulator](https://github.com/airliquide/predictable.farm.embedded-software.sensor-simulator) - node js sensor simulation using websockets

**General instance** :

  - [predictable.farm.proxy](https://github.com/airliquide/predictable.farm.proxy) - redbird nodejs proxy
  - [predictable.farm.authentication-bridge](https://github.com/airliquide/predictable.farm.authentication-bridge) - sqlite + react web app 

**Local instances** (farms) :

  - [predictable.farm.cassandra](https://github.com/airliquide/predictable.farm.cassandra) - high performance real time event historian using this nosql db
  - [predictable.farm.automation-engine](https://github.com/airliquide/predictable.farm.automation-engine) - customisation of IBM node-red 1.0 (custom UI, custom nodes for sensors, actuators and maths, conenctivity to cassandra db and custom dasboard app) 
  - [predictable.farm.dashboard](https://github.com/airliquide/predictable.farm.dashboard) - a custom web application using mustachejs , highchart js + boost , cassandra db access 

**Utilities** :

  - [predictable.farm.mirror.client](https://github.com/airliquide/predictable.farm.mirror.client) - on embedded device side : websocket listener + ssh 
  - [predictable.farm.mirror.server](https://github.com/airliquide/predictable.farm.mirror.server) - on the cloud - ssh tunnel management
  - [predictable.farm.mirror.webapp](https://github.com/airliquide/predictable.farm.mirror.webapp) - on the cloud ssh tunnel monitoring and generator thanks to websocket and online ssh terminal 


### Technical stack

The system is built on the following technologies :

**Hardware**

We use the [Siemens IOT2000 automation gateway](https://w3.siemens.com/mcms/pc-based-automation/en/industrial-iot/pages/default.aspx) with a custom interface board (see the CAD folder in [predictable.farm.embedded-software]()). The OS is built using specific [Yocto](https://www.yoctoproject.org/) layers.

> The hardware files are provided as [Fritzing](http://fritzing.org/home/) files and a BOM (spreadsheet). You can export Gerber files from Fritzing easily.

**Software**

The server side is based on a [Node-RED](https://nodered.org/) instance along with a [Cassandra](https://cassandra.apache.org/) backend, behind a [redbird](https://github.com/OptimalBits/redbird) proxy. A dashboard (NodeJS) is available. Minor utilities are provided, such as tunneling clients and servers to easily monitor the hardware devices.

## How does it work ?

The infrastructure consists of one general proxy/auth instance, and as many local farm instances as there are separate farms.

![infra schema](https://github.com/AirLiquide/airliquide.github.io/blob/main/images/infra.png "General infrastucture")

### Entry point

The main entry point is an EC2 instance that acts as a proxy and authentication frontend.

It gets all the incoming requests, authenticates the clients and triage them, and then redirects the trafic to the farm instance. Each farm has a dedicated instance.

The relevant repositories are [predictable.farm.proxy](https://github.com/airliquide/predictable.farm.proxy) and [predictable.farm.authentication-bridge](https://github.com/airliquide/predictable.farm.authentication-bridge)

### Dedicated farm instances

Each dedicated farm instance is as well an EC2 instance with an internal IP on the same network than the general instance. On it runs :

  - a local proxy (redbird)
  - 3 docker containers : one for the automation engine (Node-RED), one for Cassandra (the storage), and one for the dashboard

When a device is online and connects to send data, it targets an individual farm (_i.e. a subdomain, for instance "MyFarm"_). The general proxy and authentication mechanism will redirect to the local instance dedicated to this individual farm "MyFarm".

In case the connection is made through a websocket, or if the requested resource is `/automation`, the local proxy of the "MyFarm" instance will redirect to Node-RED. Otherwise, it will end up serving the dashboard.

## License

Predictable Farm is licensed under the MIT. A copy of the license and the relevant copyright headers are present in every repository.

## Project status

The project has been used has a POC to test different technlogies and is not in operation internally since underlaying electronics designs was not reliable enough compare to standard PLCs for industrial environement. However many of those technology bricks remains interresting and suitable for a proof of concept and more ( IBM node-red, websocket TLS, yocto Linux, Cassandra). The team was specially proud of the offine strategy generator service running on the embedded software, getting its rules from the standard IBM node red instance, capturing the flows (rules) in JSON format, sending them over websocket and interpretting them locally in case of Internet access issue, or the dashboard connectivity to cassandra database, representation of results with highchartjs and BOOST add-on to handle close to a million data points almost in realtime, without any lag. So fill free to fork the projects or contribute.



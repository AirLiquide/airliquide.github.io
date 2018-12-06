Air Liquide Open Source projects
---

# Predictable Farm

## Introduction

Predictable Farm is a connected farm system that allows to run a small to mid-sized cluster of farms automatically by capturing relevant sensor data and controling physical actuators via a visual flow editor.

The following repositories belong to the Predictable Farm Project : 

**Embedded device software** :

  - [predictable.farm.embedded-software.os](https://github.com/airliquide/predictable.farm.embedded-software.os)
  - [predictable.farm.embedded-software](https://github.com/airliquide/predictable.farm.embedded-software)
  - [predictable.farm.embedded-software.offline-strategy-generator](https://github.com/airliquide/predictable.farm.embedded-software.offline-strategy-generator)
  - [predictable.farm.embedded-software.sensor-simulator](https://github.com/airliquide/predictable.farm.embedded-software.sensor-simulator)

**General instance** :

  - [predictable.farm.proxy](https://github.com/airliquide/predictable.farm.proxy)
  - [predictable.farm.authentication-bridge](https://github.com/airliquide/predictable.farm.authentication-bridge)

**Local instances** (farms) :

  - [predictable.farm.cassandra](https://github.com/airliquide/predictable.farm.cassandra)
  - [predictable.farm.automation-engine](https://github.com/airliquide/predictable.farm.automation-engine)
  - [predictable.farm.dashboard](https://github.com/airliquide/predictable.farm.dashboard)

**Utilities** :

  - [predictable.farm.mirror.client](https://github.com/airliquide/predictable.farm.mirror.client)
  - [predictable.farm.mirror.server](https://github.com/airliquide/predictable.farm.mirror.server)
  - [predictable.farm.mirror.webapp](https://github.com/airliquide/predictable.farm.mirror.webapp)


### Technical stack

The system is built on the following technologies :

**Hardware**

We use the [Siemens IOT2000 automation gateway](https://w3.siemens.com/mcms/pc-based-automation/en/industrial-iot/pages/default.aspx) with a custom interface board (see the CAD folder in [predictable.farm.embedded-software]()). The OS is built using specific [Yocto](https://www.yoctoproject.org/) layers.

> The hardware files are provided as [Fritzing](http://fritzing.org/home/) files and a BOM (spreadsheet). You can export Gerber files from Fritzing easily.

**Software**

The server side is based on a [Node-RED](https://nodered.org/) instance along with a [Cassandra](https://cassandra.apache.org/) backend, behind a [redbird](https://github.com/OptimalBits/redbird) proxy. A dashboard (NodeJS) is available. Minor utilities are provided, such as tunneling clients and servers to easily monitor the hardware devices.

## How does it work ?

The infrastructure consists of one general proxy/auth instance, and as many local farm instances as there are separate farms.

![infra schema](https://github.com/airliquide/airliquide.github.io/raw/master/images/infra.png "General infrastucture")

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
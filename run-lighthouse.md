---
layout: page
title: "Run Lighthouse"
keywords: portworx, px-developer, px-enterprise, install, configure, container, storage, lighthouse
sidebar: home_sidebar
---

This guide shows you how you can run [PX-Enterprise Console](http://lighthouse.portworx.com/) locally.

Note: The example in this section uses Amazon Web Services (AWS) Elastic Compute Cloud (EC2) for servers in the cluster. In your deployment, you can use physical servers, another public cloud, or virtual machines.

### Prerequisite

To start, create one server, following these requirements:

* Image: Must support Docker 1.10 or later, such as:
  * [Red Hat 7.2 (HVM)](https://aws.amazon.com/marketplace/pp/B019NS7T5I) or CentOS
  * [Ubuntu 16.04 (HVM)](https://aws.amazon.com/marketplace/pp/B01JBL2M0O)
  * [Ubuntu 14.04 (HVM)](https://aws.amazon.com/marketplace/pp/B00JV9TBA6)
* Instance type: c3.xlarge
* Number of instances: 1
* Storage:
  * /dev/xvda: 8 GB boot device
* Tag (optional): Add value **px-lighthouse** as the name

### Install and configure Docker

1. Follow the Docker [install guide](https://docs.docker.com/engine/installation/) to install and start the Docker Service.
2. Verify that your Docker version is 1.10 or later.

### Step 1: Install kvdb


>**Important:**
<br/> For PX-Lighthouse, output required from this step: 
<br/>Connection string in 'etcd:http://{IP_ADDRESS}:{Port_NO}' format

* Use your existing kvdb store
* Install as a docker container from the following 
  * [etcd2/etcd3](https://github.com/coreos/etcd/blob/2724c3946eb2f3def5ed38a127be982b62c81779/Documentation/op-guide/container.md)
  
Example docker command to run etcd2 in a contianer:

```
sudo docker run -d -p 4001:4001 -p 2379:2379 -p 2380:2380 --restart always    \
     --name etcd-px quay.io/coreos/etcd:v2.3.7                                \
     -name etcd0                                                              \
     -data-dir /var/lib/etcd/                                                 \
     -advertise-client-urls http://<IP_Address>:2379,http://<IP_Address:4001  \
     -listen-client-urls http://0.0.0.0:2379                                  \
     -initial-advertise-peer-urls http://<IP_Address>:2380                    \
     -listen-peer-urls http://0.0.0.0:2380                                    \
     -initial-cluster-token etcd-cluster                                      \ 
     -initial-cluster etcd0=http://$<IP_Address>:2380                         \
     -initial-cluster-state new
```
  
  * [consul](https://hub.docker.com/_/consul/)
  
Example docker command to run consul in a contianer:

```
sudo docker run -d -p 8300:8300 -p 8500:8500 --restart always  \
     --name consul-px                                          \
     -v /tmp/consul:/consul/data                               \
     consul
```

### Step 2: Install InfluxDB

>**Important:**
<br/> For PX-Lighthouse, output required from this step: 
<br/> 1) ADMIN_USER: Admin username of influxdb for $PWX_INFLUXUSR
<br/> 2) INFLUXDB_INIT_PWD: Password of admin user for PWX_INFLUXPW 
<br/> 3) INFLUXDB_HOSTNAME in 'http://{IP_ADDRESS}:{port}' format

* [Use InfluxCloud](https://cloud.influxdata.com/)
* [Run InfluxDB as a docker container](https://github.com/tutumcloud/influxdb)

Example docker command to run influxdb in a contianer:

```
sudo docker run -d -p 8083:8083 -p 8086:8086 --restart always \
     --name influxdb                                          \
     -e ADMIN_USER="admin"                                    \
     -e INFLUXDB_INIT_PWD="password"                          \
     -e PRE_CREATE_DB="px_stats" tutum/influxdb:latest
```

### Step 3: Launch the PX-Lighthouse Container

### Docker compose method

You can run PX-Lighthouse with [docker-compose](https://docs.docker.com/compose/install/), as follows:

```
# git clone https://github.com/portworx/px-lighthouse.git
# cd px-lighthouse/quick-start
# docker-compose run portworx -daemon -k etcd://myetc.company.com:4001 -c MY_CLUSTER_ID -s /dev/nbd1 -s /dev/nbd2
```

### To run the PX-Lighthouse container


```
# sudo docker run --restart=always --name px-lighthouse -d --net=bridge \
                 -p 80:80                                               \
                 -e PWX_INFLUXUSR="$ADMIN_USER"                         \
                 -e PWX_INFLUXPW="$INFLUXDB_INIT_PWD"                   \
                 portworx/px-lighthouse                                 \
                 -d http://{IP_Address}:{Port_NO}                       \
                 -k etcd:http://{IP_Address}:{Port_NO}                   
```

Runtime command options

```
 -e  PWX_INFLUXUSR
     > Username of influxdb user with admin privilages
 -e  PWX_INFLUXPW
     > Password of PWX_INFLUXUSR
  -d http://<IP_Address>:<Port_NO>
     > Connection string of your influx db
  -k etcd:http://<IP_Address>:<Port_NO>
     > Connection string of your kbdb. 
     > If you are using consul then you can specify your connection string in 'consul:http://<IP_Address>:<Port_NO>' format
```

In your browser visit *http://IP_ADDRESS:80* to access your locally running PX-Lighthouse.

![LH-ON-PREM-FIRST-LOGIN](images/lh-on-prem-first-login-updated_2.png "First Login")
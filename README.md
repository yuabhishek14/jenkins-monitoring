# Jenkins-monitoring

Production Jenkins Monitoring with Grafana , Prometheus and InfluxDB

**Overview:**
A robust monitoring solution for a Jenkins server using Prometheus, Grafana, and InfluxDB. Setup included Docker configuration and integration of monitoring tools. Created customized Grafana dashboards to visualize critical Jenkins metrics, enhancing decision-making for administrators. Improved troubleshooting skills and adeptly resolved data visualization challenges.

- No. of Nodes
- No. of executors
- Build Queue
- Failed Builds
- Successful Builds

## Tech Stack

<p align="center">
  <img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/80447647-d723-42fc-8b60-209d9a511115" alt="Docker Logo" width="100" height="80">
  <img src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" alt="Jenkins Logo" width="60" height="70">
  <img src="https://raw.githubusercontent.com/cncf/landscape/2925475a0257f81356e1d95da3ce9f212ccc4465/hosted_logos/grafana.svg" alt="Grafana" width="100" height="80">
  <img src="https://www.vectorlogo.zone/logos/prometheusio/prometheusio-icon.svg" alt="Prometheus" width="100" height="80">
  <img src="https://www.vectorlogo.zone/logos/influxdata/influxdata-icon.svg" alt="InfluxDB" width="80" height="80">
</p>

## Prerequisite

- Basics of Docker , Jenkins , Grafana , Prometheus , InfluxDB , SQL Queries.
- Ubuntu 22.04 Machine
- Install Docker on your host

## 1. Tools Setup

To Bring the Monitoring Stack with Docker :
Firstly we will create 4 containers for Jenkins, Grafana , Prometheus and InfluxDB

### 1. Setup Jenkins

```bash
docker run -d -p 8080:8080 --name jenkins jenkins/jenkins:lts-jdk11
```

To obtain the Jenkins administrator password, access the Jenkins container:

```bash
docker exec -it jenkins bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

### 2. Setup Prometheus

1. Navigate to your home directory and create a folder named "sndee."

2. Inside the "sndee" folder, create a file named "prometheus.yml" and copy the contents from [Here](https://github.com/prometheus/prometheus/blob/main/documentation/examples/prometheus.yml)

3. Start Prometheus by running the following Docker command:

```bash
docker run -d --name prometheus-container -v /home/sndee/prometheus.yml:/etc/prometheus/prometheus.yml -e TZ=UTC -p 9090:9090 ubuntu/prometheus:2.33-22.04_beta
```

### 3. Setup Grafana

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana:8.5.5
```

### 4. Setup InfluxDB

```bash
docker run -d -p 8086:8086 --name influxdb2 influxdb:1.8.6-alpine
```

## 2. Jenkins - InfluxDB Integration

Before we can start accepting data from Jenkins into InfluxDB, we need to complete a few prerequisites. Follow these steps to set up the integration:

```bash
docker exec -it influxdb2 bash
```

to connect to influx db

```bash
influx
```

create DB

```bash
CREATE DATABASE "jenkins" WITH DURATION 1825d REPLICATION 1 NAME "jenkins-retention"
SHOW DATABASES
```

## 3. Intall Plugins

Install Influx Plugin

 <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/fd806043-6c93-4603-a1af-70058d44391f" alt="image" width="800" height="300" />

- **InfluxDB** plugin to store pipeline information
- **Job and Stage Monitoring** to store Stage and Folder information

Install Prometheus Plugin

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/87d490ff-b113-441d-8403-7656f50e31ac" alt="image" width="650" height="250" />

Restart you Jenkins .This will stop your Jenkins container so we need to start it again.

```
docker start jenkins
```

## 4. Configure InfluxDB on Jenkins

Now we will configure InfluxDB on Jenkins . Go to Configure System.

To get the pipeline data setup this :

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/c6220f22-530f-4964-a673-bffce2618aba" alt="image" width="200" height="470" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/0a3fa932-dbb0-4b2c-9925-f81b0ba8b8f7" alt="image" width="1000" height="500" />

To get the Stage data i.e how much time each stage is taking you need to setup this:

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/62bde4c7-ea42-46de-8f86-0fd18074c628" alt="image" width="350" height="600" />

## 5. Configure the Information about Jenkins in Prometheus

- When we install "Prometheus Plugin" , it exposed a endpoint - VM_IP:8080/prometheus
- So by this endpoint Prometheus will scrap the data from Jenkins
  open the prometheus configuration file

For prometheus to scrap Jenkins data we need to add the endpoint in prometheus file.

```bash
vi prometheus.yml
```

Add Jenkins endpoint

```bash
 - job_name: "jenkins"
    metrics_path: '/prometheus'
    static_configs:
      - targets: ["192.168.1.5:8080"]
```

Restart the prometheus container to sync up the new changes

Now if you go to Prometheus UI you will see the Jenkins endpoint in the target section.

## 6. Create Folders and Jobs

Now we will create Folders and Jobs to produce some data for our Dashboard.

1. Create 2 Folders **call-booking-admin** and **call-booking-user**

Create 2 to 3 jobs each in both of these folders with few basic hello world pipelines and with different no. of stages.

When you will execute these pipelines you will see at the end of the logs that the data will be stored in influxDB
<img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/bcba5909-f9eb-4c97-a61e-87dca7dbe9f6" alt="image" width="480" height="300" />

Now if you check the database you will be able to see tables created in the jenkins DB

image Here ok

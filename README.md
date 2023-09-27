# Jenkins-monitoring

Production Jenkins Monitoring with Grafana , Prometheus and InfluxDB

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/d50680a5-7199-47da-a8d3-46323715feb1)


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

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/a9035d49-ed87-46bf-9857-2ae68fca1793)

## 6. Create Folders and Jobs

Now we will create Folders and Jobs to produce some data for our Dashboard.

1. Create 2 Folders **call-booking-admin** and **call-booking-user**

Create 2 to 3 jobs each in both of these folders with few basic hello world pipelines and with different no. of stages.

When you will execute these pipelines you will see at the end of the logs that the data will be stored in influxDB
<img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/bcba5909-f9eb-4c97-a61e-87dca7dbe9f6" alt="image" width="480" height="300" />

Now if you check the database you will be able to see tables created in the jenkins DB

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/fb091839-676c-4e35-9fef-75b91a927028)


## 7. Grafana

Now we need to add the datasource for the Grafana which will be used for making  Jenkins Monitoring Dashboard

1. Add the Prometheus Endpoint

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/c50a9297-6658-4339-b931-6336e74293e1)

2. Add Influx DB Endpoint

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/380a39e4-20b9-402c-a6f1-b910dae1f347)

3. Add the DB to be used for Dashboard

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/aba1aaa0-e896-4335-8259-2c13286b16a9)

## 8. Create Dashboard

First we will create Drop downs for our Dashboard. By this we ill be able to see Data according to a particular folder or 
Job.

Go to the Varibales section and Add folder and job variable .

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/cc0ece09-02bf-4882-8c40-46d4156ea361)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/cf955f1c-5c02-42b6-919e-f2902c198348)

### 1. Monitor Jenkins status (UP or DOWN):

Panel Type - Stat 

1. Click "Add a new Panel," choose "Stat Panel."
2. Use Prometheus as the datasource, and enter this query: **up{instance="192.168.1.5:8080", job="jenkins"}**.
3. In value mapping, display "UP" with a green background for query result 1, and "DOWN" with a red background for result 0.

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/35ad4c18-b348-471e-a5b7-25c27f350b1d)

### 2. Monitor Executor count, Node count and Queue count:

Panel Type - Time Series 

1. Use Prometheus as the datasource, and enter 3 queries :
- For Executor count : jenkins_executor_count_value
- For Node count : jenkins_node_count_value
- For Queue count : jenkins_queue_size_value

2. Add the respective legend

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/588c2425-e978-4f6e-9795-cb5313f7b246)

### 3. Monitor Pipeline Success, Failure, Abort and Unstable status

Panel Type - Pie Chart 

1. Use Influxdb as the datasource, and enter 4 queries :
- For Success : 
```bash
SELECT count(build_number) FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND ("build_result" = 'SUCCESS' OR "build_result" = 'CompletedSuccess' ) AND $timeFilter 
```

- For Failure :

```bash
SELECT count(build_number) FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND ("build_result" = 'FAILURE' OR "build_result" = 'CompletedError' ) AND $timeFilter 
```

- For Abort :

```bash
SELECT count(build_number) FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND ("build_result" = 'ABORTED' OR "build_result" = 'Aborted' ) AND $timeFilter 
```

- For Unstable :

```bash
SELECT count(build_number) FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND ("build_result" = 'UNSTABLE' OR "build_result" = 'Unstable' ) AND $timeFilter 
```

2. In the Legend Mode select Table mode.

3. Add 4 Override to show Green, Red, Yellow and Grey on Success, Failure, Unstable and Abort respectively.

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/cfcb7054-c746-4ac0-b728-40a00d6d939e)


### 4. Monitor No. of Pipelines Ran

Panel Type - Stat

1. Use Influxdb as the datasource, and enter this query: 

```bash
select count(DISTINCT project_name) FROM jenkins_data WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND $timeFilter 
```
![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/af604668-abbb-4d53-bdfe-969504c41739)

### 5. Monitor No. of Builds

Panel Type - Stat

1. Use Influxdb as the datasource, and enter this query: 

```bash
SELECT count(build_number) FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND $timeFilter 
```

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/e293998b-521d-4ede-8211-c4defcb10886)

### 6. Monitor AVG Build Time

Panel Type - Stat

1. Use Influxdb as the datasource, and enter this query: 

```bash
select build_time/1000 FROM jenkins_data WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND $timeFilter 
```

2. In the Standard options select Unit as Clock(s)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/61cc1365-ed5a-4762-8365-d19f65e50455)

### 7. Latest Build Status

Panel Type - Stat

1. Use Influxdb as the datasource, and enter this query: 

```bash
SELECT build_result FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND $timeFilter  ORDER BY time DESC LIMIT 1
```

2. Add value mappings for Success , Failure , Unstable and Aborted

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/be2ff3f8-ea2c-4eca-ad6b-26c1a312cad4)

### 8. Build Details

Panel Type - Table

1. Use Influxdb as the datasource, and enter this query: 

```bash
SELECT "build_exec_time","project_path","build_number","build_causer","build_time","build_result" FROM "jenkins_data" WHERE ("project_name" =~ /^(?i)$job$/ AND "project_path" =~ /.*(?i)$folder.*$/) AND $timeFilter 
```

2. **Data Transformation**: Click on the "Transform" option and select "outer join" to display all columns.

3. **Override Properties**: Include override properties to modify the column names and unit types.

4. **Data Link Property** for "Pipeline Path" Column: To make all pipeline names clickable and redirect to the corresponding Jenkins pipeline, add the following URL:

```bash
http://192.168.1.5:8080/job/${__data.fields["jenkins_data.project_path"]}/${__data.fields["jenkins_data.build_number"]}
```
5. Value Mapping Property to Handle "job" in URL: To address the issue where "job" appears between the project folder and the job in the URL (e.g., http://192.168.1.5:8080/job/call-booking-user/job/user-ui/4/), add a value mapping property with the following regex and display text:

- Regex: **/(\/)/g**
- Display Text: **/job$1**

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/40487df5-62a3-4a32-acdc-a0c85df79b16)

### 9. No. Of Active , Inactive and Require Update Plugin

Panel Type - Stat

1. Create 3 separate panel with the following query using Prometheus as datasource :

- Active Plugins : ```bash jenkins_plugins_active ```
- Inactive Plugins : ```bash jenkins_plugins_inactive ```
- Require Update Plugins : ```bash jenkins_plugins_withUpdate ```
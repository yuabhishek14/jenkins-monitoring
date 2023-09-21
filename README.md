# jenkins-monitoring
Production Jenkins Monitoring with Grafana , Prometheus and InfluxDB

<<<<<<< HEAD
**Overview:**


=======
>>>>>>> 9b362dd10e057f7ee398461a83c041d37b07d60b
## Tech Stack

<p align="center">
  <img src="https://www.jenkins.io/images/logos/jenkins/jenkins.svg" alt="Jenkins Logo" width="60" height="70">
  <img src="https://github.com/yuabhishek14/Production-E2E-Pipeline/assets/43784560/80447647-d723-42fc-8b60-209d9a511115" alt="Docker Logo" width="100" height="80">
  <img src="https://www.vectorlogo.zone/logos/grafana/grafana-ar21.svg" alt="Grafana" width="180" height="50">
  <img src="https://www.vectorlogo.zone/logos/prometheusio/prometheusio-ar21.svg" alt="Prometheus" width="230" height="80">
  <img src="https://raw.githubusercontent.com/gilbarbara/logos/bea0759cf5fbfaad7e92e6032ff9481dd82de561/logos/influxdb.svg" alt="InfluxDB" width="100" height="80">
</p>


<<<<<<< HEAD
## 1. Tool Setup

First we will create 4 containers of Jenkins, Grafana , Prometheus and InfluxDB

### 1. Setup Jenkins 

```bash
docker run -d -p 8080:8080 --name jenkins
```
go inside the Jenkins container to get the password
```bash
docker exec -it jenkins bash
cat /var/jenkins_home/secrets/initialAdminPassword
```
### 2. Setup Prometheus 

go to home directory and create a folder sndee
 Now create a prometheus.yam file copy the contents from https://github.com/prometheus/prometheus/blob/main/documentation/examples/prometheus.yml

 ```bash
 # my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
 ```

 make prometheus up 

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
 To accept the data from Jenkins to InfluxDb we need to o some pre requisites
 login to InfluxDB and create a database

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

 ## Intall Plugins

 restart you Jenkins 
 this will stop your Jenkins container so we need to start it
  ```
  docker start jenkins
  ```

  <img src="https://github.com/yuabhishek14/jenkins-to-ansible-docker-automation/assets/43784560/a64b0ecd-c570-4a46-ba2a-f8c6f3362d0e" alt="image" width="450" height="300" /> 

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/fd806043-6c93-4603-a1af-70058d44391f" alt="image" width="450" height="300" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/87d490ff-b113-441d-8403-7656f50e31ac" alt="image" width="450" height="300" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/c6220f22-530f-4964-a673-bffce2618aba" alt="image" width="450" height="300" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/0a3fa932-dbb0-4b2c-9925-f81b0ba8b8f7" alt="image" width="450" height="300" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/62bde4c7-ea42-46de-8f86-0fd18074c628" alt="image" width="450" height="300" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/bcba5909-f9eb-4c97-a61e-87dca7dbe9f6" alt="image" width="450" height="300" />

  - When we install "Prometheus Plugin" , it exposed a endpoint - VM_IP:8080/prometheus
  - So by this endpoint Prometheus will scrap the data from Jenkins

  ## Configure the Information about Jenkins in Prometheus

  open the prometheus configuration file 

  ```bash
  vi prometheus.yml
  ```
=======
![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/fd806043-6c93-4603-a1af-70058d44391f)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/87d490ff-b113-441d-8403-7656f50e31ac)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/c6220f22-530f-4964-a673-bffce2618aba)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/0a3fa932-dbb0-4b2c-9925-f81b0ba8b8f7)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/62bde4c7-ea42-46de-8f86-0fd18074c628)

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/bcba5909-f9eb-4c97-a61e-87dca7dbe9f6)
>>>>>>> 9b362dd10e057f7ee398461a83c041d37b07d60b

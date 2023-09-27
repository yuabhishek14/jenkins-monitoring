## Integrations 

### 1. Jenkins - InfluxDB Integration

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

### 2. Intall Plugins

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

### 3. Configure InfluxDB on Jenkins

Now we will configure InfluxDB on Jenkins . Go to Configure System.

To get the pipeline data setup this :

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/c6220f22-530f-4964-a673-bffce2618aba" alt="image" width="200" height="470" />

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/0a3fa932-dbb0-4b2c-9925-f81b0ba8b8f7" alt="image" width="1000" height="500" />

To get the Stage data i.e how much time each stage is taking you need to setup this:

  <img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/62bde4c7-ea42-46de-8f86-0fd18074c628" alt="image" width="350" height="600" />

### 4. Configure the Information about Jenkins in Prometheus

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

### 5. Create Folders and Jobs

Now we will create Folders and Jobs to produce some data for our Dashboard.

1. Create 2 Folders **call-booking-admin** and **call-booking-user**

Create 2 to 3 jobs each in both of these folders with few basic hello world pipelines and with different no. of stages.

When you will execute these pipelines you will see at the end of the logs that the data will be stored in influxDB
<img src="https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/bcba5909-f9eb-4c97-a61e-87dca7dbe9f6" alt="image" width="480" height="300" />

Now if you check the database you will be able to see tables created in the jenkins DB

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/fb091839-676c-4e35-9fef-75b91a927028)


### 6. Grafana

Now we need to add the datasource for the Grafana which will be used for making  Jenkins Monitoring Dashboard

1. Add the Prometheus Endpoint

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/c50a9297-6658-4339-b931-6336e74293e1)

2. Add Influx DB Endpoint

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/380a39e4-20b9-402c-a6f1-b910dae1f347)

3. Add the DB to be used for Dashboard

![image](https://github.com/yuabhishek14/jenkins-monitoring/assets/43784560/aba1aaa0-e896-4335-8259-2c13286b16a9)
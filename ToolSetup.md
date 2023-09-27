## Tools Setup

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

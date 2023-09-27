## Create Dashboard

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
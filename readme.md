## ELK Stack + Jenkins/Gogs docker-compose
This is a docker-compose stack for ElasticSearch, Logstash, Kibana, Jenkins and Gogs.
Idea is to collect and analyze build logs from Jenkins (which is connected to Gogs for automated builds on push/commit) and push the build logs to ElasticSearch.

To run locally, instead of {public_ip} just enter hostname of services (for exp. jenkins would be http://jenkins:8081).

To run on Azure/AWS, it is recommended to use VM with at least 6Gb of ram, and after the VM is up, go to Network Security Group (search NSG) and add Inbound Security Rules for Destination port ranges:

```
Gogs - 3000
Jenkins - 8081
ElasticSearch - 9200
Logstash - 9600
Kibana - 5601
```
Default ELK password is "Elastic". Change it in docker-compose.yml file.

To Start, run in terminal:

 - `docker-compose up -d`

Then, set up accounts for services :

 - Gogs : http://{public_ip}:3000 - Create new account
 - Jenkins : http://{public_ip}:8081 - To Unlock, execute in terminal - `docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`


After the stack is up and running, go to Jenkins, and install [logstash plugin](https://plugins.jenkins.io/logstash/)

Then, go to Jenkins home > Manage Jenkins > Configure System > scroll down untill you find Logstash, and connect directly to ElasticSearch:

- `URI : https://elasticsearch:9200/jenkinslogs/data`

- `UserName : elastic`

- `password: Elastic`

- `Mime Type : application/json `

Save, and proceed to your jenkins freestyle/pipeline jobs

 - Freestyle : at the end of the file, add Post-build Action, choose "Send console logs to logstash"

 - Pipeline : wrap your commands inside steps with "timestamps {logstash{.....}}" OR add "logstashSend failBuild: true, maxLines: 1000" as the last step (see examples on [logstash plugin](https://plugins.jenkins.io/logstash/) page )

After you've modified your pipeline / freestyle, build your job, and than go to Kibana (http://{public_ip}:5601) > Management (on left bar) > Stack management > Kibana > Index patterns > Create New Index pattern , and type "jenkinslogs*" and you should see an index from your jenkins job. Select your @timestamp field as primary time field, click Crete Index pattern and voila! :)

Now you can go to Discover on Kibana's home page, and you will have all the data from jenkins build logs.

Here's an example of my Dashboard with 3 visualizations (All jobs, number of builds per job, and success/failure ratio):

![Kibana Dashboard](https://i.ibb.co/LvKppRm/DASH.jpg?raw=true)




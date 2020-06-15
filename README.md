Docker can be an extremely easy way to set up the stack.

There are various ways to install the stack with Docker. You can pull Elastic’s individual images and run the containers separately or use Docker Compose to build the stack from a variety of available images on the Docker Hub.

For this tutorial, I am using a Dockerized ELK Stack that results in: three Docker containers running in parallel, for Elasticsearch, Logstash and Kibana, port forwarding set up, and a data volume for persisting Elasticsearch data.

To clone the repository:

git clone https://github.com/deviantony/docker-elk.git

To run the stack, simply use:

=> cd /docker-elk
=> docker-compose up -d

Verifying the installation
It might take a while before the entire stack is pulled, built and initialized. After a few minutes, you can begin to verify that everything is running as expected.

=> docker ps

You’ll notice that ports on my localhost have been mapped to the default ports used by Elasticsearch (9200/9300), Kibana (5601) and Logstash (5000/5044).

Everything is already pre-configured with a privileged username and password:

user: elastic
password: changeme

And finally, access Kibana by entering: http://localhost:5601 in your browser.


Shipping data into the Dockerized ELK Stack

Our next step is to forward some data into the stack. By default, the stack will be running Logstash with the default Logstash configuration file. You can configure that file to suit your purposes and ship any type of data into your Dockerized ELK and then restart the container.


Alternatively, you could install Filebeat — either on your host machine or as a container and have Filebeat forward logs into the stack. I highly recommend reading up on using Filebeat on the project’s documentation site.

I am going to install Metricbeat and have it ship data directly to our Dockerized Elasticsearch container (the instructions below show the process for Mac).

First, I will download and install Metricbeat:

curl -L -O 
https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.1
.2-darwin-x86_64.tar.gz

tar xzvf metricbeat-6.1.2-darwin-x86_64.tar.gz

Next, I’m going to configure the metricbeat.yml file to collect metrics on my operating system and ship them to the Elasticsearch container:

cd metricbeat-6.1.2-darwin-x86_64
sudo vim metricbeat.yml
Copy
The configurations:

metricbeat.modules:
- module: system
  metricsets:
    - cpu
    - filesystem
    - memory
    - network
    - process
  enabled: true
  period: 10s
  processes: ['.*']
  cpu_ticks: false

fields:
  env: dev

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["localhost:9200"]
Copy
Last but not least, to start Metricbeat (again, on Mac only):

sudo chown root metricbeat.yml 
sudo chown root modules.d/system.yml 
sudo ./metricbeat -e -c metricbeat.yml -d "publish"
Copy
After a second or two, you will see a Metricbeat index created in Elasticsearch, and it’s pattern identified in Kibana.

curl -XGET 'localhost:9200/_cat/indices?v&pretty'

Define the index pattern, and on the next step select the @timestamp field as your Time Filter.

Creating the index pattern, you will now be able to analyze your data on the Kibana Discover page.
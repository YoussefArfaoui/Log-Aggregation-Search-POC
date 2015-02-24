# Log Aggregation &amp; Search POC:
Log Aggregation &amp; Search POC: Spring-amqp, log4j, RabbitMQ, Logstash, ElasticSearch configuration
 
Table of Contents

1.	[Introduction](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#introduction)

2.	[Requirements](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#requirements)

3.	[Project setup](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#project-setup)

  3.1.	[Create a java maven project](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#create-a-java-maven-project)

  3.2.	[Add maven dependencies](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#add-maven-dependencies)

  3.3.	[Configure log4j file](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#configure-log4j-file)

  3.4.	[Create Application java class](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#create-application-java-class)

4.	[RabbitMQ Installation](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#rabbitmq-installation)

5.	[LogStash Installation](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#logstash-installation)

6.	[ElasticSearch Installation](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#elasticsearch-installation)

7.	[Test the complete configuration](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#test-the-complete--configuration)

8.	[Conclusion](https://github.com/YoussefArfaoui/Log-Aggregation-Search-POC#conclusion)



## Introduction

Monitoring company system logs is essential for spotting problems and
halting the software deployment pipeline. Rather than manually
collecting logs from each machine in an environment, logs should be
centralized to a store that indexes them and makes them available for
searching and advanced processing via a web browser and monitoring
tools. The log store should be connected to all environments to speed up
problem diagnosis and resolution.

In this POC, we have selected some monitoring tools that can work in a
dynamic infrastructure and get integrated easily with the company
environment through scripted configuration at any time from the
application life cycle.

For the Log aggregation and searching tools we have selected
ElasticSearch solution. This solution includes

-   ElasticSearch: it’s a search server based on Lucene apache
    framework. It provides a distributed, multitenant-capable full-text
    search engine with a Restful web interface and schema-free JSON
    documents. Elasticsearch is developed in Java and is released as
    open source under the terms of the Apache License.

-   Logstash: it’s a tool that can be used to collect, process and
    forward events and log messages. Collection is accomplished via
    number of configurable input plugins. Once an input plugin has
    collected data it can be processed by any number of filters which
    modify and annotate the event data. Finally events are routed to
    output plugins which can forward the events to a variety of external
    programs including Elasticsearch, local files and several message
    bus implementations.

-   Kabana: it’s a browser based analytics and search interface for
    Elasticsearch that was developed primarily to view Logstash event
    data.

    Note: Logstash and kabana are part of the Elasticsearch solution.

To connect our log aggregator and search tools to our application
environment (simple java application) we have selected the AMQP
messaging standard, spring-amqp implementation and RabbitMQ as broker
that support this protocol

Why AMQP?

AMQP is a standard wire-level protocol and semantic framework for high
performance enterprise messaging. It was build on top of the TCP/IP
network layer and it provides a better network performance compared to
other messaging protocol. This criterion makes AMQP the preferable
protocol to handle big data like logs.

Why RabbitMQ?

RabbitMQ is an open source message broker software that implements the
Advanced Message Queuing Protocol (AMQP). The server is written in the
Erlang programming language and is built on the Open Telecom Platform
framework for clustering and failover.

## Requirements

To be able to follow this guideline, we are expecting that you have:

-   Basic java, spring and maven knowledge.

-   A development environment is configured: it means that you have
    configured your JDK, your maven and your eclipse IDE correctly.

-   Download RabbitMQ broker from
    [*http://www.rabbitmq.com/*](http://www.rabbitmq.com/)  link (the
    current version is RabbitMQ 3.4.4).

-   Download logstash from
    [*http://www.elasticsearch.org/*](http://www.elasticsearch.org/)
    link (the current version is logstash-1.4.2).

-   Download elasticsearch from
    [*http://www.elasticsearch.org/*](http://www.elasticsearch.org/)
    link (the current version is elasticsearch-1.4.4).

## Project setup

### Create a java maven project

From your eclipse IDE, create a simple maven project. The pom.xml file
will look like the following image


> ![image1](https://cloud.githubusercontent.com/assets/3845225/6345672/a8490c7a-bc08-11e4-89e6-3852b300e788.png)

### Add maven dependencies

After creating the maven project, we need to configure the maven
dependencies to add the logging capability to our project.

-   **Slf4j-api:** The Simple Logging Facade for Java (SLF4J) serves as
    a simple facade or abstraction for various logging frameworks (e.g.
    java.util.logging, logback, log4j) allowing the end user to plug in
    the desired logging framework at deployment time.

-   **Slf4j-log4j12**: It’s a slf4j bridging module for log4j API
    framework.

-   **Spring-rabbit**: It’s a spring module part of the spring-amqp
    project. It provides RabbitMQ implementation for the AMQP messaging
    standard.

> ![image2](https://cloud.githubusercontent.com/assets/3845225/6345671/a848f280-bc08-11e4-931b-c21a1449ee7d.png)

### Configure log4j file

Next is creating a log4j configuration file. If you don’t do, the log4j
framework will use the default system logging configuration.

-   Under main/resource folder, create log4j.xml file

-   Create appenders

    -   Console appender : used to redirect the application logs to the
        default console
```xml
 <appender name="console" class="org.apache.log4j.ConsoleAppender">
        <param name="Target" value="System.out" />
        <layout class="org.apache.log4j.PatternLayout">
                <param name="ConversionPattern" value="%d{HH:mm:ss.SSS} %C{1}  [%t]%-5p %m%n"/>
        </layout>
 </appender>
```
-   Amqp appender : it’s a log4J appender to publish a logging events to
    an AMQP Exchange (in our case it’s a Rabbit AMQP Exchange).

```xml
<appender class="org.springframework.amqp.rabbit.log4j.AmqpAppender" name="amqp">
          <param value="false" name="autoDelete" />
          <param value="text/plain" name="contentType" />
          <param value="false" name="declareExchange" />
          <param value="PERSISTENT" name="deliveryMode" />
          <param value="true" name="durable" />
          <param value="logs" name="exchangeName" />
          <param value="topic" name="exchangeType" />
          <param value="false" name="generateId" />
          <param value="localhost" name="host" />
          <param value="guest" name="username" />
          <param value="guest" name="password" />
          <param value="30" name="maxSenderRetries" />
          <param value="%c.%p" name="routingKeyPattern" />
          <param value="2" name="senderPoolSize" />
          <!-- <param value="/" name="virtualHost" /> -->
          <layout class="org.apache.log4j.PatternLayout">
                <param value="%d %p %t [%c] - &amp;lt;%m&amp;gt;%n" name="ConversionPattern" />
          </layout>
</appender>
```
-   Create loggers : add one logger with logging level=debug.
```xml
<logger name="com.iam.rabbitmq.log.rabbitmqlog">
       <level value="debug" />
</logger>
```

-   Add console and amqp appenders to the root configuration.
```xml
<root>
    <priority value="debug" />
    <appender-ref ref="console" />
    <appender-ref ref="amqp" />
</root>
```

### Create Application java class

Create a simple java class using eclipse that generates a log event each
5 seconds.

![image3](https://cloud.githubusercontent.com/assets/3845225/6345673/a8496210-bc08-11e4-896b-94481edc0b06.png)

## RabbitMQ Installation

The installation is quite simple. Firstly, we need to install the Erlang
language in the machine to be able to run RabbitMQ. So download and run
the [*Erlang Windows Binary File*](http://www.erlang.org/download.html).
Then just run the installer, rabbitmq-server-x.y.z.exe. It takes around
2 minutes and it will set RabbitMQ up and running as a service, with a
default configuration. A new entry in your windows applications list
will be added.

![image4](https://cloud.githubusercontent.com/assets/3845225/6345652/a7e847fa-bc08-11e4-9226-705edb242bfe.png)

-   Start the RabbitMQ server.

Run the RabbitMQ Service- start link from the application menu.

-   Start RabbitMQ web Client : logon to the default url
    <http://localhost:15672/> with the default user ‘guest’ and the
    default user password ‘guest’. ![image5](https://cloud.githubusercontent.com/assets/3845225/6345655/a7eb619c-bc08-11e4-8210-e11ce909eb0f.png)

-   Create a queue: from the Queues tab add new queue with
    name=exchange.log.topic

![image6](https://cloud.githubusercontent.com/assets/3845225/6345653/a7e9328c-bc08-11e4-9922-558ebf1748c3.png)

-   Create exchange logs :from the Exchanges tab add new exchange with
    name=logs type=topic (It should match the amqp appender
    configuration in the log4j.xml file).

![image7](https://cloud.githubusercontent.com/assets/3845225/6345656/a7ec6b82-bc08-11e4-9f64-750fb797c70e.png)

-   Bind the exchange logs to  the queue: from the Queues tab, select
    the created queue and add exchange binding to it. We put the
    RoutingKey=”\#” means we listing to event coming from exchange. We
    can do more filtering later.

![image8](https://cloud.githubusercontent.com/assets/3845225/6345654/a7ea5da6-bc08-11e4-913f-b3ea8dd47f8e.png)

The result will be something like this

![image9](https://cloud.githubusercontent.com/assets/3845225/6345657/a7ed0722-bc08-11e4-96e4-442610f2a6e4.png)

-   Test Application/RabbitMQ connection: to test if our configuration
    was correct, From eclipse run you class a java application(debug
    mode)

![image10](https://cloud.githubusercontent.com/assets/3845225/6345658/a80559ee-bc08-11e4-802e-42b4a8cfdc1c.png)

Log messages in the console

![image11](https://cloud.githubusercontent.com/assets/3845225/6345661/a80ca280-bc08-11e4-8cf0-e7ec7f3f1274.png)

Looking in the RabbitMQ web console, one connection is established, two
communication channels (in/out) was created and message stream is coming
to the created queue.

![image12](https://cloud.githubusercontent.com/assets/3845225/6345660/a80a7dfc-bc08-11e4-9e03-04e193dbf17e.png)

## LogStash Installation

After configuring the appender and the message broker, we pass to the
next step: the logstash configuration.

-   Extract the logstash-x.y.z.zip file in a local folder from your
    choice.

![image13](https://cloud.githubusercontent.com/assets/3845225/6345659/a80a19c0-bc08-11e4-81e5-dc1541329f01.png)

-   Navigate to logstash/bin folder and create new file logstash.config

![image14](https://cloud.githubusercontent.com/assets/3845225/6345663/a80e425c-bc08-11e4-8424-4ef62d1a625d.png)

Logstash is based on the plugin concept. There are three major plugin
types (input, filter and output)

-   The input type: it’s used to configure all plugins used to read the
    input stream and get it in logstash ecosystem.

-   The filter type: it’s used to configure all plugins used to add more
    processing and filtering capabilities inside in the logstash
    ecosystem.

-   The output type: it’s used to configure all plugins used to write
    the logstash output stream.

As we are planning to the read data from RabbitMQ queue and to delegate
the processing of data to ElasticSearch, will configure a RabbitMQ
plugin in the input. The most important parameters are the queue, the
host, the exchange and the key. So make sure that they match your
RabbitMQ configuration. For output will configure the elasticsearch
plugin. For more information consult the elasticsearch website

![image15](https://cloud.githubusercontent.com/assets/3845225/6345662/a80d5b8a-bc08-11e4-920b-2a8546809bb1.png)

```ruby
 input {
    rabbitmq {
       auto_delete => false
       debug => true
       durable => true
       queue => "exchange.log.topic"
       host => "localhost"
       exchange => "logs"
       key => "#"
       }
 }
 
 filter {
      mutate {
        add_field => [ "hostip", "%{host}" ]
       }
       dns {
         reverse => [ "host" ]
         action => replace
      }
 }

 output {
       elasticsearch {
         host => "localhost"
      protocol => "http"
       }
 }
```
## ElasticSearch Installation

Extract the elasticSearch-x.y.z.zip file in a local folder from your
choice.

![image13](https://cloud.githubusercontent.com/assets/3845225/6345659/a80a19c0-bc08-11e4-81e5-dc1541329f01.png)

## Test the complete  configuration

As you can see the basic configuration is very easy, only small concept
that you need to know to be able make all tools together. Next step, we
will try to run all tools together and the final result.

-   Make sure that your application and RabbitMQ broker are running as
    it was described in the previous sections.

-   From the cmd, run the elasticSearch.bat file

![image16](https://cloud.githubusercontent.com/assets/3845225/6345664/a8213ab0-bc08-11e4-83e5-19e7ea43f6df.png)

The expected message:

![image17](https://cloud.githubusercontent.com/assets/3845225/6345668/a82b3cc2-bc08-11e4-82e7-5d6072503271.png)

-   From cmd, run the logstash agent by specifying the logstash.conf
    configuration file

![image18](https://cloud.githubusercontent.com/assets/3845225/6345666/a82ab25c-bc08-11e4-8246-fb16abf14a14.png)

-   After starting the agent, launch logstash web component (logstash
    includes a web component called Kabana to provide web visualization
    of the logged data…).

![image19](https://cloud.githubusercontent.com/assets/3845225/6345667/a82ab676-bc08-11e4-9e50-b8de301e66f6.png)

-   Using your web browser, logon to <http://localhost:9292/> , you will
    get this result: a processed data is shown in the diagram

![image20](https://cloud.githubusercontent.com/assets/3845225/6345665/a828c49c-bc08-11e4-87db-cd3c9b33046e.png)

Logon again to RabbitMQ web console, and new connection is added with
type consumer and new channel is added to. You can remark that the data
is in processing mode

![image21](https://cloud.githubusercontent.com/assets/3845225/6345669/a8326a88-bc08-11e4-83a7-9e856d214a0f.png)

You can use the Kabana logstash web interface to filter and search for
specific events and messages:

![image22](https://cloud.githubusercontent.com/assets/3845225/6345670/a83fbb3e-bc08-11e4-9e88-fce94ac7207e.png)

## Conclusion

As you can see the configuration was simple. This basic configuration
can be easily extended by adding other applications and environments
using different integration protocols. Also we can add more data
processing logic using filters and other logstash capabilities.

For more advanced configuration, please consult :

<http://www.elasticsearch.org/>

<http://logstash.net/>

<http://www.rabbitmq.com/>

# Central logging

Message-consumer centralizes all oxauth logs in one place and to provide a quick access to logging data by exposing RESTful API for searching with custom conditions.

## log node:

we recoment to setup a new VM to message-consumer

## Prerequisites

we need mysql, java, maven, activemq, unzip and wget 
```
# apt-get update
# apt-get install openjdk-7-jdk-headless wget unzip mysql-server
# apt-get install mysql-server
# mysql_secure_installation
```

### Install maven
```
# wget http://www-us.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz -P /tmp
# tar xf /tmp/apache-maven-3.3.9-bin.tar.gz -C /opt
# rm -rf /tmp/apache-maven-3.3.9-bin.tar.gz
# export PATH=/opt/apache-maven-3.3.9/bin:$PATH
```

### Manual install activemq
Download it form this location using a web browser,
http://www.apache.org/dyn/closer.cgi?filename=/activemq/5.14.3/apache-activemq-5.14.3-bin.tar.gz&action=download
```
tar xf /tmp/apache-activemq-5.14.3-bin.tar.gz -C /opt
mv /opt/apache-activemq-5.14.3 /opt/activemq
```
Start activemq,
```
/opt/activemq/bin/activemq start
```
please change activemq default admin password.


## Build and install message-consumer
we need to clone message comsumer from github then run these commands,
```
git clone https://github.com/GluuFederation/message-consumer.git
cd message-consumer
mvn -Pprod clean package
mv message-consumer-master/target/message-consumer-0.0.1-SNAPSHOT.jar /opt
```

## Add sql schema to mysql

get schema from here, https://github.com/GluuFederation/message-consumer/blob/master/schema/mysql_schema.sql
login mysql shell,

```
source /path/to/file/mysql_schema.sql
```

## Configure oxauth-server logging
To configure [oxauth-server](https://github.com/GluuFederation/oxAuth/tree/master/Server) to send logging messages via JMS, just add JMSAppender into [log4j2.xml](https://github.com/GluuFederation/oxAuth/blob/master/Server/src/main/resources/log4j2.xml) e.g:

```
    <JMS name="jmsQueue"
        destinationBindingName="dynamicQueues/oxauth.server.logging"
        factoryName="org.apache.activemq.jndi.ActiveMQInitialContextFactory"
        factoryBindingName="ConnectionFactory"
        providerURL="tcp://mc.gluu.private:61616"
        userName="admin"
        password="{yourpass}">
    </JMS>

```
and `<AppenderRef ref="jmsQueue"/>` to the `root` tag in the [log4j2.xml](https://github.com/GluuFederation/oxAuth/blob/master/Server/src/main/resources/log4j2.xml#L139) file.

## message-consumer properties file

Message-consumer needs properties file to run.
Put the fellowing content in /etc/message-consumer/prod-mc-mysql.properties 
```
spring.activemq.broker-url=failover:(tcp://localhost:61616)?timeout=5000
spring.activemq.user=admin
spring.activemq.password={yourpass}
spring.activemq.packages.trust-all=true

#spring.datasource.url=jdbc:mysql://localhost:3306/gluu_log
spring.datasource.username=root
spring.datasource.password={yourpass}

spring.jpa.database=mysql
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect

spring.data.rest.base-path=/logger/api
spring.data.rest.default-page-size=20
spring.data.rest.max-page-size=100

server.port=9339


# ===================================================================
# message-consumer specific properties
# ===================================================================
message-consumer.oauth2-audit.destination=oauth2.audit.logging
message-consumer.oauth2-audit.days-after-logs-can-be-deleted=365
#every day at 1:01:am
message-consumer.oauth2-audit.cron-for-log-cleaner=0 1 1 * * ?

message-consumer.oxauth-server.destination=oxauth.server.logging
message-consumer.oxauth-server.days-after-logs-can-be-deleted=10
#every day at 1:01:am
message-consumer.oxauth-server.cron-for-log-cleaner=0 1 1 * * ?
```

## run message-consumer:
```
java -jar /opt/message-consumer-0.0.1-SNAPSHOT.jar --spring.config.location=/etc/message-consumer/ --spring.config.name=prod-mc-mysql --database=mysql
```

## Adding message-consumer to cluster

Simply add the ip of message-consumer node in host-1 and host-2 's chroot /etc/hosts file

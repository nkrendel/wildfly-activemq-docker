# wildfly-activemq-postgres
This docker file is based on the jboss/base-jdk:7 image and adds the following:

+ Wildfly 8.2.0.Final
+ ActiveMQ 5.11.1 Resource Adapter (activemq-ra.rar) configured to run as an embedded broker
+ ActiveMQ 5.11.1 Web Console (amq-console.war) which will connect to the embedded broker
+ Postgresql 9.3-1102 jdbc4 driver

It will also add an admin user with sample credentials admin/admin.

The ActiveMQ web console is accessible via `http://{host}:8080/amq-console`.


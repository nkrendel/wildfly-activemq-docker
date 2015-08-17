# Use latest jboss/base-jdk:7 image as the base, and based upon the jboss/wildfly Dockerfile
FROM jboss/base-jdk:7

# Set some needed environment variables
ENV WILDFLY_VERSION 8.2.0.Final
ENV WILDFLY_SHA1 d78a864386a9bc08812eed9781722e45812a7826
ENV JBOSS_HOME /opt/jboss/wildfly
ENV POSTGRES_VERSION 9.3-1102
ENV GITHUB_URL https://github.com/nkrendel/wildfly-activemq-docker
ENV REPO_URL https://repository.apache.org/content/repositories/releases

# Add the WildFly distribution to /opt, and make wildfly the owner of the extracted tar content
# Make sure the distribution is available from a well-known place
RUN cd $HOME \
    && curl -O https://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.tar.gz \
    && sha1sum wildfly-$WILDFLY_VERSION.tar.gz | grep $WILDFLY_SHA1 \
    && tar xf wildfly-$WILDFLY_VERSION.tar.gz \
    && mv $HOME/wildfly-$WILDFLY_VERSION $JBOSS_HOME \
    && rm wildfly-$WILDFLY_VERSION.tar.gz

# Add postgres module
RUN curl -O https://jdbc.postgresql.org/download/postgresql-$POSTGRES_VERSION.jdbc4.jar \
    && mv $HOME/postgresql-$POSTGRES_VERSION.jdbc4.jar /tmp/

# Add ActiveMQ Resource Adapter from Github
RUN curl -L -O $GITHUB_URL/raw/master/activemq-ra/activemq-ra.rar \
    && mv $HOME/activemq-ra.rar $JBOSS_HOME/standalone/deployments 

# Add the jboss-cli commands for postgres and ActiveMQ
ADD execute.sh /tmp/execute.sh
ADD commands.cli /tmp/commands.tmp

# Replace postgres version in command file to make it more generic
RUN sed -e "s/POSTGRES_VERSION/$POSTGRES_VERSION/g" /tmp/commands.tmp > /tmp/commands.cli

# Start up server, run cli, and remove history directory to fix "Could not rename" error
RUN /tmp/execute.sh \
    && rm -rf $JBOSS_HOME/standalone/configuration/standalone_xml_history

# Add ActiveMQ Web Console
RUN cd $HOME \
    && curl $REPO_URL/org/apache/activemq/activemq-web-console/5.11.1/activemq-web-console-5.11.1.war -o amq-console.war \
    && mv $HOME/amq-console.war $JBOSS_HOME/standalone/deployments \
    && rm /tmp/postgresql-$POSTGRES_VERSION.jdbc4.jar

# Add ActiveMQ Web Console related properties to standalone.sh
RUN sed -i -e "s/SERVER_OPTS=\"\"/SERVER_OPTS=\"-Dwebconsole.type=properties -Dwebconsole.jms.url=tcp:\/\/localhost:61616 -Dwebconsole.jmx.url=service:jmx:http-remoting-jmx:\/\/localhost:9990\"/" $JBOSS_HOME/bin/standalone.sh

# Add user
RUN /opt/jboss/wildfly/bin/add-user.sh admin admin --silent

# Expose the ports we're interested in
EXPOSE 8080 9990

# Set the default entrypoint to run on boot
# This will boot WildFly in the standalone mode and bind to all interface
ENTRYPOINT ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]
CMD ["-Dactivemq.data=wildfly/standalone/data/activemq"]

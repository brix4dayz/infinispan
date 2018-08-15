# Use latest jboss/base-jdk:8 image as the base
FROM jboss/base-jdk:8

# Set the INFINISPAN_SERVER_HOME env variable
ENV INFINISPAN_SERVER_HOME /opt/jboss/infinispan-server

# Set the INFINISPAN_VERSION env variable
ENV INFINISPAN_VERSION 8.2.8.Final

ENV MGMT_USER admin

ENV MGMT_PASS admin

# Ensure signals are forwarded to the JVM process correctly for graceful shutdown
ENV LAUNCH_JBOSS_IN_BACKGROUND true

# Server download location
ENV DISTRIBUTION_URL https://repository.jboss.org/nexus/content/repositories/releases/org/infinispan/server/infinispan-server-build/$INFINISPAN_VERSION/infinispan-server-build-$INFINISPAN_VERSION.zip

ENV JGROUPS_VERSION 3.6.16.Final
ENV JGROUPS_K8S_VERSION 0.9.3
ENV OAUTH_VERSION 20100527

ENV JGROUPS_JAR jgroups-$JGROUPS_VERSION.jar
ENV JGROUPS_K8S_JAR kubernetes-$JGROUPS_K8S_VERSION.jar
ENV JGROUPS_K8S_COMMON_JAR common-$JGROUPS_K8S_VERSION.jar
ENV JGROUPS_K8S_DNS_JAR dns-$JGROUPS_K8S_VERSION.jar
ENV OAUTH_JAR oauth-$OAUTH_VERSION.jar

ENV JGROUPS_URL https://repository.jboss.org/nexus/content/repositories/releases/org/jgroups/jgroups/$JGROUPS_VERSION/$JGROUPS_JAR
ENV JGROUPS_K8S_URL https://repository.jboss.org/nexus/content/repositories/releases/org/jgroups/kubernetes/kubernetes/$JGROUPS_K8S_VERSION/$JGROUPS_K8S_JAR
ENV JGROUPS_K8S_COMMON_URL https://repository.jboss.org/nexus/content/repositories/releases/org/jgroups/kubernetes/common/$JGROUPS_K8S_VERSION/$JGROUPS_K8S_COMMON_JAR
ENV JGROUPS_K8S_DNS_URL https://repository.jboss.org/nexus/content/repositories/releases/org/jgroups/kubernetes/dns/$JGROUPS_K8S_VERSION/$JGROUPS_K8S_DNS_JAR
ENV OAUTH_URL http://central.maven.org/maven2/net/oauth/core/oauth/$OAUTH_VERSION/$OAUTH_JAR

ENV JGROUPS_DIR $INFINISPAN_SERVER_HOME/modules/system/layers/base/org/jgroups
ENV JGROUPS_K8S_DIR $JGROUPS_DIR/kubernetes/
ENV JGROUPS_K8S_COMMON_DIR $JGROUPS_K8S_DIR/common/main
ENV JGROUPS_K8S_DNS_DIR $JGROUPS_K8S_DIR/dns/main
ENV OAUTH_DIR $INFINISPAN_SERVER_HOME/modules/system/layers/base/net/oauth/core/main

# Download and extract the Infinispan Server
USER root

ENV HOME /opt/jboss/

# Infinispan
RUN INFINISPAN_SHA=$(curl $DISTRIBUTION_URL.sha1); curl -o /tmp/server.zip $DISTRIBUTION_URL && sha1sum /tmp/server.zip | grep $INFINISPAN_SHA \
    && unzip -q /tmp/server.zip -d $HOME && mv $HOME/infinispan-server-* $INFINISPAN_SERVER_HOME && rm /tmp/server.zip \
    && chown -R 1000.0 $INFINISPAN_SERVER_HOME \
    && chmod -R g+rw $INFINISPAN_SERVER_HOME \
    && find /opt/jboss/infinispan-server/ -type d -exec chmod g+x {} +

# JGroups
RUN mkdir -p $JGROUPS_DIR/main \
    && curl -o $JGROUPS_DIR/main/$JGROUPS_JAR $JGROUPS_URL \
    && ln -s $JGROUPS_DIR/main/$JGROUPS_JAR $JGROUPS_DIR/main/jgroups.jar

# JGroups K8s
RUN mkdir -p $JGROUPS_K8S_DIR/main \
    && curl -o $JGROUPS_K8S_DIR/main/$JGROUPS_K8S_JAR $JGROUPS_K8S_URL \
    && ln -s $JGROUPS_K8S_DIR/main/$JGROUPS_K8S_JAR $JGROUPS_K8S_DIR/main/kubernetes.jar

# JGroups K8s Common
RUN mkdir -p $JGROUPS_K8S_COMMON_DIR \
    && curl -o $JGROUPS_K8S_COMMON_DIR/$JGROUPS_K8S_COMMON_JAR $JGROUPS_K8S_COMMON_URL \
    && ln -s $JGROUPS_K8S_COMMON_DIR/$JGROUPS_K8S_COMMON_JAR $JGROUPS_K8S_COMMON_DIR/common.jar

# JGroups K8s DNS
RUN mkdir -p $JGROUPS_K8S_DNS_DIR \
    && curl -o $JGROUPS_K8S_DNS_DIR/$JGROUPS_K8S_DNS_JAR $JGROUPS_K8S_DNS_URL \
    && ln -s $JGROUPS_K8S_DNS_DIR/$JGROUPS_K8S_DNS_JAR $JGROUPS_K8S_DNS_DIR/dns.jar

# OAuth
RUN mkdir -p $OAUTH_DIR \
    && curl -o $OAUTH_DIR/$OAUTH_JAR $OAUTH_URL \
    && ln -s $OAUTH_DIR/$OAUTH_JAR $OAUTH_DIR/oauth.jar

USER 1000

# Copy entrypoint script
COPY docker-entrypoint.sh /usr/local/bin
COPY is_running.sh /usr/local/bin
COPY modules/jgroups/module.xml $JGROUPS_DIR/main
COPY modules/jgroups/kubernetes/module.xml $JGROUPS_K8S_DIR/main
COPY modules/jgroups/kubernetes/common/module.xml $JGROUPS_K8S_COMMON_DIR
COPY modules/jgroups/kubernetes/dns/module.xml $JGROUPS_K8S_DNS_DIR
COPY modules/oauth $OAUTH_DIR


ENTRYPOINT ["docker-entrypoint.sh"]

# Expose Infinispan server ports
EXPOSE 7600 8080 8181 9990 11211 11222 57600 8888

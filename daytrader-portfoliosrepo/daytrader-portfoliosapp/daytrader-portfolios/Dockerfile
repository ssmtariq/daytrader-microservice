##
##  Use debian base image; instead of linux alpine -- it includes bash
##
#FROM openjdk:11-jre
FROM ubuntu:18.04
##
##  Sets the author field of the generated image
##
MAINTAINER Donald Vines <donald_vines@hotmail.com>

RUN apt update -y \
    && apt-get upgrade -y\
    && apt install software-properties-common -y \
    && add-apt-repository ppa:deadsnakes/ppa -y \
    && add-apt-repository ppa:openjdk-r/ppa -y \
    && apt update -y \
    && apt-get install python3 -y \
    && apt-get install python3-pip -y \
    && apt-get install -y git build-essential \
    && apt-get install -y libnuma-dev

RUN alias python=python3
RUN apt install -y --reinstall -o Dpkg::Options::="--force-confask,confnew,confmiss" --reinstall ca-certificates-java ssl-cert openssl ca-certificates

RUN apt -y install openjdk-11-jre openjdk-11-jdk

ARG MAVEN_VERSION=3.8.6
ARG USER_HOME_DIR="/root"
RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
    && apt-get install curl -y \
    && echo "Downlaoding maven" \
    && curl -fsSL -o /tmp/apache-maven.tar.gz  https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz \
    && echo "Unziping maven" \
    && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
    && echo "Cleaning and setting links" \
    && rm -f /tmp/apache-maven.tar.gz \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
RUN mvn -version


##
##  Declare Docker ARGs whose values are set when building the image; but they are not available to the container
##
##  1.  App/WAR Name & Version
ARG APP_ARTIFACTID
ARG WAR_ARTIFACTID
ARG APP_VERSION
##  2.  Exposed Port
ARG EXPOSE_PORT
##  3.  Database's URL
ARG DATABASE_DRIVER
ARG DATABASE_URL
ARG DATABASE_USERNAME
ARG DATABASE_PASSWORD
##
##  Set Docker ENV Values that are not only available when building the image, but also to the running container
##
##  1. SSL Variables
ENV DAYTRADER_KEYSTORE_FILENAME=/var/ssl/daytrader/keystore.jks
ENV DAYTRADER_KEYSTORE_PASSWORD=password
ENV DAYTRADER_TRUSTSTORE_LOCATION=/var/ssl/daytrader/truststore.jks
ENV DAYTRADER_TRUSTSTORE_PASSWORD=password
##  2.  Database Variables
ENV DAYTRADER_DATABASE_DRIVER=${DATABASE_DRIVER}
ENV DAYTRADER_DATABASE_URL=${DATABASE_URL}
ENV DAYTRADER_DATABASE_USERNAME=${DATABASE_USERNAME}
ENV DAYTRADER_DATABASE_PASSWORD=${DATABASE_PASSWORD}
##  3.  Tomcat Variables
ENV SERVER_PORT=${EXPOSE_PORT}
ENV SERVER_PORT_HTTPS=${EXPOSE_PORT}
##  4.  Routing Variables
ENV DAYTRADER_ACCOUNTS_SERVICE=https://daytrader-accounts
ENV DAYTRADER_GATEWAY_SERVICE=https://daytrader-gateway
ENV DAYTRADER_PORTFOLIOS_SERVICE=https://daytrader-portfolios
ENV DAYTRADER_QUOTES_SERVICE=https://daytrader-quotes
##  5.  Logging Variables
ENV DAYTRADER_LOG_FILENAME=/var/log/daytrader/${APP_ARTIFACTID}-${APP_VERSION}.log
ENV DAYTRADER_LOG_LEVEL=TRACE
ENV DAYTRADER_LOG_APPENDER=ConsoleAppender
##
##  Setup the environment for the running container
##
##  1.  Create Database Folder 
RUN mkdir -p -m 0777 /var/dat/daytrader
##  2.  Create Logging Folder, Logging File, & Set Permissions
RUN mkdir -p -m 0777 /var/log/daytrader
RUN touch $DAYTRADER_LOG_FILENAME
RUN chmod 666 $DAYTRADER_LOG_FILENAME
##  3.  Create SSL Folder, Add Truststore, & Set Permissions
RUN mkdir -p -m 0777 /var/ssl/daytrader
ARG JKS_FILE=target/${WAR_ARTIFACTID}-${APP_VERSION}/WEB-INF/classes/truststore.jks
ADD ${JKS_FILE} $DAYTRADER_TRUSTSTORE_LOCATION
RUN chmod 666 $DAYTRADER_TRUSTSTORE_LOCATION
##  4.  Inform Docker that the container listens on the specified port
EXPOSE ${EXPOSE_PORT}
##
##  Run the application in the container
##
##  1.  Setup the JAVA_OPTS for the application
ENV JAVA_OPTS="-Djavax.net.ssl.trustStore=/var/ssl/daytrader/truststore.jks -Djavax.net.ssl.trustStorePassword=password"
##  2.  Add the WAR to the container 
ARG WAR_FILE=target/${WAR_ARTIFACTID}-${APP_VERSION}.war
ADD ${WAR_FILE} app.war
##  3.  Start the executable WAR
#ENTRYPOINT exec java $JAVA_OPTS -jar app.war



#RUN alias python=python3

# RUN mkdir /etc/ssl/certs/java/
#RUN apt install -y --reinstall -o Dpkg::Options::="--force-confask,confnew,confmiss" --reinstall ca-certificates-java ssl-cert openssl ca-certificates

#RUN apt -y install openjdk-11-jre openjdk-11-jdk

# Downloading and installing Maven


# RUN pip install bintrees google-api-python-client

COPY src/ src/
COPY sperf/ sperf/

WORKDIR sperf/

ENV JXPerf_HOME /sperf
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64
ENV MAVEN_HOME /usr/share/maven
ENV PATH $JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
ENV LD_LIBRARY_PATH $JXPerf_HOME/bin/kissmalloc:$JXPerf_HOME/build/preload:$LD_LIBRARY_PATH
ENV JAVA_AGENT $JXPerf_HOME/thirdparty/allocation-instrumenter/target/java-allocation-instrumenter-HEAD-SNAPSHOT.jar

#Replace with instrumenting class package
#RUN sed -i 's/replaceWithClassName/edu\/fudan\/common\//' /sperf/thirdparty/allocation-instrumenter/src/main/java/com/google/monitoring/runtime/instrumentation/AllocationInstrumenter.java
#RUN grep -C 0 "fudan" /sperf/thirdparty/allocation-instrumenter/src/main/java/com/google/monitoring/runtime/instrumentation/AllocationInstrumenter.java

RUN make -j

RUN mkdir /cache
WORKDIR /cache

RUN sysctl -a | grep perf_event_paranoid

CMD ["sysctl", "-w", "kernel.perf_event_paranoid=1"]

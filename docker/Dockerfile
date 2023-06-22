FROM openjdk:8-jre-alpine
WORKDIR /home/alpine
RUN apk update && apk add wget
RUN wget -O amq.tar.gz https://www.apache.org/dyn/closer.cgi?filename=/activemq/5.17.4/apache-activemq-5.17.4-bin.tar.gz&action=download && tar -xvf amq.tar.gz
EXPOSE 8161 61616 5672 61613 1833
CMD ["/bin/sh","apache-activemq-5.15.10/bin/activemq","console"]

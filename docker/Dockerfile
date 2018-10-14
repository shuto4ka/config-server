FROM ubuntu:18.04

ARG JAVA_VERSION=11
ENV JDK_FILE openjdk-${JAVA_VERSION}_linux-x64_bin.tar.gz

ENV JAVA_OPTS -Xms128m -Xmx1024m
ENV JMX_OPTS -Djava.rmi.server.hostname=config-server
ENV TZ Europe/Moscow

LABEL description="Config-server image"

RUN apt-get update && apt-get install -y -q --no-install-recommends wget tzdata && apt-get clean
RUN wget --no-check-certificate -qO  /tmp/$JDK_FILE https://download.java.net/java/ga/jdk$JAVA_VERSION/$JDK_FILE
RUN tar -xf /tmp/$JDK_FILE -C /opt \
  && rm /tmp/$JDK_FILE \
  && ln -s /opt/jdk-${JAVA_VERSION} /opt/java

RUN rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get clean

RUN echo "Adding app user and group" \
    && groupadd --system --gid 1000 application \
    && useradd --system --gid application --uid 1000 --shell /bin/bash --home /app config-server \
    && mkdir /app \
    && chown config-server /app

ARG app_vers

COPY --chown=1000:0 config-server-${app_vers}.jar /app/config-server.jar

USER config-server
WORKDIR /app

CMD /opt/java/bin/java $JMX_OPTS $JAVA_OPTS -jar /app/config-server.jar --spring.config.location=file:/app/application.yml
FROM alpine:3.8

ARG JAVA_VERSION=11
ENV JDK_FILE openjdk-${JAVA_VERSION}_linux-x64_bin.tar.gz

ENV JAVA_OPTS -Xms128m -Xmx1024m
ENV JMX_OPTS -Djava.rmi.server.hostname=config-server
ENV TZ Europe/Moscow

LABEL description="Config-server image"

RUN apk update && apk add -q --no-cache wget tzdata
RUN wget --no-check-certificate -qO  /tmp/$JDK_FILE https://download.java.net/java/ga/jdk$JAVA_VERSION/$JDK_FILE
RUN mkdir /opt \
  && tar -xf /tmp/$JDK_FILE -C /opt \
  && rm /tmp/$JDK_FILE \
  && ln -s /opt/jdk-${JAVA_VERSION} /opt/java

RUN rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime

RUN echo "Adding app user and group" \
    && addgroup -S --gid 1000 application \
    && adduser -S -G application -u 1000 -s /bin/bash -h /app config-server \
#    && mkdir /app \
    && chown config-server /app

ARG app_vers

COPY --chown=1000:0 config-server-${app_vers}.jar /app/config-server.jar
COPY --chown=1000:0 application.yml /app/application.yml

USER config-server
WORKDIR /app

CMD /opt/java/bin/java $JMX_OPTS $JAVA_OPTS -jar /app/config-server.jar --spring.config.location=file:/app/application.yml

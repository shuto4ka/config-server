FROM openjdk:11.0.1-jre-slim

ENV JAVA_OPTS -Xms128m -Xmx1024m
ENV JMX_OPTS -Djava.rmi.server.hostname=config-server
ENV TZ Europe/Moscow

RUN rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime

RUN echo "Adding app user and group" \
    && addgroup --system --gid 1000 application \
    && adduser --system --ingroup application -u 1000 --shell /bin/bash --home /app config-server \
#    && mkdir /app \
    && chown config-server /app

ARG app_vers

COPY --chown=1000:0 config-server-${app_vers}.jar /app/config-server.jar
COPY --chown=1000:0 application.yml /app/application.yml

USER config-server
WORKDIR /app

CMD java $JMX_OPTS $JAVA_OPTS -jar /app/config-server.jar --spring.config.location=file:/app/application.yml

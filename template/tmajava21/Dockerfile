FROM maven:3.9.9-eclipse-temurin-21 as builder

WORKDIR /home/app
COPY . /home/app

# build shared lib
RUN cd function/shared-lib && mvn clean install 
# build function
RUN cd function && mvn clean package -U -DskipTests 

FROM ghcr.io/openfaas/of-watchdog:0.10.9 AS watchdog

FROM eclipse-temurin:21-jre-alpine as ship

RUN addgroup --system app \
    && adduser --system --ingroup app app

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

WORKDIR /home/app
COPY --from=builder /home/app/function/target/tmajava-1.0-SNAPSHOT-jar-with-dependencies.jar /home/app/tmajava.jar
user app

WORKDIR /home/app

ENV upstream_url=http://127.0.0.1:8082
ENV mode=http

ENV fprocess="java -jar tmajava.jar"
EXPOSE 8080

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
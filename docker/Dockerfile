FROM eclipse-temurin:17 AS builder
WORKDIR application
ARG ARTIFACT_NAME
COPY ${ARTIFACT_NAME}.jar application.jar
RUN java -Djarmode=layertools -jar application.jar extract



FROM alpine:latest AS agent
WORKDIR /agent

# curl 설치
RUN apk add --no-cache curl

# OpenTelemetry Java Agent 다운로드
RUN curl -sSL -o opentelemetry-javaagent.jar \
    https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v2.13.0/opentelemetry-javaagent.jar

# Pyroscope OpenTelemetry Extention 다운로든
RUN curl -sSL -o pyroscope-otel.jar \
    https://github.com/grafana/otel-profiling-java/releases/download/v0.11.0/pyroscope-otel.jar

FROM eclipse-temurin:17
WORKDIR application

ARG EXPOSED_PORT
EXPOSE ${EXPOSED_PORT}

ENV SPRING_PROFILES_ACTIVE=docker

COPY --from=builder application/dependencies/ ./

# OpenTelemetry Java Agent 파일 복사
COPY --from=agent agent/opentelemetry-javaagent.jar ./
COPY --from=agent agent/pyroscope-otel.jar ./

# fix for https://stackoverflow.com/questions/51115856/docker-failed-to-export-image-failed-to-create-image-failed-to-get-layer
# (only last copy caused issue)
# this seems to be triggered by using btrfs:
# https://github.com/moby/moby/issues/36573
RUN true
COPY --from=builder application/spring-boot-loader/ ./
RUN true
COPY --from=builder application/snapshot-dependencies/ ./
RUN true
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "-javaagent:/opentelemetry-javaagent.jar org.springframework.boot.loader.launch.JarLauncher"]

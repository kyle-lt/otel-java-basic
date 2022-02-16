# otel-java-basic

## Overview
This project was built in order to easily spin up a handful of services that encapsulate a basic OpenTelemetry pipeline.  There are two app services that are spun up, and they are already instrumented.  `otel-java-basic` is instrumented with the [OpenTelemetry Java Instrumentation](https://github.com/open-telemetry/opentelemetry-java-instrumentation) JAR, and `appd-java-basic` is instrumented with [AppDynamics Java Instrumentation](https://docs.appdynamics.com/22.2/en/application-monitoring/install-app-server-agents/java-agent) JAR.

Both app services send OpenTelemetry Spans to the [OpenTelemetry Dev Collector](https://hub.docker.com/r/otel/opentelemetry-collector-dev).

With zero configuration, the OpenTelemetry Collector sends Spans to a [Jaeger All-In-One](https://hub.docker.com/r/jaegertracing/all-in-one) backend.

With some configuration, the OpenTelemetry Collector **also** sends Spans to an [AppDynamics](https://docs.appdynamics.com/22.2/en/application-monitoring/ingest-opentelemetry-trace-data) backend.

## Super Quick Basic Start - Try This First!!

1. Clone this repository to your local machine.

```bash
git clone https://github.com/kyle-lt/otel-java-basic.git
```

2. Change into this project's directory

```bash
cd otel-java-basic
```

3. Start the project with Docker Compose

```bash
docker-compose up -d
```

4. Open the **OTel-Instrumented** Spring PetClinic app on `http://$DOCKER_HOSTNAME:8080` (`$DOCKER_HOSTNAME` is generally `localhost`), and click around for a bit.  

5. Open the **AppD-Instrumented** Spring PetClinic app on `http://$DOCKER_HOSTNAME:8081` (`$DOCKER_HOSTNAME` is generally `localhost`), and click around for a bit.  

6. Then, open Jaeger on `http://$DOCKER_HOSTNAME:16686`, choose `otel-java-basic` or `appd-java-basic` from the services drop-down menu, click the "Find Traces" button, and then choose a trace!

> If some of that didn't work, or you want to know more, read on!

> **//TODO** In addition, it has been built toward exporting OpenTelemetry Spans to AppDynamics, so there are components setup to do so, and require some futher configuration to make them work.

The stack consists of:
- Jaeger
- OpenTelemetry Collector (no agent)
- [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) with [OpenTelemetry Java Instrumentation](https://github.com/open-telemetry/opentelemetry-java-instrumentation) JAR
- [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) with [AppDynamics Java Instrumentation](https://docs.appdynamics.com/22.2/en/application-monitoring/install-app-server-agents/java-agent) JAR

More information about the [Compose Services](#docker-compose-services) below.  For the most part, I am just using the latest images available.

   > __Note:__  This project was built/tested only on Docker for Mac, and uses Docker-Compose

It's not necessary to build anything in this project.  All images can be pulled from Docker Hub when you run with [Docker Compose](#quick-start-with-docker-compose).

## Quick Start with Docker Compose
### Prerequisites
In order to run this project, you'll need:
- Docker for Mac
- Docker Compose 
  <br />  

   > __Note:__  The Docker versions must support Docker Compose File version 3.2+

### Steps to Run

   > __Note:__  If only using the Jaeger backend (and not the AppDynamics backend), **only steps 1 and 4** need to be followed! :)

1. Clone this repository to your local machine.
2. Copy the `.env_public` file to a file named `.env` in the root project directory, and configure it appropriately.

   > __IMPORTANT:__ Detailed information regarding `.env` file can be found [below](#env-file).  This __MUST__ be done for this project to work **with the AppDynamics backend!**

3. Copy the `otel-collector-config.yaml` file to a file named `appd-otel-collector-config.yaml` in the root project directory, and configure it appropriately.

   > __IMPORTANT:__ Detailed information regarding `appd-otel-collector-config.yaml` file can be found [below](#appd-otel-collector-config.yaml-file).  This __MUST__ be done for this project to work **with the AppDynamics backend!**

4. Use Docker Compose to start
```bash
$ docker-compose up -d
```

5. Open the **OTel-Instrumented** Spring PetClinic app on `http://$DOCKER_HOSTNAME:8080` (`$DOCKER_HOSTNAME` is generally `localhost`), and click around for a bit.  

6. Open the **AppD-Instrumented** Spring PetClinic app on `http://$DOCKER_HOSTNAME:8081` (`$DOCKER_HOSTNAME` is generally `localhost`), and click around for a bit.

7. Then, open Jaeger on `http://$DOCKER_HOSTNAME:16686`, choose `otel-java-basic` or `appd-java-basic` from the services drop-down menu, click the "Find Traces" button, and then choose a trace!

## Docker Compose Services
### jaeger
Jaeger tracing backend.  

Jaeger "all-in-one" Image, pulled from [`jaegertracing/all-in-one:latest`](https://hub.docker.com/r/jaegertracing/all-in-one).

By default, accessible on `http://$DOCKER_HOSTNAME:16686`.


### otel-collector
OpenTelemetry Collector.

OpenTelemetry development Image, pulled from [`otel/opentelemetry-collector-dev:latest`](https://hub.docker.com/r/otel/opentelemetry-collector-dev).

There is no UI, merely a pipeline that is configured via `appd-otel-collector-config.yaml`.


### otel-java-basic
Spring PetClinic app.

Spring PetClinic Image, pulled from [`kjtully/otel-java-basic:latest`](https://hub.docker.com/r/kjtully/otel-java-basic) Image built from the `otel-trainig` [fork](https://github.com/kyle-lt/spring-petclinic/tree/otel-training) of the Spring PetClinic Repo.

By default, accessible on `http://$DOCKER_HOSTNAME:8080`.


### appd-java-basic
Spring PetClinic app.

Spring PetClinic Image, pulled from [`kjtully/appd-java-basic:latest`](https://hub.docker.com/r/kjtully/appd-java-basic) Image built from the `otel-trainig` [fork](https://github.com/kyle-lt/spring-petclinic/tree/otel-training) of the Spring PetClinic Repo.

By default, accessible on `http://$DOCKER_HOSTNAME:8081`.

## More Notes on Configuration

### docker-compose.yml
This file is located in the project root and manages building and running the Docker containers. It uses the `.env` file to populate environment variables for the project to work properly.

### .env File
This file contains all of the environment variables that need to be populated in order for the project to run, and for the performance tools to operate.  Items that *must* be tailored to your environment are:

#### AppDynamics Controller Configuration
This configuration is used by the AppD Hybrid Java Agent to connect to the AppD Controller of your choice.
```bash
# AppD Agent
APPDYNAMICS_AGENT_ACCOUNT_ACCESS_KEY=<access_key>
APPDYNAMICS_AGENT_ACCOUNT_NAME=<customer1>
APPDYNAMICS_CONTROLLER_HOST_NAME=<controller_host>
APPDYNAMICS_CONTROLLER_PORT=<8090>
APPDYNAMICS_CONTROLLER_SSL_ENABLED=<false_or_true>
APPDYNAMICS_AGENT_APPLICATION_NAME=otel-java-basic
# Tier and Node names are set in docker-compose.yml individually
```  

#### AppDynamics OpenTelemetry Ingestion Service API Key
API Key for the AppDynamics OpenTelemetry Ingestion Service.
```bash
# AppD OTel Ingest API Key
X_API_KEY=<my_x_api_key>
```

### appd-otel-collector-config.yaml File

#### OpenTelemetry Processors
The `value:` sections of the below should be configured appropriately.
```bash
processors:
  resource:
    attributes:
      - key: appdynamics.controller.account
        action: upsert
        value: "my-appd-account"
      - key: appdynamics.controller.host
        action: upsert
        value: "my-controller-host"
      - key: appdynamics.controller.port
        action: upsert
        value: 443
  batch:
  ```

#### OpenTelemetry Exporters
The `endpoint:` section of the below should be configured appropriately.  Note, the `x-api-key` header is defined in the `.env` file.
  ```bash
exporters:
  ...
  otlphttp:
    endpoint: https://my-appd-endpoint.com
    headers: {"x-api-key": "${X_API_KEY}"}
```

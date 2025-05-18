Fix for Docker Monitoring in SigNoz

Overview
SigNoz provides built-in support for monitoring Docker containers through the OpenTelemetry (OTEL) Collector. However, the default configuration in the original setup had two major problems:

Logs from essential Docker containers were being mistakenly filtered out.

The Docker Compose file used outdated versions of some key services, which led to compatibility issues when setting up Docker monitoring.

Problems Identified

1. OTEL Collector Was Filtering Out Important Logs
The configuration file for the OTEL Collector (otel-collector-config.yaml) included a filter that excluded logs from several critical containers. These included logspout, frontend, alertmanager, query-service, otel-collector, clickhouse, and zookeeper.

Here’s a snippet from the original config:

receivers:
  - type: filter
    id: signoz_logs_filter
    expr: 'attributes.container_name matches "^signoz-(logspout|frontend|alertmanager|query-service|otel-collector|clickhouse|zookeeper)"'
Because of this filter, logs from the containers listed above were not being captured.

2. Services Were Running on Outdated Versions
The docker-compose-minimal.yaml file in the clickhouse-setup directory was pointing to older versions of key services, which caused some issues:

query-service:
  image: signoz/query-service:${DOCKER_TAG:-0.56.0}

frontend:
  image: signoz/frontend:${DOCKER_TAG:-0.56.0}
These outdated versions were not fully compatible with the rest of the setup.

Changes Made

1. Disabled the Log Filter in the OTEL Collector
To ensure all logs are captured, especially from the essential services, the filter section in the OTEL Collector config was commented out:

#   type: filter
#   id: signoz_logs_filter
#   expr: 'attributes.container_name matches "^signoz-(logspout|frontend|alertmanager|query-service|otel-collector|clickhouse|zookeeper)"'

2. Updated the Versions of Core Services
The query-service and frontend containers were upgraded to the latest stable release (version 0.73.0):

query-service:
  image: signoz/query-service:${DOCKER_TAG:-0.73.0}

frontend:
  image: signoz/frontend:${DOC

Outcome:

Logs from all essential Docker containers are now being collected and shown in SigNoz.
The OTEL Collector is no longer excluding important logs.
Upgrading to the latest service versions has improved system compatibility and performance.KER_TAG:-0.73.0}


  
# signoz
signoz demo with docker-compose 

```bash
ansible-playbook up.yml
```

This does docker compose up on an "empty" signoz in a codespace. Use this as a starting point for instrumenting your app.

```bash
ansible-playbook down.yml
```

This does docker compose down on the clickhouse-setup/docker-compose-minimal.yaml (the same docker-compose file from up.yml)

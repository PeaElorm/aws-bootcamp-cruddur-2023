# Week 2 — Distributed Tracing

## Sending Data To HoneyComb
1. Open your repo in gitpod and run the following in your terminal...be sure you are in the aws-bootcamp-cruddur-2023/frontend-react-js directory
```yaml
export HONEYCOMB_API_KEY="IGIb5LZ7SNpJVAUmqlZhaC"
gp env HONEYCOMB_API_KEY="IGIb5LZ7SNpJVAUmqlZhaC"

export  HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

Add the following Env Vars to the backend-flask in docker compose
```yaml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

1. Install packages
```yaml
pip install opentelemetry-api \
    opentelemetry-sdk \
    opentelemetry-exporter-otlp-proto-http \
    opentelemetry-instrumentation-flask \
    opentelemetry-instrumentation-requests
``` 

# Week 2 — Distributed Tracing

## Sending Data To HoneyComb
1. Open your repo in gitpod and run the following in your terminal...be sure you are in the aws-bootcamp-cruddur-2023/frontend-react-js directory. The API key is from the honeycomb account.
```yaml
export HONEYCOMB_API_KEY="<API KEY>"
gp env HONEYCOMB_API_KEY="<API KEY>"

export  HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

Add the following Env Vars to the backend-flask in docker compose
```yaml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

1. Install packages; Install these packages to instrument a Flask app with OpenTelemetry. Add the following to the requirement.txt file:
```yaml
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
``` 

pip install -r requirements.txt in your backend-flask directory

2. Initializing
Add the following to your existing Flask app initialization file app.py (or similar). These updates will create and initialize a tracer and Flask instrumentation to send data to Honeycomb:
Updates to the app.py file
```yaml
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
Initialize tracing and an exporter that can send data to Honeycomb;
```yaml
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

# Initialize automatic instrumentation with Flask
```yaml
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

3. Running docker compose up and check honeycomb to verify traces
4. Here is my output in Honeycomb
![traces1](https://user-images.githubusercontent.com/68542385/223675765-07fdbc3a-0f35-42a5-a26f-c87e9baf1db5.PNG)
![traces 2](https://user-images.githubusercontent.com/68542385/223675792-f69b1e99-2a33-48f9-9371-2ebec6cae364.PNG)


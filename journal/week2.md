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
4. Run the backend and refresh
5. Here is my output in Honeycomb
![traces1](https://user-images.githubusercontent.com/68542385/223675765-07fdbc3a-0f35-42a5-a26f-c87e9baf1db5.PNG)
![traces 2](https://user-images.githubusercontent.com/68542385/223675792-f69b1e99-2a33-48f9-9371-2ebec6cae364.PNG)


## Distributed Tracing with AWS X-ray
1. To work with AWS X-ray, the following should be added to the requirement.txt file to download the needed SDK
```yaml
aws-xray-sdk
```
2. Add the following to the docker compose file
```yaml 
version: "3.8"
services:
    backend-flask:
        environment:
            ...
            AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
            AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
          
    ...
    xray-daemon:
        image: "amazon/aws-xray-daemon"
        environment:
            AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
            AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
            AWS_REGION: "us-east-1"
        command:
            - "xray -o -b xray-daemon:2000"
        ports:
            - 2000:2000/udp
```           
3 This sets our required environment variables and defines a container for the x-ray daemon
4. Next was to group our traces on aws x-ray, The following command as issued in the terminal. PS: I already have the aws cli installed.
```yaml 
$ aws xray create-group \
> ---group-name "Cruddur" \
> --filter-expression "service(\"backend-flask\")"
```
5. This returned my aws parameters, indicating success
6. Next, we need to create a smapling rule for the xray services; to do this we have to create an xray.json file in aws-bootcamp-cruddur-2023/aws/json
```yaml
{
"SamplingRule": {
    "RuleName": "Cruddur",
    "ResourceARN": "*",
    "Priority": 9000,
    "FixedRate": 0.1,
    "ReservoirSize": 5,
    "ServiceName": "backend-flask",
    "ServiceType": "*",
    "Host": "*",
    "HTTPMethod": "*",
    "URLPath": "*",
    "Version": 1
    }
}
```
7. Input the following in the terminal;
```yaml
$ aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
8. Next, we add the following to the app.py;
```yaml
...
...

# X_RAY
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
#xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
xray_recorder.configure(service='backend-flask') # To ensure all traces can be grouped under the Cruudr group created

......
......

app = Flask(__name__)

#XRAY
XRayMiddleware(app, xray_recorder)
``` 




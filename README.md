# Serverless Face Recognition Pipeline (AWS Lambda + SQS + ECR)

Two stage serverless pipeline for face detection and identity recognition on video frames. A client submits frames to a detection function, detected faces are passed via SQS to a recognition function, and the client retrieves results from a response queue. Packaged as Lambda container images in ECR to handle ML dependencies cleanly.

> Source code is not public. This repository documents architecture, interfaces, and operational behavior.

---

## System overview

- Ingress: HTTP POST to `face-detection` (Lambda Function URL)
- Stage 1: `face-detection` runs MTCNN and publishes detected faces to a request queue
- Stage 2: `face-recognition` is triggered by the request queue, computes embeddings with InceptionResnetV1 (vggface2), matches against reference embeddings, and publishes results to a response queue
- Egress: client polls the response queue for request_id plus result

---

## Architecture

### Components
- Lambda functions: `face-detection`, `face-recognition`
- SQS queues: request queue (detection to recognition), response queue (recognition to client)
- Deployment: Lambda container images stored in ECR
- Observability: CloudWatch logs

~~~mermaid
flowchart LR
  Client[Client or IoT camera] -->|POST video frame| FD[Lambda face-detection]
  FD -->|faces plus request_id| REQ[Request queue]
  REQ -->|triggers| FR[Lambda face-recognition]
  FR -->|request_id plus result| RESP[Response queue]
  Client <-->|poll| RESP

  ECR[ECR container image] --> FD
  ECR --> FR
  FD --> CW[CloudWatch logs]
  FR --> CW
~~~

---

## Interface contract

Client to face-detection (HTTP POST)

The request body contains:

content: base64 encoded image bytes  
request_id: unique ID for correlation  
filename: input image name  

face-detection extracts these fields from the JSON formatted body in the Lambda event.

face-detection to request queue

face-detection publishes detected faces along with request_id to the request queue to trigger recognition.

face-recognition to response queue

face-recognition publishes a dict to the response queue:

~~~json
{ "request_id": "<id>", "result": "<classification result>" }
~~~

---

## Implementation notes

Loose coupling via SQS

Detection and recognition are decoupled through queues so each stage can scale and fail independently.

Request correlation

request_id is preserved across stages for end to end tracing and correct result matching.

Containerized deployment via ECR

Both functions are deployed as container images in ECR to package ML dependencies reliably.

---

## Technologies

Core

Python  
Docker (Lambda container images)  

AWS Services

AWS Lambda (Function URL ingress, SQS triggered execution)  
Amazon SQS (request queue triggers recognition, response queue polled by client)  
Amazon ECR (container image registry for Lambda images)  
Amazon CloudWatch (logs and diagnostics)  

Libraries

facenet-pytorch (MTCNN face detection)  
boto3 (AWS SDK)  
awslambdaric (Lambda runtime interface client)  
Pillow, opencv-python, requests (image handling and utilities)  

---

## Benchmark (representative run)

Workload

100 requests (video frames)

Metric

Time from HTTP POST accepted by face-detection to result available in the response queue for the same request_id

Results

Requests completed: 100/100  
Failed requests: 0  
Average end to end latency: X.XX seconds  

Notes

Latency depends on image size, Lambda memory allocation (CPU scales with memory), cold start rate, and queue depth.

---

## Operational notes

Queue hygiene and cost control

During development, ensure messages are deleted after processing so you do not accidentally retrigger functions and incur unnecessary invocations.

Monitoring and debugging

CloudWatch logs are used for:

request tracing by request_id  
inference failures and timeouts  
throughput and latency debugging  

---

## Repository notes

This repository is intended to present the architecture and design of the project only. Source code is not shared online. If you’d like to review implementation details, please contact me.
```

# Serverless Face Detection Pipeline

This project demonstrates a serverless pipeline for face detection using AWS Lambda, SQS, and ECR. It was built to support stateless, high-throughput processing of image events while maintaining low operational overhead.

## Architecture Overview

- Images are pushed to an SQS queue.
- A containerized Lambda function, deployed via ECR, is triggered for each message.
- The Lambda function performs face detection using the MTCNN model and logs results to CloudWatch or stores them in an object store.

## Technologies

- Python, Docker
- AWS Lambda, SQS, ECR, CloudWatch
- facenet-pytorch, boto3

## Features

- Sub-2-second average response time across 100 concurrent invocations
- Retry handling and failure isolation using SQS
- Minimal cold start times via provisioned concurrency and container-based deployment

## Notes

This repository is intended to present the architecture and design of the project only. Source code is not shared online due to course guidelines. Please contact me for the code.

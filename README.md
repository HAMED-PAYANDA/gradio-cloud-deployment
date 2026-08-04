
# ☁️ Serverless Gradio Deployment on IBM Code Engine

An end-to-end DevOps pipeline for containerizing a Gradio UI web application and deploying it to IBM Cloud Code Engine using Docker and serverless architecture.

[![Python](https://img.shields.io/badge/Python-3.10-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Gradio](https://img.shields.io/badge/Gradio-5.23.2-FF5722?style=for-the-badge&logo=gradio&logoColor=white)](https://gradio.app/)
[![IBM Cloud](https://img.shields.io/badge/IBM_Cloud-Code_Engine-052FAD?style=for-the-badge&logo=ibm&logoColor=white)](https://cloud.ibm.com/)
[![Status](https://img.shields.io/badge/Status-Deployed_&_Verified-success?style=for-the-badge)](#-deployment-verification)

</div>

---

## 📌 Project Overview

This repository demonstrates a complete cloud deployment pipeline for containerizing Python web applications and hosting them on **IBM Cloud Code Engine**. 

By leveraging serverless container technology, the application eliminates manual virtual machine configuration, automatically manages container image builds directly from local source files, stores them inside the **IBM Cloud Container Registry (ICR)**, and provisions a secure HTTPS public endpoint with custom port forwarding (`7860`).

---

## 🏗️ Architecture & Deployment Workflow

```text
┌─────────────────────────┐       ┌──────────────────────────┐       ┌─────────────────────────────┐
│ Local Source Directory  │ ────> │  IBM Code Engine Build   │ ────> │ IBM Container Registry (ICR)│
│ (demo.py / Dockerfile)  │       │  (build-local-docker)    │       │ (us.icr.io/${NAMESPACE})    │
└─────────────────────────┘       └──────────────────────────┘       └──────────────┬──────────────┘
                                                                                    │
                                                                                    ▼
┌─────────────────────────┐                                          ┌─────────────────────────────┐
│  Live Public App URL    │ <─────────────────────────────────────── │ Serverless Application      │
│  (HTTPS Endpoint:7860)  │                                          │ (IBM Code Engine Container) │
└─────────────────────────┘                                          └─────────────────────────────┘

1.	Local Application Specs: Develop application script (demo.py), specify dependencies (requirements.txt), and construct image build instructions (Dockerfile).
	2.	Context Target & Project Selection: Target active IBM Cloud resource groups (production) and select the Code Engine project instance.
	3.	Automated Image Build: Upload source files directly to IBM Code Engine, compile the Docker container image, and push artifacts to the private IBM Container Registry (ICR).
	4.	Serverless Application Deployment: Instantiate container demo1 with 2GB ephemeral storage, minimum scale set to 1, exposing port 7860 over public HTTPS.
🛠️ Tech Stack & Tools
Category	Technology	Purpose
Application UI	Gradio v5.23.2	Rapid web application interface framework
Language Runtime	Python 3.10	Core application execution environment
Container Runtime	Docker	Blueprint definition and containerization
Serverless Platform	IBM Cloud Code Engine	Fully managed containerized workload hosting
Image Registry	IBM Cloud Container Registry (ICR)	Enterprise private image repository
CLI Management	IBM Cloud CLI (ibmcloud ce)	Command-line project orchestration and builds
📁 Project Structure
gradio-cloud-deployment/
├── myapp/
│   ├── demo.py               # Main Gradio application code
│   ├── Dockerfile            # Multi-stage Docker container specification
│   └── requirements.txt      # Application dependencies
└── README.md                 # Project documentation

⚙️ Step-by-Step Implementation
1. Application Development & Container Blueprint
Create project folder and files:
mkdir myapp && cd myapp
touch demo.py Dockerfile requirements.txt

requirements.txt
requests
gradio==5.23.2

demo.py
import gradio as gr

def greet(name, intensity):
    return "Hello, " + name + "!" * int(intensity)

demo = gr.Interface(
    fn=greet,
    inputs=["text", "slider"],
    outputs=["text"],
)

demo.launch(server_name="0.0.0.0", server_port=7860)

Dockerfile
FROM python:3.10

WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip3 install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "demo.py"]

2. IBM Code Engine Orchestration
Step A: Target Resource Group & Project Context
# Target the active resource group
ibmcloud target -g production

# Select the assigned Code Engine project
ibmcloud ce project select -n "Code Engine - sn-labs-hamedpayanda"

Step B: Build Container Image from Local Source
# Create build configuration pointing to IBM Container Registry
ibmcloud ce build create --name build-local-dockerfile1 \
                         --build-type local --size large \
                         --image us.icr.io/${SN_ICR_NAMESPACE}/myapp1 \
                         --registry-secret icr-secret

# Submit build run with current directory source context
ibmcloud ce buildrun submit --name buildrun-local-dockerfile1 \
                            --build build-local-dockerfile1 \
                            --source .

Verify build status:
ibmcloud ce buildrun get -n buildrun-local-dockerfile1

Step C: Deploy Serverless Container
ibmcloud ce application create --name demo1 \
                               --image us.icr.io/${SN_ICR_NAMESPACE}/myapp1 \
                               --registry-secret icr-secret \
                               --es 2G \
                               --port 7860 \
                               --minscale 1

Retrieve public URL endpoint:
ibmcloud ce app get --name demo1 --output url

📸 Deployment Verification
The screenshot below confirms successful build execution (Status: succeeded), image push to the registry, serverless app instantiation, and live rendering of the Gradio interface over a public IBM Code Engine URL endpoint:
![Gradio Web App Interface](screenshot.png)  




•	Build Run ID: c77307fb-430a-4f3e-b902-6e93e1ec1410
•	Build Status: succeeded
•	Port Target: 7860
•	Ephemeral Storage: 2G
•	Active Public Endpoint: https://demo1.2d36vojef1v4.us-south.codeengine.appdomain.cloud

---

👤 Author
Hamed Payanda
•	GitHub: @HAMED-PAYANDA
Completed as part of the IBM CognitiveClass.ai / Skills Network Cloud Engineering Curriculum.

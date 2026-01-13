# MLOps & LLMOps Engineering Excellence

This repository serves as a comprehensive technical guide and implementation framework for MLOps (Machine Learning Operations).


---

## 💻 CI/CD: DevOps vs. MLOps

MLOps introduces a "Third Pillar" to traditional CI/CD: **Continuous Training (CT)**. Unlike standard software, ML performance depends on code, data, and the model artifact.

| Feature | Traditional DevOps | MLOps |
| :--- | :--- | :--- |
| **Core Artifact** | Code & Configuration | Code, Data, & Model |
| **Triggers** | Git Commits | Code changes, Data changes, Performance decay |
| **Testing** | Unit & Integration tests | Data validation, Model quality, Bias detection |
| **Version Control** | Code versioning (Git) | Versioning for Code (Git), Data (DVC/Feature Stores), and Models (Model Registry, e.g., MLflow) |

---

## 🚀 The MLOps CI/CD is multi-trigger
Code change: CI -> CT ->CD: </br>
Data Change: CT -> CD: New data arrives (e.g., a nightly job aggregates new customer data) </br>
Model Degradation: CT -> CD: </br>


### CI – The Core Trigger Mechanism: The Git Webhook
When a developer (or data scientist) pushes new code to the central repository (e.g., a git push to the main or a feature branch), the following happens:
1.	The Event: The version control system (like GitHub, GitLab, or Azure DevOps) detects the new commit/merge.
2.	The Notification (Webhook): The system immediately sends an automated message, called a webhook, to the central CI/CD orchestrator (e.g. GitHub Actions, Jenkins, or Kubeflow).
3.	The Activation: The orchestrator receives the signal and initiates the first stage of the pipeline (Continuous Integration).

Please check MLOPS\CI-workflow.yml.<br>
This CI workflow is now ready to be saved inside the. github/workflows/ directory of your GitHub repository. 

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]



### It primarily consists of 4 jobs.

### 🛠 Job 1: Unit Testing and Model Performance Check

Runs the test suite across multiple Python versions. Evaluate Model Performance (Precision/Recall Check) <br>
Please check MLOPS\model-performance-pytest.py <br>
        # CRITICAL: python test must use assertions <br>
        # (e.g., 'assert precision >= 0.85') to fail the step if the metric threshold is not met. <br>

In CI-workflow.yml we have below line      </br> 
run: pytest tests/model-performance-pytest.py --cov=src/ --cov-report=xml <br>

--cov=src/ --cov-report=xml <br>
This ensures that while you are running a performance test, you are measuring how much of your source code is being exercised by that test. <br>

<img width="805" height="191" alt="image" src="https://github.com/user-attachments/assets/e38b8246-8192-4aa9-ac61-90108f78ab06" />

run: pytest --cov=./ --cov-report=xml"
The above will run pytest on entire folder.

---

##  📈 Job 2: Build and Push Docker Image
Builds a Docker image, tagging it with the commit SHA and the latest tag (only on pushes to main).<br>
<img width="738" height="232" alt="image" src="https://github.com/user-attachments/assets/2e6cd063-d96e-4116-aa8d-b055cdbd2a52" /> <br>

Please see the MLOPS\Dockerfile for more information.<br>
📌 CI Workflow Context <br>
In CI-workflow.yml, the step that builds the image is:<br>

- name: Build and Push Docker image <br>
        uses: docker/build-push-action@v5 <br>
        with:<br>
          context: .  # This is the build context (the root directory) <br>
          # ... <br>


• The line context: . means the Docker daemon looks in the current directory (the repository root) for the build context. <br>
• By default, the docker build command looks for a file named Dockerfile within that context. <br>
You should place the file directly in the main directory: <br>
Deployment Readiness: Pushes the tested and built image to the GitHub Container Registry (ghcr.io), making it ready for deployment to a service like Kubernetes, Google Cloud Run, or AWS ECS.  <br>

---

## 🕵️ Job 3: Continuous Training (CT) - Train and Store New Model
Please see the continuous_training Job in the CI-workflow.yml file.
This job will use a runner to pull the newly built Docker image, execute the training script inside the container, and handle the resulting model artifact.

docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
#### Run the training script inside the container **
          
#### The -v mounts the 'artifacts' directory for the model to be saved locally
  docker run --rm \
     -v ${PWD}/artifacts:/app/artifacts \
     ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest \
     python src/train.py --output-path /app/artifacts/new_model.pkl


## 🕵️ Case Study: Real-Time Fraud Detection
The system is designed for sub-100ms latency using a microservices architecture.
- **Inference Service:** A Python/FastAPI service optimized for throughput.
- **Feature Store:** Redis/Hopsworks for low-latency retrieval of user transaction history.
- **Troubleshooting:** If performance drops, we analyze for **Covariate Drift** (input distribution changes) or **Concept Drift** (the definition of fraud has evolved).

---

## 🤖 The LLMOps Pipeline
LLMOps extends MLOps to handle the unique challenges of non-deterministic models.



### Evaluation & Guardrails
- **LLM-as-a-Judge:** Using frameworks like **Ragas** or **DeepEval** to have a strong model (e.g., GPT-4) grade a smaller model (e.g., Llama-3).
- **Metrics:** Faithfulness, Relevance, and Tone.
- **Guardrails:** Implementation of **NVIDIA NeMo Guardrails** to prevent PII leakage and toxicity.

### Recommended Stack
| Component | Tools |
| :--- | :--- |
| **Orchestration** | LangChain, LlamaIndex |
| **Vector DB** | Pinecone, Milvus, ChromaDB |
| **Serving** | vLLM, TGI, Ollama |
| **Monitoring** | Arize Phoenix, Weights & Biases |

---

# Contextual Chatbot IRT

**Contextual Chatbot IRT** is an intelligent question-answering system that performs local semantic search on documents stored in Amazon S3. It uses a `SentenceTransformer` model to embed both the knowledge base and incoming user queries, returning the most relevant information directly from the stored documents. Designed for internal knowledge retrieval, this serverless chatbot can be deployed via AWS Lambda and integrated with a static frontend.

---

## 🔍 How It Works

```
User Query
   ↓
Lambda (app.py)
   ↓
SentenceTransformer (local inference)
   ↓
Cosine Similarity Search
   ↓
Best Matching Chunk (from S3-based text)
   ↓
Chatbot Response
```

---

## 📦 Pipeline Components

- **S3 Document Storage:** A `.txt` file (`knowledge_base_info.txt`) is stored in an S3 bucket.
- **Model Retrieval from S3:** A pre-trained `SentenceTransformer` model is also stored in the S3 bucket and downloaded into the Lambda container on cold start.
- **Local Embedding & Search:**
  - The text file is chunked using paragraph breaks.
  - Embeddings are generated using the locally loaded model.
  - A user query is embedded and compared with all chunks using cosine similarity.
- **Lambda Inference:** All logic is handled by the `app.py` Lambda function running in a containerized environment.

---

## 🧱 Tech Stack

- **AWS Lambda (Container):** Backend inference engine.
- **Amazon S3:** Stores model and knowledge base text.
- **Python / Boto3 / SentenceTransformers / Torch:** Embedding and semantic search.
- **Docker:** Containerization of the Lambda function.
- **Terraform:** Infrastructure as code to deploy Lambda, S3, and API Gateway.

---

## 🚀 Getting Started

### 1. Clone the Repository
```bash
git clone https://github.com/Mustafa3946/contextual-chatbot-irt.git
cd contextual-chatbot-irt
```

### 2. Configure AWS Credentials
Ensure the AWS CLI is authenticated with a user/role that can access S3 and deploy Lambda.

### 3. Upload Resources to S3
- Upload your `knowledge_base_info.txt` to `s3://s3-contextual-chatbot-irt-docs/`.
- Upload the SentenceTransformer model files under the prefix `model/`.

### 4. Build and Push the Lambda Container Image
```bash
cd backend
docker build -t chatbot-lambda .
aws ecr create-repository --repository-name chatbot-lambda  # If not already created
docker tag chatbot-lambda:latest <your-ecr-uri>:latest
docker push <your-ecr-uri>:latest
```

### 5. Deploy Infrastructure with Terraform
```bash
cd terraform
terraform init
terraform apply
```

### 6. Start Querying
Send POST requests to the deployed API Gateway endpoint with a JSON body:
```json
{
  "message": "What is the purpose of the IRT project?"
}
```

---

## 📂 Project Structure

```
contextual-chatbot-irt/
├── frontend/               # Static chat UI (hosted on GitHub Pages)
│   └── index.html
├── backend/                # Lambda container backend
│   ├── app.py              # Lambda handler (semantic search)
│   ├── requirements.txt    # Python dependencies
│   ├── download_model.py   # Pre-download model (optional for local dev)
│   └── Dockerfile          # Lambda container image
├── terraform/              # Terraform infrastructure scripts
│   ├── main.tf
│   ├── s3.tf
│   ├── lambda.tf
│   └── api_gateway.tf
├── documents/              # Source document(s)
│   └── knowledge_base_info.txt
└── scripts/                # Future utilities (e.g., batch preprocessing)
    └── ingest_docs.py      # [Optional] for document ingestion
```

---

## 🛡️ Security & Privacy

- All data and models are stored privately in your AWS S3 bucket.
- Lambda runs in an isolated VPC (optional) for added security.
- No third-party API calls or LLMs are used; inference is fully local.

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

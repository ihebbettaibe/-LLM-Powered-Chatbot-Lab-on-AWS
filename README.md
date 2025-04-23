# 🤖 LLM-Powered Chatbot Lab on AWS

> Build an intelligent document Q&A system powered by Large Language Models, Vector Databases, and AWS services.

## 📋 Overview

This comprehensive lab guides you through building a production-ready, LLM-powered chatbot that can intelligently answer questions from your PDF documents. The solution leverages AWS's powerful cloud infrastructure and AI services to create a scalable, secure, and responsive question-answering system.

**Key Features:**
- 💬 Natural language Q&A from PDF content
- 🔍 Semantic search using vector embeddings
- ☁️ Serverless, scalable AWS architecture
- 🧠 Advanced LLM reasoning with Claude and Titan models
- 🔒 Enterprise-grade security with IAM and encryption

## 🧰 Prerequisites

Before starting, ensure you have:

* **AWS Account** with appropriate permissions
* **Ubuntu 24.04 VM** on AWS (t3.medium or better recommended)
* **Python 3.8+** installed
* **AWS CLI** installed and configured with admin access
* **AWS Bedrock** access enabled for:
  * **Titan Embeddings v1** - For generating document vectors
  * **Claude v2** - For intelligent question answering

> 💡 **Tip:** If you don't have access to these models in Bedrock, request access through the AWS Console under the Bedrock service.

## ⚙️ Environment Setup

### Clone the Repository
```bash
git clone https://github.com/yourusername/aws-llm-chatbot-lab.git
cd aws-llm-chatbot-lab
```

### Install Dependencies
```bash
# Create and activate virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate

# Install required packages
pip3 install -r requirements.txt
```

### Required Python Packages
```
boto3>=1.26.0
langchain>=0.0.267
streamlit>=1.24.0
opensearch-py>=2.2.0
pypdf>=3.12.0
```

## 🔄 Architecture Workflow

Our chatbot follows this processing flow:

1. **Document Processing** - PDFs are uploaded, parsed, and chunked
2. **Vector Embedding** - Text chunks are converted to numerical vectors
3. **Vector Storage** - Embeddings are indexed in OpenSearch
4. **Query Processing** - User questions are vectorized and matched
5. **Response Generation** - LLM synthesizes answers based on relevant context

## 🚀 Implementation Steps

### ☁️ Step 1: Upload PDFs to S3

Create an S3 bucket and upload your PDF files:

```bash
python3 create-S3-and-put-docs.py \
  --bucket_name <YourBucketName> \
  --local_path <PathToYourPDFFiles>
```

**Parameters:**
- `bucket_name`: Globally unique S3 bucket name
- `local_path`: Directory containing your PDF files

> 📌 **Note:** The script automatically creates the bucket if it doesn't exist and handles the upload of multiple PDFs.

### 📊 Step 2: Create a Vector Store

Set up an OpenSearch vector database collection:

```bash
python3 create-vector-db.py \
  --collection_name <CollectionName> \
  --iam_user <YourIAMUser> \
  --account_id <YourAccountID>
```

**Under the hood:**
- Creates necessary IAM roles and policies
- Configures secure access with AWS-managed KMS encryption
- Sets up the OpenSearch collection with appropriate dimensions
- Returns the endpoint URL for your vector database

> ⚠️ **Important:** Save the endpoint URL returned by this script as you'll need it in subsequent steps.

### 🧠 Step 3: Vectorize PDF Files

Process your documents and store embeddings:

```bash
python3 vectorise-store.py \
  --bucket_name <YourBucketName> \
  --endpoint <VectorDBEndpoint> \
  --index_name <IndexName> \
  --local_path <LocalPath> \
  --chunk_size 1000 \
  --chunk_overlap 200
```

**Process:**
1. Downloads PDFs from S3 bucket
2. Splits documents into semantic chunks
3. Uses Titan Embeddings to generate vectors
4. Creates and configures the OpenSearch index
5. Stores the vectors with their source metadata

**Optional Parameters:**
- `chunk_size`: Text characters per chunk (default: 1000)
- `chunk_overlap`: Overlap between chunks (default: 200)
- `embedding_dimension`: Size of embedding vectors (default: 1536)

> 💡 **Performance Tip:** Adjust chunk size and overlap based on your document complexity. Smaller chunks improve retrieval precision but may lose context.

### 🌐 Step 4: Deploy the Chatbot

#### 🔐 Pre-Deployment Configuration

1. **Create an EC2 Key Pair** for secure SSH access:
   ```bash
   aws ec2 create-key-pair --key-name ChatbotKeyPair --query 'KeyMaterial' --output text > ChatbotKeyPair.pem
   chmod 400 ChatbotKeyPair.pem
   ```

2. **Create a Security Group** with proper access rules:
   ```bash
   aws ec2 create-security-group \
     --group-name ChatbotSecurityGroup \
     --description "Security group for LLM Chatbot" \
     --vpc-id <YourVpcId>
     
   # Allow SSH, HTTP, HTTPS and Streamlit port
   aws ec2 authorize-security-group-ingress \
     --group-id <SecurityGroupId> \
     --protocol tcp \
     --port 22 \
     --cidr 0.0.0.0/0
     
   aws ec2 authorize-security-group-ingress \
     --group-id <SecurityGroupId> \
     --protocol tcp \
     --port 8501 \
     --cidr 0.0.0.0/0
   ```

3. **Create a `config.ini` file** with your AWS credentials:
   ```ini
   [aws]
   aws_access_key_id = YOUR_AWS_ACCESS_KEY_ID
   aws_secret_access_key = YOUR_AWS_SECRET_ACCESS_KEY
   region = us-east-1
   
   [opensearch]
   endpoint = YOUR_OPENSEARCH_ENDPOINT
   index_name = YOUR_INDEX_NAME
   
   [bedrock]
   embedding_model = amazon.titan-embed-text-v1
   llm_model = anthropic.claude-v2
   
   [app]
   app_title = "Document Q&A Chatbot"
   app_subtitle = "Ask questions about your PDFs"
   debug_mode = False
   ```

#### 🚀 Launch EC2 Instance

Deploy your chatbot to an EC2 instance:

```bash
python3 create_instance.py \
  --ami_id "ami-08919ae65ab65be94" \
  --instance_type "t3.medium" \
  --key_pair_name <KeyPairName> \
  --security_group_id <SecurityGroupID> \
  --iam_role "ChatbotRole" \
  --user_data_script "startup.sh"
```

**Deployment Process:**
- Provisions an EC2 instance with Ubuntu 24.04
- Automatically installs dependencies on startup
- Configures AWS credentials and environment
- Launches the Streamlit web interface

> ⏱️ **Note:** Allow 3-5 minutes for the instance to fully initialize and start the application.

### 💬 Step 5: Interact with the Chatbot

Once the EC2 instance is running:

1. Access the chatbot interface at:
   ```
   http://<PublicIP-of-your-instance>:8501
   ```

2. The interface allows you to:
   - Ask natural language questions about your documents
   - View source references for each answer
   - Adjust model parameters for response quality
   - Upload additional documents through the web UI

## 📁 Project Structure

```
chatbot-lab/
├── README.md                    # This documentation
├── requirements.txt             # Python dependencies
├── create-S3-and-put-docs.py    # S3 bucket creation and document upload
├── create-vector-db.py          # OpenSearch setup script
├── vectorise-store.py           # Document processing and embedding
├── create_instance.py           # EC2 deployment script
├── startup.sh                   # EC2 initialization script
├── chatbot.py                   # Streamlit application
├── utils/
│   ├── document_processor.py    # PDF parsing utilities
│   ├── vector_store.py          # OpenSearch interface
│   └── llm_interface.py         # Bedrock models interface
├── config/
│   ├── config_template.ini      # Configuration template
│   └── logging_config.py        # Logging setup
└── tests/                       # Unit and integration tests
```

## 🔧 Advanced Configuration

### Customizing the Vector Database

Modify the OpenSearch configuration for improved performance:

```bash
python3 create-vector-db.py \
  --collection_name <CollectionName> \
  --vector_dimension 1536 \
  --index_nodes 2 \
  --node_type "m6g.large.search" \
  --iam_user <YourIAMUser> \
  --account_id <YourAccountID>
```

### Optimizing Embedding Generation

For larger document sets, parallelize the embedding process:

```bash
python3 vectorise-store.py \
  --bucket_name <YourBucketName> \
  --endpoint <VectorDBEndpoint> \
  --index_name <IndexName> \
  --local_path <LocalPath> \
  --batch_size 10 \
  --max_workers 4
```

## 🧹 Cleanup

To avoid ongoing AWS charges, remember to delete all resources when finished:

```bash
# Delete EC2 instance
aws ec2 terminate-instances --instance-ids <instance-id>

# Delete S3 bucket and contents
aws s3 rb s3://<YourBucketName> --force

# Delete OpenSearch domain
aws opensearch delete-domain --domain-name <DomainName>

# Delete IAM roles and policies
aws iam detach-role-policy --role-name ChatbotRole --policy-arn <PolicyArn>
aws iam delete-role --role-name ChatbotRole
```

## 🔒 Security Best Practices

This implementation follows AWS security best practices:
- IAM roles with least privilege principle
- KMS encryption for data at rest
- Security groups limiting network access
- No hard-coded credentials in code

## 📚 Resources

- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock)
- [OpenSearch Developer Guide](https://opensearch.org/docs/latest/)
- [LangChain Framework](https://python.langchain.com/docs/get_started/introduction)
- [Streamlit Documentation](https://docs.streamlit.io/)

## 📞 Support

For issues or questions:
- Open an issue on GitHub
- Contact the maintainer at [your-email@example.com](mailto:your-email@example.com)

---

*Happy building! May your chatbot be intelligent and your AWS bill reasonable.* 😊

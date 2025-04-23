🤖 LLM-Powered Chatbot Lab on AWS
This lab walks you through building a Large Language Model (LLM)-based chatbot that can answer questions from PDF files using AWS services like S3, OpenSearch, and Bedrock (Titan + Claude). You'll upload documents, generate embeddings, store them in a vector database, and deploy a chatbot app to interact with them.
🧰 Prerequisites

Ubuntu 24.04 VM on AWS
Python 3.8 or higher
AWS CLI installed and configured
Access to the following models via AWS Bedrock:

Titan Embeddings v1
Claude v2



⚙️ Environment Setup

Clone or copy the chatbot lab folder into your VM.
Install required dependencies:

bashpip3 install -r requirements.txt
☁️ Step 1: Upload PDFs to S3
Create an S3 bucket and upload your PDF files:
bashpython3 create-S3-and-put-docs.py \
--bucket_name <YourBucketName> \
--local_path <PathToYourPDFFiles>
📌 Replace placeholders with your actual bucket name and local file path.
📊 Step 2: Create a Vector Store
Create a vector database collection on OpenSearch:
bashpython3 create-vector-db.py \
--collection_name <CollectionName> \
--iam_user <YourIAMUser> \
--account_id <YourAccountID>
This script will:

Set up IAM access
Configure policies and encryption
Return your vector store endpoint

🧠 Step 3: Vectorize PDF Files
Ensure you have access to Titan Embedding v1 and Claude v2 in Bedrock.
Run:
bashpython3 vectorise-store.py \
--bucket_name <YourBucketName> \
--endpoint <VectorDBEndpoint> \
--index_name <IndexName> \
--local_path <LocalPath>
This script:

Downloads PDFs from S3
Splits them into chunks
Generates embeddings
Creates an index in OpenSearch
Stores the vectors

🌐 Step 4: Deploy the Chatbot
🔐 Pre-Deployment

Create a Key Pair for SSH access.
Create a Security Group:

Allow inbound: SSH, HTTP/HTTPS, and Port 8501
Allow outbound: All traffic


Create a config.ini file in the same directory:

ini[aws]
aws_access_key_id = YOUR_AWS_ACCESS_KEY_ID
aws_secret_access_key = YOUR_AWS_SECRET_ACCESS_KEY
region = us-east-1

[opensearch]
endpoint = YOUR_OPENSEARCH_ENDPOINT
index_name = YOUR_INDEX_NAME
🚀 Launch EC2 Instance
Run the deployment script:
bashpython3 create_instance.py \
--ami_id "ami-08919ae65ab65be94" \
--key_pair_name <KeyPairName> \
--security_group_id <SecurityGroupID>
The instance will:

Inject your credentials and config
Activate the virtual environment
Start the chatbot using Streamlit

💬 Step 5: Interact with the Chatbot
Once the EC2 instance is running, go to:
http://<PublicIP-of-your-instance>:8501
Start asking questions about your PDF documents!
📁 Folder Structure
chatbot-lab/
├── requirements.txt
├── create-S3-and-put-docs.py
├── create-vector-db.py
├── vectorise-store.py
├── create_instance.py
├── chatbot.py
└── config.ini # create this manually
🧹 Cleanup
Remember to delete:

The EC2 instance
S3 bucket
OpenSearch collection
...to avoid unexpected charges on your AWS account.

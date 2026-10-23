**Deploy vLLM**
mkdir -p /root/rag-platform
cd /root/rag-platform
git clone repo

docker compose up -d 
docker compose logs -f llm

**Once the model has loaded, verify the API:**
curl http://127.0.0.1:8000/v1/models

**Then test a chat completion:**
curl http://127.0.0.1:8000/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "Qwen/Qwen2.5-7B-Instruct",
    "messages": [
      {"role": "user", "content": "Explain RAG in two sentences."}
    ],
    "max_tokens": 100
  }'

**  Open the Web-UI URL in your browser:**
http://YOUR_SERVER_IP:3000

**Web-UI Connection to LLM**
Go to Profile name ==> Admin Panel ==>settings ==> connections ==> Manage OpenAI API Connections 
and set value as "http://rag-llm:8000/v1"

**Test:**

Ask a normal question through UI and make sure it is inferencing

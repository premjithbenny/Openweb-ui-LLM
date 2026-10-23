# Deploy vLLM with Docker Compose

Deploy a vLLM inference server and connect it to a Web UI using an OpenAI-compatible API. This setup allows you to run a local LLM and interact with it through a browser.

## Architecture

```text
User
  |
  v
Web UI (Port 3000)
  |
  | OpenAI-compatible API
  v
vLLM Server (Port 8000)
  |
  v
Qwen/Qwen2.5-7B-Instruct
```

## Prerequisites

Before starting, ensure you have:

- A Linux server with Docker installed.
- Docker Compose v2 (`docker compose`).
- Git installed.
- An NVIDIA GPU with compatible NVIDIA drivers and container runtime, if running GPU inference.
- Sufficient RAM, GPU VRAM, and disk space for the selected model.
- A Web UI deployment accessible through your browser.

Verify Docker and GPU availability:

```bash
docker --version
docker compose version
nvidia-smi
```

> **Note:** The commands below assume the repository contains a valid `compose.yaml` or `docker-compose.yml` file that defines the `llm` service and exposes the vLLM API on port `8000`.

## 1. Deploy vLLM

Create the deployment directory:

```bash
mkdir -p /root/rag-platform
cd /root/rag-platform
```

Clone the deployment repository:

```bash
git clone <REPOSITORY_URL> .
```

Replace `<REPOSITORY_URL>` with the actual Git repository URL.

Start the services:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

## 2. Monitor Model Loading

Follow the vLLM container logs:

```bash
docker compose logs -f llm
```

Wait for the model to finish loading and the API server to become ready.

Press `Ctrl+C` to stop following the logs. This does not stop the container.

## 3. Verify the vLLM API

Once the model has loaded, verify that the server responds:

```bash
curl http://127.0.0.1:8000/v1/models
```

A successful response should contain a JSON object listing the model available through the API.

If the API is unavailable, check the service logs:

```bash
docker compose logs --tail=100 llm
```

Confirm that port `8000` is correctly published in the Compose configuration.

## 4. Test Chat Completion

Send a test request to the vLLM chat completions endpoint:

```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "Qwen/Qwen2.5-7B-Instruct",
    "messages": [
      {
        "role": "user",
        "content": "Explain RAG in two sentences."
      }
    ],
    "max_tokens": 100
  }'
```

### Expected result

The server should return a JSON response containing the generated answer under:

```text
choices[0].message.content
```

If the request succeeds, vLLM is serving the model and generating responses.

> **Important:** The model name in the request must match a model ID accepted by your vLLM server. Check `/v1/models` if you receive a model-not-found error.

## 5. Open the Web UI

Open the Web UI in your browser:

```text
http://YOUR_SERVER_IP:3000
```

Replace `YOUR_SERVER_IP` with the IP address or hostname of your server.

Log in using an account with administrator access.

## 6. Connect the Web UI to vLLM

Configure the Web UI to use the vLLM OpenAI-compatible API.

1. Log in to the Web UI.
2. Open **Profile → Admin Panel**.
3. Navigate to **Settings → Connections**.
4. Open **Manage OpenAI API Connections**.
5. Configure the connection with the following values.

| Setting | Value |
|---|---|
| API Base URL | `http://rag-llm:8000/v1` |
| Model | `Qwen/Qwen2.5-7B-Instruct` |
| API Key | Use the configured vLLM API key, if required |

Save the connection and ensure the configured model is available for selection in the Web UI.

### Important: Docker networking

The URL `http://rag-llm:8000/v1` assumes:

- The vLLM container or service is reachable as `rag-llm`.
- The Web UI and vLLM are connected to the same Docker network.
- Port `8000` is the internal API port exposed by the vLLM service.

If the Web UI runs outside that Docker network, use a reachable server address instead, such as:

```text
http://YOUR_SERVER_IP:8000/v1
```

Ensure that network access is appropriately restricted.

## 7. Test End-to-End Inference

After configuring the connection:

1. Open a normal chat in the Web UI.
2. Select the configured Qwen model, if required.
3. Ask a question, for example:

   ```text
   Explain Retrieval-Augmented Generation (RAG) in simple terms.
   ```

4. Confirm that the UI returns a generated response.
5. Check the vLLM logs to verify that the request reaches the inference server:

   ```bash
   docker compose logs -f llm
   ```

**Deployment is successful when:**

- The vLLM container is running.
- The model appears in `/v1/models`.
- The chat completion API returns a generated answer.
- The Web UI successfully sends prompts to vLLM and displays responses.

## 8. Troubleshooting

### vLLM container is not running

```bash
docker compose ps
docker compose logs --tail=200 llm
```

Check the container logs for model download failures, GPU errors, insufficient memory, or configuration issues.

### API is not responding

```bash
curl -v http://127.0.0.1:8000/v1/models
```

Verify that the server has completed startup and that the API port is correctly mapped.

### Web UI cannot connect to vLLM

From the Web UI container, test connectivity:

```bash
docker exec <WEB_UI_CONTAINER> \
  curl -v http://rag-llm:8000/v1/models
```

Replace `<WEB_UI_CONTAINER>` with the actual Web UI container name.

If the connection fails, check Docker network membership and DNS resolution:

```bash
docker network ls
docker inspect <WEB_UI_CONTAINER>
docker inspect <VLLM_CONTAINER>
```

Both containers must share a suitable Docker network for the service hostname to resolve.

### Model loading fails

```bash
docker compose logs --tail=200 llm
nvidia-smi
```

Check GPU availability, VRAM capacity, model download status, and the vLLM configuration.

### Chat completion returns an error

Confirm the available model ID:

```bash
curl -s http://127.0.0.1:8000/v1/models
```

Use the model ID returned by the API in your chat completion request and Web UI configuration.

## 9. Useful Commands

| Operation | Command |
|---|---|
| Start services | `docker compose up -d` |
| Stop services | `docker compose down` |
| Restart services | `docker compose restart` |
| Check status | `docker compose ps` |
| Follow logs | `docker compose logs -f llm` |
| View recent logs | `docker compose logs --tail=100 llm` |
| Verify API | `curl http://127.0.0.1:8000/v1/models` |

## Security Notes

- Do not expose the vLLM API directly to the public internet without appropriate authentication and access controls.
- Restrict access to the Web UI's administrative settings.
- Use firewall rules to limit access to ports `3000` and `8000`.
- Do not commit API keys, passwords, or other secrets to the repository.
- For production deployments, configure persistent storage and suitable resource limits.

## Deployment Summary

The deployment consists of three main steps:

1. Deploy vLLM and load the Qwen model.
2. Verify inference through the OpenAI-compatible API.
3. Configure the Web UI to connect to vLLM and test a normal chat.

Once all three steps succeed, the Web UI is ready to use the deployed LLM for inference.

# Execution Service

FastAPI service for running short code snippets and validating YAML. It supports Python, Java, JavaScript, and YAML/Kubernetes structure validation through a single `/run` API.

## Features

- Executes Python, Java, and JavaScript snippets with a configurable timeout.
- Validates YAML syntax and checks common Kubernetes manifest structure mistakes.
- Exposes health, readiness, liveness, and Prometheus-style metrics endpoints.
- Runs on port `8002` by default.
- Docker image uses a multi-stage build and a non-root runtime user.

## Project Structure

```text
.
+-- Dockerfile
+-- main.py
+-- README.md
+-- requirements.txt
```

## Run Locally

Create and activate a virtual environment:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
pip install -r requirements.txt
```

Start the service:

```powershell
uvicorn main:app --host 0.0.0.0 --port 8002 --reload
```

Open the API docs at:

```text
http://localhost:8002/docs
```

## Environment Variables

| Name | Default | Description |
| --- | --- | --- |
| `EXECUTION_TIMEOUT_SECONDS` | `30` | Maximum runtime for executed code snippets. |
| `LOG_LEVEL` | `INFO` | Python logging level. |

## API Endpoints

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/health` | Basic health check. |
| `GET` | `/ready` | Readiness check with available runtimes. |
| `GET` | `/live` | Liveness check. |
| `GET` | `/metrics` | Request count and total request latency metrics. |
| `POST` | `/run` | Execute or validate code. |

## Run Code

Request body:

```json
{
  "code": "print('hello from execution-service')",
  "tab_id": "example-1",
  "language": "python"
}
```

PowerShell example:

```powershell
Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:8002/run `
  -ContentType "application/json" `
  -Body '{"code":"print(''hello from execution-service'')","tab_id":"example-1","language":"python"}'
```

Response:

```json
{
  "output": "hello from execution-service\n"
}
```

Supported language values include:

- `python` or `py`
- `java`
- `javascript`, `js`, or `node`
- `yaml` or `yml`

## YAML Validation Example

```powershell
$body = @"
{
  "code": "apiVersion: v1\nkind: Pod\nmetadata:\n  name: demo\nspec:\n  containers:\n    - name: app\n      image: nginx",
  "tab_id": "yaml-1",
  "language": "yaml"
}
"@

Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:8002/run `
  -ContentType "application/json" `
  -Body $body
```

## Docker

Build the image:

```powershell
docker build -t execution-service:local .
```

Run the container:

```powershell
docker run --rm -p 8002:8002 execution-service:local
```

The container includes Python dependencies, `default-jdk-headless` for Java execution, and `nodejs` for JavaScript execution.

## Notes

This service executes user-provided code. Run it only in a controlled environment with suitable container, network, CPU, memory, and timeout controls.

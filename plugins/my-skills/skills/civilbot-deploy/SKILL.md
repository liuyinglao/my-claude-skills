---
name: civilbot-deploy
description: Deploy the CivilBot chatbot-api backend locally with Docker or to Google Cloud Run. Use when the user says "deploy", "deploy backend", "deploy to cloud run", "deploy for mr", "redeploy", or mentions deploying civilbot, chatbot-api, or the FastAPI backend.
version: 1.0.0
---

# CivilBot Backend Deployment

## What This Skill Does

Deploys the CivilBot FastAPI backend (`chatbot-api`) either locally via Docker Compose or to Google Cloud Run (project `civilbot-90b36`, region `us-west1`).

Two services are deployed together:
- **chatbot-api** — main FastAPI service (public, `--allow-unauthenticated`)
- **code-execution-worker** — sandboxed code runner (internal only, `--no-allow-unauthenticated`)

## Prerequisites

- Docker Desktop running
- `gcloud` CLI authenticated (`gcloud auth login`)
- GCP project set to `civilbot-90b36`
- Required secret `portkey-api-key` in Secret Manager

## Local Development

```bash
# From repo root
export PATH="/Applications/Docker.app/Contents/Resources/bin:$PATH"
export PORTKEY_API_KEY='your-portkey-api-key'
docker compose up --build
# API at http://localhost:8000
curl http://localhost:8000/health
```

## Cloud Run Deployment

```bash
cd /path/to/civilbot/chatbot-api
./deploy_cloudrun.sh
```

The script (at `chatbot-api/deploy_cloudrun.sh`) does everything automatically:
1. Authenticates Docker with Artifact Registry (`us-west1-docker.pkg.dev`)
2. Checks/creates required secrets in Secret Manager
3. Builds both images for `linux/amd64` (required for OpenSeesPy)
4. Pushes images tagged with the current git commit SHA
5. Deploys `code-execution-worker` (2Gi RAM, 1 CPU, 60s timeout, min 1)
6. Deploys `chatbot-api` (2Gi RAM, 2 CPU, 300s timeout, concurrency 4, min 1, max 20)
7. Wires service-to-service IAM so main API can invoke the worker

### Optional env vars for the deploy script

```bash
CLEANUP_AFTER=true KEEP_VERSIONS=5 ./deploy_cloudrun.sh  # prune old images after deploy
PORTKEY_API_KEY='...' ./deploy_cloudrun.sh                 # create secret on first run
```

## Post-Deploy Verification

```bash
# Replace URL with actual output from the script
curl https://chatbot-api-v5ox3dwtwq-uw.a.run.app/health

# Stream logs
gcloud run logs tail chatbot-api --region us-west1
gcloud run logs tail code-execution-worker --region us-west1
```

## Secrets Reference

| Secret name               | Required | Purpose                          |
|---------------------------|----------|----------------------------------|
| `portkey-api-key`         | Yes      | Portkey LLM routing              |
| `firebase-service-account`| Optional | Firebase auth                    |
| `wandb-api-key`           | Optional | W&B Weave LLM tracing            |
| `wandb-project`           | Optional | W&B project name                 |
| `resend-api-key`          | Optional | Magic link email delivery        |
| `magic-link-base-url`     | Optional | Frontend URL for magic links     |
| `magic-link-from-email`   | Optional | Sender address for magic links   |
| `session-secret-key`      | Optional | Magic link session signing       |

Create a missing secret:
```bash
echo -n 'VALUE' | gcloud secrets create SECRET_NAME --data-file=-

# Grant Cloud Run access if needed
PROJECT_NUMBER=$(gcloud projects describe civilbot-90b36 --format='value(projectNumber)')
gcloud secrets add-iam-policy-binding SECRET_NAME \
  --member="serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

## GCP Configuration

| Key             | Value                                          |
|-----------------|------------------------------------------------|
| Project ID      | `civilbot-90b36`                               |
| Region          | `us-west1`                                     |
| Artifact Registry | `us-west1-docker.pkg.dev/civilbot-90b36/civilbot-repo` |
| Main service    | `chatbot-api`                                  |
| Worker service  | `code-execution-worker`                        |

## Troubleshooting

| Error | Fix |
|-------|-----|
| Docker not running | `open -a Docker` and wait for whale icon |
| Docker not in PATH | `export PATH="/Applications/Docker.app/Contents/Resources/bin:$PATH"` |
| Secret not found | See "Create a missing secret" above |
| Push fails (auth) | `gcloud auth configure-docker us-west1-docker.pkg.dev` |
| 401/403 on worker | Check IAM binding: main API SA needs `roles/run.invoker` on worker |

## References

- QUICKSTART: `chatbot-api/QUICKSTART.md`
- Deploy script: `chatbot-api/deploy_cloudrun.sh`
- GCP project: https://console.cloud.google.com/run?project=civilbot-90b36

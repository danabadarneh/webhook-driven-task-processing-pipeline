# Webhook-Driven Task Processing Pipeline

TypeScript service that receives webhooks, queues background jobs, processes payloads, and delivers results to subscribers with retry logic.

## Architecture

- `API service` (Express):
  - CRUD for pipelines
  - webhook ingestion endpoint `/webhooks/:webhookKey`
- `PostgreSQL`:
  - stores pipelines, events, jobs, and per-subscriber delivery attempts
- `Worker service`:
  - pulls pending jobs
  - applies action types
  - delivers results to subscribers with exponential backoff retries

## Action Types

1. `uppercase`: uppercases all string values in payload recursively
2. `pick_fields`: keeps only selected fields (`config.fields`)
3. `add_metadata`: appends processing metadata (`eventId`, `processedAt`)

## Data Flow

1. Create pipeline via `POST /pipelines`
2. Send webhook to generated `sourceUrl`
3. API stores event and enqueues job (`202 Accepted`)
4. Worker processes action and creates delivery rows
5. Worker POSTs to subscribers with retry on failure
6. Job becomes `succeeded` or `failed`

## API

### Create pipeline

`POST /pipelines`

```json
{
  "name": "Order pipeline",
  "action": {
    "type": "pick_fields",
    "config": { "fields": ["orderId", "status"] }
  },
  "subscribers": [
    "https://example.com/hook-1",
    "https://example.com/hook-2"
  ]
}
```

### List pipelines

`GET /pipelines`

### Get pipeline

`GET /pipelines/:id`

### Update pipeline

`PUT /pipelines/:id`

### Delete pipeline

`DELETE /pipelines/:id`

### Ingest webhook

`POST /webhooks/:webhookKey`

Body: any JSON payload.

## Local Run (Docker)

```bash
cp .env.example .env
docker compose up --build
```

API base URL: `http://localhost:8080`

Health check:

```bash
curl http://localhost:8080/health
```

## Local Run (without Docker)

```bash
npm ci
cp .env.example .env
npm run migrate
npm run dev
```

In another terminal:

```bash
npm run dev:worker
```

## CI

GitHub Actions pipeline runs:

- `npm ci`
- `npm test`
- `npm run build`

#RUN 
-----------------------------------------------------------------------
- `cd /Users/danabadarneh/Desktop/webhook-driven-task-processing-pipeline`
- `cp .env.example .env`
-`npm install`
-`npm run build`
-`docker compose up --build -d`
-`curl http://localhost:8080/health`
-----------------------------------------------------------------------
Create pipeline:
-----------------------------------------------------------------------
- ` curl http://localhost:8080/health
curl -X POST http://localhost:8080/pipelines \
  -H "Content-Type: application/json" \
  -d '{"name":"demo-interview","action":{"type":"add_metadata"},"subscribers":["https://webhook.site/53170d31-bc50-4638-8667-29363384db93"]}'
`
-------------------------------------------------------------------------
Send webhook to sourceUrl :
-------------------------------------------------------------------------
- `curl -X POST http://localhost:8080/webhooks/7558e9c217c7803aa69f73fe \
  -H "Content-Type: application/json" \
  -d '{"orderId":"A777","status":"created","note":"final interview run"}'`


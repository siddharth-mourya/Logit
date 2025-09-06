# Backend architecture and folder layout
**Entire backend architecture and folder layout** to explicitly support your real-world IoT use cases:

* REST admin APIs (Org, Users, Devices, Ports, Alerts)
* **MQTT ingestion pipeline** (devices publish values)
* Real-time UI updates (Socket.IO / WebSocket)
* Time-series storage (MongoDB time-series collection)
* Calibration + alert evaluation at ingestion
* Simple notification hooks (email/webhook) and audit trail
* Modular TypeScript codebase so you can scale later

I’ll first show the **high-level architecture & dataflow**, then a **detailed TypeScript folder layout** with responsibilities for each part, and finally operational / scaling notes, topic naming, and processing sequences.

# Architecture — high level

Devices (MQTT) → MQTT Broker → Ingestion Worker → Processing Pipeline → DB (MongoDB) + Cache (Redis) → API Layer (REST) → Frontend (React)
Also: Ingestion Worker → Alert Evaluator → Notification Service → Alert Events stored; Ingestion Worker → real-time push via Socket.IO.

Possible Key components:

* **MQTT Broker** (Mosquitto / EMQX / Aedes) — accepts device telemetry.
* **Ingestion Worker** (Node TypeScript service) — subscribes to topics, validates, applies calibration, checkpoints & writes to DB, emits events.
* **REST API** (Express + TypeScript) — admin/ui endpoints: manage orgs, users, devices, ports, rules, historical queries, exports.
* **Realtime Layer** (Socket.IO) — push recent values/alerts to UI.
* **MongoDB** — metadata collections + time-series collection for values.
* **Redis (optional)** — cache for latest value per port and small pub/sub to scale real-time.
* **Notification Service** — send emails/SMS/webhooks when alerts trigger.

# Dataflow — sequence (for each device message)

1. Device publishes telemetry JSON to MQTT topic (e.g. `org/{orgId}/device/{deviceId}/telemetry`).
2. MQTT Broker receives message; Worker (subscriber) consumes it.
3. Worker validates message schema and checks device registration + auth.
4. Worker applies `calibration = value * scaling + offset` (from Port record).
5. Worker persists calibrated value to MongoDB time-series `values` collection (with orgId, deviceId, portNumber, timestamp, rawValue, calibratedValue, quality, ingestTs).
6. Worker writes latest value to Redis (optional) for O(1) UI reads.
7. Worker evaluates alert rules for that port (in-memory rules cache or DB lookup). If rule fires -> create `AlertEvent` in DB and enqueue notification.
8. Worker emits real-time event to connected clients via Socket.IO (or Redis pub/sub if scaled).
9. Batch job periodically archives raw records older than retention to S3 (Parquet) and deletes or compresses DB rows per retention policy.

## 🏗️ High-Level Data Flow

1. **Device → MQTT Broker**

   * Each device sends readings (values from ports) to an MQTT topic.
   * Example: `org/123/device/456/telemetry`.

2. **MQTT Worker → Backend**

   * Our Node.js service listens to the broker.
   * For every reading:

     * Validate the payload.
     * Apply calibration:

       ```
       calibrated = raw * scaling + offset
       ```
     * Save to MongoDB.
     * Check alerts.
     * Send updates to frontend (via WebSocket/Socket.IO).
---


# Topic naming (recommended)

* Telemetry (array of port readings): `org/{orgId}/device/{deviceId}/telemetry`
* Device lifecycle (online/offline): `org/{orgId}/device/{deviceId}/lifecycle`
* Commands (server -> device): `org/{orgId}/device/{deviceId}/command`
* Command responses: `org/{orgId}/device/{deviceId}/cmdres`

Payload pattern (telemetry example):

```json
{
  "ts": "2025-09-06T12:00:00Z",
  "readings": [
    { "port": 1, "value": 230.5 },
    { "port": 2, "value": 0 }
  ]
}
```



# TypeScript Project — folder structure (complete, ready for coding)

```
iot-monitor-backend/
├─ src/
│  ├─ config/
│  │  ├─ env.ts                 # env loader + typed config
│  │  ├─ db.ts                  # mongodb connection
│  │  ├─ mqtt.ts                # mqtt client / connection helper
│  │  └─ redis.ts               # optional redis client
│  │
│  ├─ infra/                    # infra integrations & adapters
│  │  ├─ broker/                # MQTT broker adapters
│  │  │  ├─ mqttBroker.ts       # wrapper around mqtt client
│  │  ├─ notifier/              # notifications adapter (email, webhook)
│  │  │  ├─ emailClient.ts
│  │  │  └─ webhookClient.ts
│  │  └─ storage/               # S3/MinIO archiver
│  │     └─ archiver.ts
│  │
│  ├─ modules/                  # feature modules (domain-driven)
│  │  ├─ auth/
│  │  │  ├─ auth.controller.ts
│  │  │  ├─ auth.service.ts
│  │  │  ├─ auth.routes.ts
│  │  │  └─ auth.types.ts
│  │  │
│  │  ├─ organization/
│  │  │  ├─ organization.model.ts
│  │  │  ├─ organization.service.ts
│  │  │  └─ organization.routes.ts
│  │  │
│  │  ├─ user/
│  │  │  ├─ user.model.ts
│  │  │  ├─ user.service.ts
│  │  │  └─ user.routes.ts
│  │  │
│  │  ├─ device/
│  │  │  ├─ device.model.ts
│  │  │  ├─ device.service.ts
│  │  │  └─ device.routes.ts
│  │  │
│  │  ├─ port/
│  │  │  ├─ port.model.ts
│  │  │  ├─ port.service.ts
│  │  │  └─ port.routes.ts
│  │  │
│  │  ├─ value/
│  │  │  ├─ value.model.ts      # time-series collection model
│  │  │  ├─ value.service.ts    # ingestion helpers (used by worker + API)
│  │  │  ├─ value.controller.ts # historical queries via REST
│  │  │  └─ value.routes.ts
│  │  │
│  │  └─ alert/
│  │     ├─ alertRule.model.ts
│  │     ├─ alert.event.model.ts
│  │     ├─ alert.service.ts
│  │     └─ alert.routes.ts
│  │
│  ├─ workers/
│  │  ├─ mqttWorker.ts          # subscribes to topics and processes telemetry
│  │  └─ archiveWorker.ts       # scheduled archival/rollup jobs
│  │
│  ├─ shared/                   # common types, DTOs
│  │  ├─ types.ts
│  │  └─ dto/
│  │     ├─ telemetry.dto.ts
│  │     └─ apiResponses.ts
│  │
│  ├─ middlewares/
│  │  ├─ auth.middleware.ts
│  │  ├─ error.middleware.ts
│  │  └─ validate.middleware.ts
│  │
│  ├─ utils/
│  │  ├─ calibration.ts         # apply scaling & offset
│  │  ├─ pagination.ts
│  │  └─ logger.ts
│  │
│  ├─ app.ts                    # express app setup (routes + socket)
│  └─ server.ts                 # starts API + workers + mqtt client
│
├─ scripts/
│  └─ db-init.ts                # seed initial data (company admin, sample org)
├─ docs/
│  └─ architecture.md
├─ tsconfig.json
├─ package.json
└─ .env
```

# Responsibilities — per component (short)

* `mqttWorker.ts`

  * Maintains MQTT connection, subscribes to `org/+/device/+/telemetry`.
  * For each message → parse → validate → call `value.service.ingestTelemetry()`.

* `value.service.ts`

  * Applies per-port calibration (scaling, offset).
  * Stores raw & calibrated value in MongoDB time-series `values` (with tags: orgId, deviceId, port).
  * Publishes latest to Redis and emits to Socket.IO.
  * Calls `alert.service.evaluate(portId, calibratedValue)`.

* `alert.service.ts`

  * Maintains cached alert rules (in-memory or Redis).
  * Evaluates rules; on trigger → create `AlertEvent` record and call `notifier` adapter.

* `notifier` (infra)

  * Simple pluggable interface: `sendEmail`, `sendSMS`, `postWebhook`.
  * Initially implement `webhookClient` + console email via Nodemailer later.

* `device.service.ts` / `port.service.ts`

  * CRUD for devices/ports (CompanyAdmin usage).
  * Return port calibration & threshold config.

* `auth.service.ts`

  * JWT issuance, company-admin vs org-admin roles, token validation.

* `app.ts`

  * Mount routes, initialize Socket.IO, attach middleware.

* `server.ts`

  * Entrypoint: connect DB, start app, start MQTT worker and background workers.


# ⚙️ **Architecture Style**

We’ll use a **Modular Monolith** pattern (feature-based modules).
Each **module** (User, Device, Port, etc.) contains:

* `model.ts` → Mongoose schema & TypeScript interface
* `service.ts` → Business logic
* `controller.ts` → Request handlers (calls services)
* `routes.ts` → Express routes
* `types.ts` → Shared TS types for this module

This makes it **easy to scale** later:

* If system grows → split modules into microservices.
* If not → keep monolith, still clean.

---

# 🔑 **Tech Setup**

* **Runtime**: Node.js
* **Language**: TypeScript
* **Framework**: Express.js
* **Database**: MongoDB with Mongoose
* **Auth**: JWT
* **Validation**: Zod or Joi (for request body validation)
* **Logging**: Winston or Pino (starter can just use `console.log`)



# MongoDB modeling & indexes — important decisions

**Collections**:

* `organizations`
* `users`
* `devices`
* `ports`
* `values` → **time-series collection** with `timeField: ts`, `metaField: { orgId, deviceId, port }`
* `alertrules`
* `alertevents`

**Indexes (critical)**:

* `values` meta tags: ensure `metadata.orgId`, `metadata.deviceId`, `metadata.port` are frequently used by time-series queries.
* For historical queries: compound index on `(metadata.deviceId, metadata.port, ts)` — Mongo handles time-series indexing, ensure queries use metadata filters.
* `devices.serialNumber` unique index.
* `ports.deviceId + portNumber` unique compound index.
* `alertrules.organizatonId` and `alertevents.status + triggeredAt`.

# Calibration & storage fields (per value)

A single persisted value document should contain:

* `ts` (timestamp)
* `metadata`: `{ orgId, deviceId, portNumber }`
* `rawValue`
* `calibratedValue`
* `quality` (optional)
* `ingestTs`
* `rawPayload` (optional small json for debugging)

# Reliability & fault handling

* **MQTT QoS**: Devices use QoS 1 (at least once). Ensure ingestion is idempotent (dedupe by device+port+ts if needed).
* **Validation**: Worker validates schema & timestamp ranges. Bad messages routed to DLQ (error collection) for inspection.
* **Backpressure**: If DB slow, buffer messages into local RocksDB/LevelDB or use Redis list. For starter, rely on Node worker + small in-memory queue and set alarms.
* **Monitoring**: Export simple Prometheus metrics (ingest rate, errors) — add later.

# Scaling path (from starter → production)

Starter:

* Single Node app runs Express + MQTT worker + Socket.IO.
* MongoDB Atlas single cluster.
* Redis optional single instance.

Scale:

1. Split ingestion worker into separate process (dedicated container) and run multiple consumers.
2. Introduce Kafka between MQTT Broker and processing workers for buffering & replay.
3. Scale Socket.IO using Redis adapter for multi-instance pub/sub.
4. MongoDB sharding or move time-series to specialized TSDB if needed (ClickHouse/Timescale) for analytics.
5. Use Kubernetes for orchestration.

# Operational notes / env variables (minimum)

* `MONGO_URI`
* `PORT`
* `JWT_SECRET`
* `MQTT_URL`
* `MQTT_USERNAME` / `MQTT_PASSWORD`
* `REDIS_URL` (optional)
* `ARCHIVE_S3_BUCKET` / AWS creds (optional)

# Documentation & onboard checklist (for repo)

* `docs/architecture.md` — include diagrams (textual + arrow flows), ingestion sequence, and failure modes.
* `README.md` — setup quickstart (env, local run using Docker Compose for Mongo + optional Mosquitto + Redis).
* `scripts/db-init.ts` — seed company admin + sample org + sample device + sample ports.
* API docs: Add Swagger later; for now, document endpoints in `docs/api.md`.

---

If this looks good I’ll start the **next message** by generating the **TypeScript project bootstrap** (package.json deps, tsconfig, ESLint basic rules) and then create the **core files** in order:

1. `tsconfig.json` + install packages
2. `src/config/env.ts` and `src/config/db.ts`
3. `src/app.ts` + `src/server.ts`
4. `src/infra/mqttBroker.ts` (MQTT client wrapper)
5. Basic `modules/organization` and `modules/user` models and routes so admin APIs exist
6. `workers/mqttWorker.ts` wiring to the broker (no heavy business logic yet)

Tell me to proceed and I’ll generate the exact files and code in the next response.


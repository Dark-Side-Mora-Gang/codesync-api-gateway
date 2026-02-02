# codesync-api-gateway

## 📌 Purpose of This Service

This repository acts as the **single entry point** for all client requests.
It does **NOT** contain business logic. Instead, it:

* Routes requests to backend microservices
* Terminates TLS (HTTPS)
* Handles authentication validation
* Applies rate limiting & security headers
* Proxies WebSocket connections

This is a **true production-grade API Gateway** design.

---

## 📁 File & Folder Hierarchy

```
codesync-api-gateway/
├── nginx/
│   ├── nginx.conf              # Main NGINX config (entry point)
│   ├── upstreams.conf          # Backend service definitions
│   ├── routes/
│   │   ├── auth.conf           # Routes → auth-service
│   │   ├── collab.conf         # Routes → collaboration-service (WS)
│   │   ├── compiler.conf       # Routes → compiler-service
│   │   └── version.conf        # Routes → version-service
│   ├── security/
│   │   ├── headers.conf        # Security headers (CORS, CSP, etc.)
│   │   └── rate-limit.conf     # Rate limiting rules
│   └── websocket.conf          # WebSocket proxy settings
│
├── docker/
│   ├── Dockerfile              # API Gateway container
│   └── nginx-entrypoint.sh     # Startup script (env substitution)
│
├── env/
│   ├── .env.example            # Example environment variables
│   └── .env                    # Actual env (gitignored)
│
├── docker-compose.yml          # Local orchestration of services
├── .gitignore
├── README.md
└── architecture.md             # How services communicate (docs)
```

---

## 🔌 How This Gateway Connects to Other Backends

### 🔁 Backend Services (Internal Network)

All services run on the **same Docker network**.
NGINX talks to them using **service names** (not localhost).

```
Frontend
   ↓
API Gateway (NGINX)
   ↓
-------------------------------------------------
| auth-service        :3001                     |
| collaboration-svc   :3002 (WebSocket)         |
| compiler-service    :3003                     |
| version-service     :3004                     |
-------------------------------------------------
```

No backend is exposed publicly except the **gateway**.

---

## 🧭 Routing Strategy (Professional)

| Client Request  | Routed To                  |
| --------------- | -------------------------- |
| /api/auth/*     | auth-service               |
| /api/collab/*   | collaboration-service (WS) |
| /api/compiler/* | compiler-service           |
| /api/version/*  | version-service            |

---

## 📦 What You Need to Install (Gateway Machine)

### Mandatory

* Docker
* Docker Compose

### Optional (for debugging)

* curl
* httpie
* Postman

⚠️ You **do NOT** install Node.js here.
NGINX only proxies traffic.

---

## 🐳 docker-compose.yml (Conceptual View)

This gateway does **not** own services, but knows where they live.

* All services share one Docker network
* Service names act as DNS

Example:

```
services:
  api-gateway:
    ports:
      - "80:80"
    depends_on:
      - auth-service
      - collab-service
```

---

## 🔐 Security Responsibilities (Gateway Level)

Handled **ONLY here**:

* CORS
* Rate limiting
* JWT verification (basic)
* IP filtering
* HTTPS termination

Handled **NOT here**:

* Business authorization
* Database access
* Core logic

---

## 🧠 Why This Is Professional-Level

* Clear separation of concerns
* Zero business logic in gateway
* WebSocket-safe routing
* Production-ready structure
* Mirrors real systems (Netflix, Uber, Stripe)

---

> "This API Gateway is designed as a lightweight reverse proxy using NGINX, enforcing security and routing policies while delegating business logic to independently deployed microservices."

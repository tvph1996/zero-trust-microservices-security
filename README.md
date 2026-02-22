# Zero-Trust Microservices Security & Observability

This project implements a secure, observable microservices architecture using **Zero-Trust** principles. It demonstrates a complete pipeline for securing service-to-service communication, identity management, and distributed monitoring.


<img width="788" height="570" alt="hung" src="https://github.com/user-attachments/assets/fb02e41c-6292-4a04-b509-eadf432c3e11" />

<img width="1280" height="640" alt="image" src="https://github.com/user-attachments/assets/3a8b8e7a-6884-4704-b1fa-52ce5f157938" />

<img width="1648" height="578" alt="image" src="https://github.com/user-attachments/assets/8cfecab7-7e9a-4ac5-9cb1-8cff50e10172" />




## Technical Overview
The system consists of a FastAPI REST gateway, a gRPC backend service, and a MongoDB database. Every connection is secured, and every request is traced across the stack.

### Security Implementation
- **Ingress Protection**: Traefik acts as the entry point, handling HTTPS termination and masking internal service topology. Only the REST API is exposed to the outside world.
- **Identity & Access Management**: Keycloak serves as the OIDC provider. The REST service validates JWT bearer tokens for all incoming requests, ensuring only authenticated users can interact with the system.
- **Mutual TLS (mTLS)**: Zero-trust is enforced at the network level. All internal communication between the REST and gRPC services is encrypted and verified using a custom Certificate Authority (CA), requiring services to prove their identity to each other.
- **Service Isolation**: Services are non-exposed by default. Traffic is only routed via Traefik through explicit Docker labels.

### Observability & Resilience
- **Distributed Tracing**: Full-stack instrumentation with OpenTelemetry (OTel), allowing request flows to be tracked through the REST and gRPC layers into Jaeger.
- **Performance Metrics**: Real-time metrics collection via Prometheus, with visualization through Grafana dashboards.
- **Fault Tolerance**: A Circuit Breaker pattern is implemented in the REST service to prevent cascading failures if the backend gRPC service or MongoDB becomes unavailable.

## Architecture Stack
- **Gateway**: Traefik v2.11
- **Auth**: Keycloak 25.0
- **Services**: FastAPI (REST), Python (gRPC)
- **Monitoring**: Prometheus, Grafana, Jaeger, OpenTelemetry Collector, Portainer
- **Database**: MongoDB

## Getting Started
### Deployment
Run the entire stack using Docker Compose:
```bash
docker compose up -d
```

### Key Endpoints
- **REST API**: `https://localhost/api` (Root endpoint requires local CA certificate)
- **Auth Service**: `http://localhost:8081` (Keycloak)
- **Monitoring**: `https://localhost:9443` (Portainer)
- **Metrics**: `http://localhost:3000` (Grafana)
- **Tracing**: `http://localhost:16686` (Jaeger)

## Security Verification
The following highlights the results of the zero-trust implementation:
- **Encrypted Communication**: Traffic captured between services is unreadable due to the mTLS encryption.
- **Fail-Safe Authentication**: Requests without a valid JWT or with an expired token are rejected with a `401 Unauthorized` status.
- **Lateral Movement Prevention**: Services only accept requests from verified identities within the mTLS network; external attempts to bypass the gateway are blocked.
- **Circuit Breaker**: Requests to the API gracefully fail when backend services are down, preventing a system-wide freeze.

## Usage Example
### 1. Authenticate with Keycloak
Fetch a JWT access token:
```
curl -s -X POST 'http://localhost:8081/realms/dsa-lab/protocol/openid-connect/token' \
-d 'client_id=rest-client' \
-d 'grant_type=password' \
-d 'username=tvph1996' \
-d 'password=admin' \
| jq -r .access_token > token.jwt
```

### 2. Access the Secured API
Use the token and the CA certificate to interact with the protected `/items` endpoint:
```
curl -X POST https://localhost/api/items \
--cacert security/certs/ca.crt \
-H "Authorization: Bearer $(cat token.jwt)" \
-H "Content-Type: application/json" \
-d '{"id": 1, "name": "Secure Item"}'
```

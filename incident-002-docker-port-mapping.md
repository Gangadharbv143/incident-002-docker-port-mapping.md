# Incident 002: Docker Port Mapping Misconfiguration

**Category:** Docker Networking  
**Environment:** Ubuntu 22.04 / WSL2  
**Image:** `devops-cicd-fixed:v1`  
**Status:** Resolved

## 1. Incident Description

A Flask application container was running, but users could not access the application through the published host port.

This incident was reproduced intentionally in a separate test container.

## 2. Investigation

Commands used:

```bash
docker ps --filter name=incident002-bad-port
docker port incident002-bad-port
curl -i --max-time 5 http://localhost:8083
docker logs incident002-bad-port
```

## 3. Findings

Container status:

```text
Up 2 minutes
```

Incorrect port mapping:

```text
8083 -> 5001
```

Flask application listening port:

```text
0.0.0.0:5000
```

HTTP test:

```text
curl: (56) Recv failure: Connection reset by peer
```

## 4. Root Cause

Docker published host port 8083 to container port 5001, but the Flask application was listening on container port 5000.

This port mismatch prevented successful access to the application.

## 5. Recovery

Stopped and removed the faulty test container:

```bash
docker stop incident002-bad-port
docker rm incident002-bad-port
```

Created a corrected container:

```bash
docker run -d \
  --name incident002-fixed \
  -p 8083:5000 \
  devops-cicd-fixed:v1
```

## 6. Verification

```bash
docker ps --filter name=incident002-fixed
curl -i http://localhost:8083
```

Results:

- Container status: Up
- Correct port mapping: 8083 -> 5000
- HTTP response: 200 OK
- Application response: `CI/CD Pipeline Working!`

## 7. Preventive Measures

- Verify application listening ports before publishing Docker ports.
- Check Docker port mappings.
- Test application endpoints using curl.
- Review container logs during troubleshooting.
- Configure application-level health checks.

## 8. Key Learning

A running Docker container does not guarantee application availability.

Always verify the application process, listening port, port mapping and HTTP response.

**Outcome: Incident successfully reproduced, diagnosed and resolved in a local lab simulation.**

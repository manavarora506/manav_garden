-Thin layer in front of client. Client only needs to know one endpoint, where API gateway will route client accordingly
-Can also have middleware like auth, logging, caching 
1) Validate request
2) Run middleware (auth, rate limiting, etc)
3) Route to correct service
4) Transform the response

\
Perfect — let’s reframe this as a **Low-Level Design (LLD)** problem for an **Amazon SDE 1 interview**, where you're asked:

> **"Design an API Gateway."**

They’re not expecting production-ready infra, but they want to see if you:

- Understand how requests are routed
    
- Think about extensibility (e.g., rate-limiting, auth, service discovery)
    
- Use appropriate abstractions (classes, interfaces)
    
- Handle edge cases (timeouts, retries, unregistered services)
    

---

## 🧱 1. Start with High-Level Architecture

**Clarify first:**

- What kind of API Gateway? L7 HTTP proxy?
    
- Do we need:
    
    - Path-based routing? (`/auth` → auth service)
        
    - Method-specific rules?
        
    - Rate limiting?
        
    - JWT auth?
        
    - Circuit breakers?
        

Assume the minimum: path-based routing with optional logging + error handling.

---

## 🧠 2. Define Core Components

|Component|Responsibility|
|---|---|
|`APIGateway`|Receives requests, routes to correct backend|
|`Router`|Determines which service to forward to|
|`ServiceRegistry`|Stores service → URL mappings|
|`Forwarder`|Sends the request to the backend service|
|`Strategy` (optional)|Middleware: logging, retry, auth, etc|

---

## 🐍 3. Python LLD Implementation

### `service_registry.py`

```python
class ServiceRegistry:
    def __init__(self):
        self.services = {}

    def register(self, name: str, url: str):
        self.services[name] = url

    def get(self, name: str) -> str:
        return self.services.get(name)
```

---

### `router.py`

```python
class Router:
    def __init__(self, registry):
        self.registry = registry

    def route(self, path: str) -> str:
        parts = path.strip("/").split("/")
        service_name = parts[0] if parts else None
        return self.registry.get(service_name)
```

---

### `forwarder.py`

```python
import httpx

class Forwarder:
    async def forward(self, base_url: str, path: str, request):
        url = f"{base_url}/{ '/'.join(path.strip('/').split('/')[1:]) }"
        async with httpx.AsyncClient() as client:
            response = await client.request(
                request.method,
                url,
                headers=request.headers.raw,
                params=dict(request.query_params),
                content=await request.body()
            )
            return response
```

---

### `gateway.py`

```python
from fastapi import FastAPI, Request
from service_registry import ServiceRegistry
from router import Router
from forwarder import Forwarder
from fastapi.responses import JSONResponse, Response

app = FastAPI()

registry = ServiceRegistry()
registry.register("auth", "http://localhost:8001")
registry.register("user", "http://localhost:8002")

router = Router(registry)
forwarder = Forwarder()

@app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
async def gateway(path: str, request: Request):
    base_url = router.route(path)
    if not base_url:
        return JSONResponse({"error": "Service not found"}, status_code=404)
    try:
        resp = await forwarder.forward(base_url, path, request)
        return Response(
            content=resp.content,
            status_code=resp.status_code,
            headers=resp.headers
        )
    except Exception:
        return JSONResponse({"error": "Downstream error"}, status_code=502)
```

---

## 🔁 4. How to Explain in Interview

> "I designed the system using clear modular components.
> 
> - `ServiceRegistry` handles dynamic mappings between service names and base URLs.
>     
> - `Router` reads the incoming request path to identify the service.
>     
> - `Forwarder` handles the actual HTTP request forwarding with headers and query params.
>     
> - The gateway exposes a single endpoint and uses these to forward requests.
>     

> This structure supports adding middleware like auth, retries, or logging easily by inserting decorators or hooks into the `gateway` handler."

---

## ✅ 5. Follow-Up Extensions

|Feature|Approach|
|---|---|
|Rate Limiting|Add Redis-backed count in middleware|
|JWT Auth|Inspect request headers before forwarding|
|Retry Logic|Retry on timeout using `httpx`|
|Service Discovery|Replace registry with Consul, ECS, etc.|
|Metrics|Use Prometheus to track service calls|

---

Let me know if you want to do the **same LLD but with gRPC**, or if you want to prep for **follow-up questions** you might be asked on this design!

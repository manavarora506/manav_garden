Design Load Balancer code

from abc import ABC, abstractmethod
from typing import List
import threading

class Server:
    def __init__(self, server_id):
        self.server_id = server_id
        self._is_healthy = True
        self._connections = 0
        self._lock = threading.Lock()

    def is_healthy(self):
        return self._is_healthy

    def set_healthy(self, healthy):
        self._is_healthy = healthy

    def get_connections(self):
        with self._lock:
            return self._connections

    def increment_connections(self):
        with self._lock:
            self._connections += 1

    def decrement_connections(self):
        with self._lock:
            self._connections = max(0, self._connections - 1)

    def __repr__(self):
        return f"Server({self.server_id})"

# Placeholder Request class
class Request:
    pass

# Strategy interface
class LoadBalancingStrategy(ABC):
    @abstractmethod
    def get_server(self, servers: List[Server], request: Request) -> Server:
        pass

# Round Robin Strategy
class RoundRobinStrategy(LoadBalancingStrategy):
    def __init__(self):
        self._index = 0
        self._lock = threading.Lock()

    def get_server(self, servers: List[Server], request: Request) -> Server:
        healthy_servers = [s for s in servers if s.is_healthy()]
        if not healthy_servers:
            raise Exception("No healthy servers available")

        with self._lock:
            server = healthy_servers[self._index % len(healthy_servers)]
            self._index += 1
            return server

# Least Connections Strategy
class LeastConnectionsStrategy(LoadBalancingStrategy):
    def get_server(self, servers: List[Server], request: Request) -> Server:
        healthy_servers = [s for s in servers if s.is_healthy()]
        if not healthy_servers:
            raise Exception("No healthy servers available")

        return min(healthy_servers, key=lambda s: s.get_connections())

# Singleton LoadBalancer
class LoadBalancer:
    _instance = None

    def __new__(cls):
        if not cls._instance:
            cls._instance = super(LoadBalancer, cls).__new__(cls)
            cls._instance._servers = []
            cls._instance._strategy = None
        return cls._instance

    def add_server(self, server: Server):
        self._servers.append(server)

    def remove_server(self, server: Server):
        self._servers = [s for s in self._servers if s != server]

    def set_strategy(self, strategy: LoadBalancingStrategy):
        self._strategy = strategy

    def get_server(self, request: Request) -> Server:
        if not self._strategy:
            raise Exception("No strategy set")
        return self._strategy.get_server(self._servers, request)

# Example Usage
if __name__ == "__main__":
    # Create servers
    server1 = Server("server1")
    server2 = Server("server2")

    # Set up Load Balancer
    lb = LoadBalancer()
    lb.add_server(server1)
    lb.add_server(server2)

    # Strategy
    lb.set_strategy(RoundRobinStrategy())
    # lb.set_strategy(LeastConnectionsStrategy())  # Try this too

    # Simulate Requests
    for i in range(5):
        req = Request()
        selected = lb.get_server(req)
        print(f"Request {i+1} routed to {selected.server_id}")


Key Notes:

Great question — you're looking at the `__new__` method in Python, which is being used here to implement the **Singleton pattern** for the `LoadBalancer` class.

---

## 🧠 What is `__new__`?

- `__new__` is a **special method** in Python that **creates a new instance** of a class.
    
- It’s called **before** `__init__`, and it’s where you can **control instance creation**.
    

---

## 🧱 Why Use `__new__` Here?

The code is trying to **make sure only one instance of `LoadBalancer` is ever created** — a **singleton**.

```python
class LoadBalancer:
    _instance = None  # Class-level variable to hold the singleton instance

    def __new__(cls):
        if not cls._instance:
            # If no instance exists yet, create one and store it in cls._instance
            cls._instance = super(LoadBalancer, cls).__new__(cls)
            cls._instance._servers = []      # initialize state
            cls._instance._strategy = None
        return cls._instance
```

So even if you write:

```python
lb1 = LoadBalancer()
lb2 = LoadBalancer()
```

Both `lb1` and `lb2` will point to the **same object**.

---

## 🔁 How It Works with `__init__`

Even though `__init__` is called **every time** you create an object, `__new__` makes sure that only **one object is ever actually created**.

```python
lb1 = LoadBalancer()
lb2 = LoadBalancer()

print(lb1 is lb2)  # True
```

---

## ✅ Why Singleton for LoadBalancer?

- You want a **single point of control** over all servers.
    
- Ensures that server list and strategy are consistent across all parts of your app.
    

---

## TL;DR

- `__new__` is a low-level method used to control how/if a new object is created.
    
- In this case, it's used to **enforce the singleton pattern** for the LoadBalancer.
    


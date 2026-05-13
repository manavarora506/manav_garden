---
title: "Backend & Systems Learning Questions"
tags:
  - learning
  - backend
  - devops
  - reference
---

# Backend & Systems Learning Questions

## Docker

- Demonstrate basic images
- Demonstrate hot reload of a webserver (bind mount)
- Demonstrate pulling in redis and communicating to it over a network through docker compose
  - Ask how docker interacts with the OS and provisions these resources (ex: linux networking namespace)
- What is the docker daemon (and alternatives ex: podman)

## Docker + Webservers

- Demonstrate pulling in nginx as a simple load balancer with docker scale
  - Understand L4 and L7 load balancing, geo distributed load balancing, WSGI, reverse proxy
- Demonstrate Django, Gunicorn
- Demonstrate Celery workers + some message broker
- Demonstrate traefik
- Async Webserver (fastapi)
- Demonstrate websocket streaming, HTTP streaming

## Distributed Webservers

- Demonstrate Kubernetes
  - Understand containerd, kubernetes networking model

## Fullstack Modern Frontend

- Demonstrate React SPA, Vercel + Next.JS

---

# Concepts Worth Learning at a Super High Level

## OS & Shell Fundamentals

- History of OS + What is a kernel exactly
- What is a shell exactly (an interpreter, ex: bash, zsh, fish)
- What is a terminal emulator exactly (I/O GUI for the shell)
- What are the GNU core utilities (C programs)
- What is the pipe (just a buffer and two file descriptors)

## Networking & Internet

- Internet Stack / OSI Model
- DNS Resolution, Recursive Resolvers, Providers, Registry, etc.
- IPv4, IPv6
- TCP, HTTP Streaming, Websockets, UDP
- SSL, TLS

## Browser & Web Platform

- History of HTML / HTTP / Browser / DOM / XML / HTML5
- What is a browser? (C++ program with networking stack, renderer, JS interpreter)
- Browser Architecture (Web API, task queue micro/macro, history of JS API, etc.)
- PHP, V8 Engine, AJAX, DOM Parser, DOM API, React, VDOM
- Node, ES module vs Node package, bundlers (Webpack, Vite)
- React + NextJS, React SPA, Page Routers, Vercel
- Serverless, Edge Computing
- Server Components, `use client`, hydration
- Wasm

## Language Implementation

- Interpreted vs Compiled, JIT compilation
- PyPy, calling into C from Python (ex: jax)
- Scripting languages that work closely with compiled/embedded languages (lua in engines, redis lua, etc.)

## Concurrency Models

- How do various languages support concurrency?
  - Coroutine / green threads
  - Explicit thread control / POSIX thread
  - Default thread pool
  - Event loop

## Authentication

- Basic Auth / Bearer / JWT
- OAuth

## Startups & Business

- Pre-seed / accelerators
- Funding rounds
- Typical equity model / structure

---

# Python Deep Dive

## Language Implementation

- How is the language implemented - objects, scope

## Useful Tricks for Interviewing

- `dataclass` + frozen, order
- `Counter`, `defaultdict`, `sortedlist`

## Modules

- `asyncio`, `multiprocessing`
- `celery`

### Client-side HTTP

- `requests`, `aiohttp`, `tenacity`

## Package Management

- venv, poetry, uv

# API Gateway

## 1. API Gateway

A specialized reverse proxy that sits in front of a set of microservices to aggregate, transform, and secure API calls

## 2. Other Routing Components

> [!NOTE]
> Overlap exists, a reverse proxy may also act as a load balancer or API gateway.

### 2.1. Forward Proxy

An intermediary server between a client and the internet. The client explicitly configures this proxy.

### 2.2. Reverse Proxy

An intermediary server in front of backend servers; the client (backend) is unaware of it.

### 2.3. Web Server

A software component that serves HTTP content (static or dynamic) to clients (usually browsers).

### 2.4. Load Balancer

A network component that distributes incoming traffic across multiple backend instances.

### 2.5. Backend for Frontend 

A custom API layer tailored for a specific frontend application (e.g., mobile vs web). BFF is not a product, it is typically a custom-developed backend, often co-evolving with frontend development.

## 3. Proxy Config

### 3.1. Permanent Redirect

A 308 Permanent Redirect (HTTP 308) status code indicates that the requested resource has been permanently moved to a new URL, and the client should update its records to reflect the new location. It's similar to a 301 Moved Permanently status, but 308 specifically preserves the request method (like POST or PUT) and body. 
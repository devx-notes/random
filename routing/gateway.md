# API Gateway

## 1. API Gateway

A specialized reverse proxy that sits in front of a set of microservices to aggregate, transform, and secure API calls

## 2. Other Routing Components

> [!NOTE
> Overlap exists, a reverse proxy may also act as a load balancer or API gateway.

### Forward Proxy

An intermediary server between a client and the internet. The client explicitly configures this proxy.

### Reverse Proxy

An intermediary server in front of backend servers; the client (backend) is unaware of it.

### Web Server

A software component that serves HTTP content (static or dynamic) to clients (usually browsers).

### Load Balancer

A network component that distributes incoming traffic across multiple backend instances.

### Backend for Frontend 

A custom API layer tailored for a specific frontend application (e.g., mobile vs web). BFF is not a product, it is typically a custom-developed backend, often co-evolving with frontend development.
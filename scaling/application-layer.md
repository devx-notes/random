# Scaling Application Layer

In the application layer, while Partition Tolerance (P) is generally assumed (due to the possibility of network failures across microservices or data centers), and Consistency (C) is usually offloaded to the data layer, the Availability (A) dimension becomes the primary architectural concern

## 1. Availability

This is the most explicit factor that is addressed in application design

- **Throughput**: a system with high throughput can handle more traffic and transactions, making it better equipped to maintain availability during peak loads or failures

## 2. Consistency

Although data consistency is primarily enforced at the storage layer, applications still make decisions based on assumptions about data state
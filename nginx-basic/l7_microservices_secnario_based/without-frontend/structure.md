# Architecture
- 1 API Gateway

- 1 Eureka (Service Registry)

- 2 Microservices (app-1, app-2)

- app-1 has 2 instances

```bash
                      Client
                        │
                        ▼
                 ┌────────────────┐
                 │     NGINX      │  ← Edge Load Balancer
                 └────────────────┘
                        │
                        ▼
                ┌──────────────────┐
                │  API Gateway     │ (Spring Cloud Gateway)
                │  (Multiple Pods) │
                └──────────────────┘
                        │
                        ▼
                 ┌───────────────┐
                 │   EUREKA      │
                 │ Service Reg.  │
                 └───────────────┘
                        │
         ┌──────────────┴──────────────┐
         ▼                             ▼
   ┌──────────────┐              ┌──────────────┐
   │   APP-1      │              │   APP-2      │
   │ Instance A   │              │ Instance A   │
   ├──────────────┤              └──────────────┘
   │ Instance B   │
   └──────────────┘

```
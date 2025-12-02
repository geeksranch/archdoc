# Overall System Architecture

```mermaid
graph TD
    Client[Client Application]
    API[API Gateway]
    Auth[Authentication Service]
    S1[Service 1]
    S2[Service 2]
    DB[(Database)]
    
    Client --> API
    API --> Auth
    API --> S1
    API --> S2
    S1 --> DB
    S2 --> DB
    Auth --> DB
```

# Data Flow Diagram

```mermaid
flowchart TD
    Input[User Input]
    UI[UI Layer]
    API[API Layer]
    BL[Business Logic]
    DB[(Database)]
    Output[User Output]

    Input --> UI
    UI --> API
    API --> BL
    BL --> DB
    DB --> Output
    BL --> Output
```
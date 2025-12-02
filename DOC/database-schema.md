# Database Schema

```mermaid
erDiagram
    USERS ||--o{ ORDERS : places
    USERS {
        string id PK
        string name
        string email
    }
    ORDERS {
        string id PK
        string user_id FK
        date order_date
        float amount
    }
    PRODUCTS ||--o{ ORDERS : included_in
    PRODUCTS {
        string id PK
        string name
        float price
    }
    ORDERS ||--o{ ORDER_ITEMS : contains
    ORDER_ITEMS {
        string id PK
        string order_id FK
        string product_id FK
        int quantity
    }
}```
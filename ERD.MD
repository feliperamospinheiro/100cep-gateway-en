```mermaid
erDiagram
    orders ||--o{ payments : has
    orders ||--o{ items : contains
    customers ||--o{ orders : places
    sellers ||--o{ items : sells

    orders {
        string order_id
        string customer_id
        string status
        datetime purchase_timestamp
    }

    payments {
        string order_id
        string payment_type
        int installments
        double payment_value
    }

    items {
        string order_id
        string seller_id
        double price
        double freight_value
    }

    customers {
        string customer_id
        string city
        string state
    }

    sellers {
        string seller_id
        string city
        string state
    }

---

# 🏗️ **7. Lakehouse Architecture (Mermaid)**

```markdown
```mermaid
flowchart TD
    A[Kaggle CSVs] --> B[Bronze Layer - Raw Delta Tables]
    B --> C[Silver Layer - Cleaned Tables]
    C --> D[Gold Layer - Analytics Models]
    D --> E[Business KPIs / SQL Queries / Dashboards]

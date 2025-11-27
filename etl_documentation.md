# Documentação do ETL — 100cep Gateway

Este documento descreve as transformações aplicadas em cada camada.

---

# 🟫 Bronze → Silver (Limpeza)

### Orders
- Conversão de timestamps
- Normalização de status
- Remoção de linhas duplicadas
- Ajuste de colunas nulas relacionadas a entregas

### Payments
- Inferência de tipo numérico
- Correção de categorias
- Agrupamento por order_id quando necessário

### Items
- Conversão de valores numéricos
- Padronização de chaves

---

# 🟧 Silver → Gold (Modelagem Analítica)

### Gold — transações agregadas
- Join entre orders + payments + items
- Cálculo de GMV
- Ticket médio
- Indicadores por seller, cidade e método

### Gold — risco / chargeback
- Criação da taxa de chargeback
- Agrupamento por categoria (método, seller)
- Score simples baseado em taxa relativa

---

# Linhagem
Kaggle CSV
→ Bronze (raw)
→ Silver (cleaned)
→ Gold (analytics)

---

# Scripts
- `01_bronze_ingestion.py`
- `02_silver_cleaning.py`
- `03_gold_analytics.py`
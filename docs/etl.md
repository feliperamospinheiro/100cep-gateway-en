# Documentação do ETL — 100cep Gateway

Este documento descreve as transformações aplicadas em cada camada.

---

# 🟫 Bronze → Silver (Limpeza)

# 🟧 Silver → Gold (Modelagem Analítica)

---

# Linhagem
Kaggle CSV + datasets/ai_dataset/chargebacks_dataset.csv
→ Bronze (raw)
→ Silver (cleaned)
→ Gold (analytics)

---

# Scripts
- `01_bronze.py`
- `02_silver.py`
- `03_gold.py`
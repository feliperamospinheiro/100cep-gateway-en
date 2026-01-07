<h1 align="center">Documentação do ETL — 100cep Gateway</h1>

Este documento descreve detalhadamente o processo de **Extract, Transform, Load (ETL)** implementado no MVP, incluindo todas as transformações aplicadas em cada camada da arquitetura Medallion (Bronze → Silver → Gold).

---

## 📋 Visão Geral

O pipeline de dados segue a arquitetura **Medallion Architecture**, organizada em três camadas:

- **🥉 Bronze (Raw)**: Dados brutos, exatamente como foram coletados
- **🥈 Silver (Cleaned)**: Dados limpos, padronizados e validados
- **🥇 Gold (Analytics)**: Dados agregados e modelados para análise

### Fluxo de Dados

```
Kaggle (CSV) → Upload para UC Volumes → Bronze (Delta) → Silver (Delta) → Gold (Delta)
```

---

## 🥉 Camada Bronze — Ingestão

### Objetivo
Armazenar os dados brutos **sem nenhuma transformação**, garantindo:
- ✅ Auditabilidade completa
- ✅ Possibilidade de reprocessamento
- ✅ Histórico imutável da fonte original

### Processo de Ingestão

#### 1. Origem dos Dados
- **Dataset Principal**: Brazilian E-Commerce Public Dataset by Olist (Kaggle)
- **Dataset Adicional**: chargebacks_dataset.csv (gerado por IA)
- **Armazenamento**: Unity Catalog Volumes no Databricks

#### 2. Tabelas Criadas

| Tabela Bronze | Arquivo de Origem | Registros | Colunas |
|--------------|-------------------|-----------|---------|
| `bronze_customers` | olist_customers_dataset.csv | ~99k | 5 |
| `bronze_geolocation` | olist_geolocation_dataset.csv | ~1M | 5 |
| `bronze_orders` | olist_orders_dataset.csv | ~99k | 8 |
| `bronze_order_items` | olist_order_items_dataset.csv | ~112k | 7 |
| `bronze_order_payments` | olist_order_payments_dataset.csv | ~103k | 5 |
| `bronze_order_reviews` | olist_order_reviews_dataset.csv | ~99k | 7 |
| `bronze_products` | olist_products_dataset.csv | ~32k | 9 |
| `bronze_sellers` | olist_sellers_dataset.csv | ~3k | 4 |
| `bronze_product_category` | product_category_name_translation.csv | 71 | 2 |
| `bronze_chargebacks` | chargebacks_dataset.csv | ~1k | 5 |

#### 3. Transformações Aplicadas (Mínimas)

```python
# Normalização de nomes de colunas
# Exemplo: customer_id → customer_id (mantido)
# Exemplo: zip_code_prefix → zip_code_prefix (mantido)

# Inferência automática de schema
df = spark.read.csv(path, header=True, inferSchema=True)

# Persistência em Delta
df.write.format("delta").mode("overwrite").saveAsTable("bronze_table_name")
```

**Nota**: Tipos de dados podem estar incorretos nesta camada (ex: timestamps como strings).

---

## 🥈 Camada Silver — Limpeza e Padronização

### Objetivo
Transformar dados brutos em dados **confiáveis** e **consistentes**, aplicando:
- ✅ Correção de tipos de dados
- ✅ Tratamento de valores nulos
- ✅ Deduplicação
- ✅ Validação de domínios
- ✅ Padronização de nomenclaturas

### Transformações Detalhadas

#### 1. **silver_customers**

**Origem**: `bronze_customers`

**Transformações**:
```sql
-- Renomeação de colunas para português
customer_id → cliente_id
customer_zip_code_prefix → cep_prefixo

-- Validação de CEP (5 dígitos)
WHERE LENGTH(cep_prefixo) = 5

-- Remoção de duplicatas
DISTINCT ON (cliente_id)
```

**Qualidade**:
- ❌ Nulos removidos: customer_id
- ✅ Unicidade garantida: cliente_id
- ✅ Formato validado: cep_prefixo

---

#### 2. **silver_orders**

**Origem**: `bronze_orders`

**Transformações**:
```sql
-- Conversão de tipos temporais
CAST(order_purchase_timestamp AS TIMESTAMP)
CAST(order_approved_at AS TIMESTAMP)
CAST(order_delivered_carrier_date AS TIMESTAMP)
CAST(order_delivered_customer_date AS TIMESTAMP)
CAST(order_estimated_delivery_date AS TIMESTAMP)

-- Renomeação
order_id → pedido_id
customer_id → cliente_id
order_status → status_pedido

-- Filtros de qualidade
WHERE pedido_id IS NOT NULL
  AND cliente_id IS NOT NULL
  AND data_compra IS NOT NULL
```

**Qualidade**:
- ✅ Timestamps corrigidos de STRING para TIMESTAMP
- ❌ Pedidos sem ID ou cliente removidos
- ✅ Status de pedido validado (delivered, shipped, canceled, etc.)

---

#### 3. **silver_order_payments**

**Origem**: `bronze_order_payments`

**Transformações**:
```sql
-- Conversão numérica
CAST(payment_value AS DECIMAL(10,2))
CAST(payment_installments AS INT)

-- Renomeação
order_id → pedido_id
payment_type → tipo_pagamento
payment_value → valor_pagamento

-- Validação de domínio
WHERE tipo_pagamento IN ('credit_card', 'boleto', 'voucher', 'debit_card')
  AND valor_pagamento > 0

-- Agregação (múltiplos pagamentos por pedido)
GROUP BY pedido_id, tipo_pagamento
```

**Qualidade**:
- ✅ Valores negativos removidos
- ✅ Tipos de pagamento padronizados
- ✅ Agregação de pagamentos múltiplos

---

#### 4. **silver_order_items**

**Origem**: `bronze_order_items`

**Transformações**:
```sql
-- Conversão numérica
CAST(price AS DECIMAL(10,2))
CAST(freight_value AS DECIMAL(10,2))

-- Renomeação
order_id → pedido_id
seller_id → vendedor_id
price → preco
freight_value → frete

-- Agregação por pedido
SUM(preco) AS preco_total
SUM(frete) AS frete_total
```

**Qualidade**:
- ✅ Agregação de múltiplos itens por pedido
- ✅ Cálculo de totais (preço + frete)

---

#### 5. **silver_sellers**

**Origem**: `bronze_sellers`

**Transformações**:
```sql
-- Renomeação
seller_id → vendedor_id
seller_zip_code_prefix → cep_prefixo

-- Validação
WHERE vendedor_id IS NOT NULL
  AND LENGTH(cep_prefixo) = 5
```

---

#### 6. **silver_geolocation**

**Origem**: `bronze_geolocation`

**Transformações**:
```sql
-- Conversão geoespacial
CAST(geolocation_lat AS DOUBLE)
CAST(geolocation_lng AS DOUBLE)

-- Renomeação
geolocation_zip_code_prefix → cep_prefixo
geolocation_city → cidade
geolocation_state → estado
geolocation_lat → latitude
geolocation_lng → longitude

-- Deduplicação (múltiplas coordenadas por CEP)
-- Estratégia: manter mediana de lat/lng
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY latitude)

-- Validação de coordenadas
WHERE latitude BETWEEN -33.75 AND 5.27  -- Limites do Brasil
  AND longitude BETWEEN -73.99 AND -34.79
```

**Qualidade**:
- ✅ Coordenadas inválidas removidas
- ✅ Deduplicação por CEP (mediana geográfica)
- ✅ Estados padronizados (sigla de 2 letras)

---

#### 7. **silver_chargebacks**

**Origem**: `bronze_chargebacks`

**Transformações**:
```sql
-- Renomeação
order_id → pedido_id
chargeback_reason → motivo_chargeback
chargeback_status → status_chargeback
issuer_response → resposta_emissor
acquirer_response → resposta_adquirente

-- Validação de status
WHERE status_chargeback IN ('pending', 'approved', 'denied', 'investigating')

-- Enriquecimento com categorias
CASE 
  WHEN motivo_chargeback LIKE '%fraud%' THEN 'fraude'
  WHEN motivo_chargeback LIKE '%not received%' THEN 'nao_recebido'
  ELSE 'outros'
END AS categoria_chargeback
```

---

### Resumo de Qualidade — Silver

| Tabela | Registros Bronze | Registros Silver | % Perda | Principal Motivo |
|--------|-----------------|------------------|---------|------------------|
| customers | 99,441 | 99,441 | 0% | - |
| orders | 99,441 | 99,441 | 0% | - |
| order_payments | 103,886 | 99,440 | 4.3% | Agregação por pedido |
| order_items | 112,650 | 99,441 | 11.7% | Agregação por pedido |
| geolocation | 1,000,163 | 19,015 | 98.1% | Deduplicação por CEP |
| sellers | 3,095 | 3,095 | 0% | - |
| chargebacks | 1,000 | 1,000 | 0% | - |

---

## 🥇 Camada Gold — Modelagem Analítica

### Objetivo
Criar tabelas **dimensionais** e **fatos** otimizadas para análise de negócio.

### Modelo Dimensional

```
       ┌─────────────────┐
       │   dim_data      │
       ├─────────────────┤
       │ data_calendario │
       │ dia             │
       │ mes             │
       │ ano             │
       │ dia_semana      │
       └────────┬────────┘
                │
       ┌────────▼────────────────────────┐
       │      fato_transacoes            │
       ├─────────────────────────────────┤
       │ pedido_id (PK)                  │
       │ cliente_id (FK)                 │
       │ vendedor_id (FK)                │
       │ data_pedido (FK)                │
◄──────┤ tipo_pagamento                  │
       │ valor_transacao                 │
       │ preco_total                     │
       │ frete_total                     │
       │ status_pedido                   │
       └──┬──────────────┬───────────┬───┘
          │              │           │
┌─────────▼──────┐ ┌─────▼──────┐ ┌─▼────────────┐
│ dim_clientes   │ │dim_vendedor│ │dim_chargeback│
├────────────────┤ ├────────────┤ ├──────────────┤
│ cliente_id (PK)│ │vendedor_id │ │ pedido_id    │
│ cep_prefixo(FK)│ │cep_prefixo │ │ motivo_cb    │
└───────┬────────┘ └─────┬──────┘ │ status_cb    │
        │                │        │ resposta_*   │
        │  ┌─────────────▼────────┴┐
        │  │  dim_geolocalizacao   │
        └──►─────────────────────────┤
           │ cep_prefixo (PK)      │
           │ cidade                │
           │ estado                │
           │ latitude              │
           │ longitude             │
           └───────────────────────┘
```

### Tabelas Gold Criadas

#### **1. fato_transacoes**

**Query de criação**:
```sql
CREATE TABLE gold_fato_transacoes AS
SELECT 
  o.pedido_id,
  o.cliente_id,
  i.vendedor_id,
  DATE(o.data_compra) AS data_pedido,
  TIME(o.data_compra) AS horario_pedido,
  p.tipo_pagamento,
  p.valor_pagamento AS valor_transacao,
  i.preco_total,
  i.frete_total,
  o.status_pedido
FROM silver_orders o
INNER JOIN silver_order_payments p ON o.pedido_id = p.pedido_id
INNER JOIN silver_order_items i ON o.pedido_id = i.pedido_id
WHERE o.status_pedido = 'delivered'
```

**Métricas calculadas**:
- GMV (Gross Merchandise Value) = SUM(preco_total)
- Receita de Frete = SUM(frete_total)
- Ticket Médio = AVG(valor_transacao)
- Taxa de Conversão por método de pagamento

---

#### **2. dim_data**

**Query de criação**:
```sql
CREATE TABLE gold_dim_data AS
SELECT DISTINCT
  DATE(data_compra) AS data_calendario,
  DAY(data_compra) AS dia,
  MONTH(data_compra) AS mes,
  YEAR(data_compra) AS ano,
  DAYOFWEEK(data_compra) AS dia_semana_num,
  CASE DAYOFWEEK(data_compra)
    WHEN 1 THEN 'Domingo'
    WHEN 2 THEN 'Segunda'
    WHEN 3 THEN 'Terça'
    WHEN 4 THEN 'Quarta'
    WHEN 5 THEN 'Quinta'
    WHEN 6 THEN 'Sexta'
    WHEN 7 THEN 'Sábado'
  END AS nome_dia_semana,
  MONTHNAME(data_compra) AS nome_mes
FROM silver_orders
```

---

#### **3. dim_chargebacks**

Mantém informações de chargebacks já processadas na Silver.

---

#### **4. dim_clientes, dim_vendedores, dim_geolocalizacao**

Dimensões diretamente promovidas da camada Silver para Gold.

---

### Análises Implementadas (Gold)

#### **A. Método de Pagamento Mais Utilizado**
```sql
SELECT 
  tipo_pagamento,
  COUNT(*) AS total_transacoes,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS percentual
FROM gold_fato_transacoes
GROUP BY tipo_pagamento
ORDER BY total_transacoes DESC
```

#### **B. Faturamento Histórico 2017**
```sql
SELECT 
  d.nome_mes,
  d.mes,
  SUM(f.valor_transacao) AS faturamento_mensal
FROM gold_fato_transacoes f
JOIN gold_dim_data d ON f.data_pedido = d.data_calendario
WHERE d.ano = 2017
GROUP BY d.mes, d.nome_mes
ORDER BY d.mes
```

#### **C. Taxa de Chargeback**
```sql
SELECT 
  COUNT(DISTINCT CASE WHEN c.pedido_id IS NOT NULL THEN f.pedido_id END) AS pedidos_com_chargeback,
  COUNT(DISTINCT f.pedido_id) AS total_pedidos,
  ROUND(COUNT(DISTINCT CASE WHEN c.pedido_id IS NOT NULL THEN f.pedido_id END) * 100.0 / 
        COUNT(DISTINCT f.pedido_id), 2) AS taxa_chargeback_pct
FROM gold_fato_transacoes f
LEFT JOIN gold_dim_chargebacks c ON f.pedido_id = c.pedido_id
```

#### **D. Risco por Método de Pagamento**
```sql
SELECT 
  f.tipo_pagamento,
  COUNT(DISTINCT f.pedido_id) AS total_pedidos,
  COUNT(DISTINCT c.pedido_id) AS pedidos_chargeback,
  ROUND(COUNT(DISTINCT c.pedido_id) * 100.0 / COUNT(DISTINCT f.pedido_id), 2) AS taxa_cb_pct
FROM gold_fato_transacoes f
LEFT JOIN gold_dim_chargebacks c ON f.pedido_id = c.pedido_id
GROUP BY f.tipo_pagamento
ORDER BY taxa_cb_pct DESC
```

#### **E. Análise Geográfica de Chargebacks**
```sql
SELECT 
  g.estado,
  COUNT(DISTINCT f.pedido_id) AS total_pedidos,
  COUNT(DISTINCT c.pedido_id) AS pedidos_chargeback,
  ROUND(COUNT(DISTINCT c.pedido_id) * 100.0 / COUNT(DISTINCT f.pedido_id), 2) AS taxa_cb_pct
FROM gold_fato_transacoes f
JOIN gold_dim_clientes cl ON f.cliente_id = cl.cliente_id
JOIN gold_dim_geolocalizacao g ON cl.cep_prefixo = g.cep_prefixo
LEFT JOIN gold_dim_chargebacks c ON f.pedido_id = c.pedido_id
GROUP BY g.estado
ORDER BY taxa_cb_pct DESC
LIMIT 10
```

---

## 🔄 Linhagem de Dados

### Fluxo Completo

```
📦 Kaggle (CSV)
    ↓
📂 Unity Catalog Volumes (/Volumes/catalog/schema/volume/)
    ↓
🥉 Bronze Layer (Delta Tables - catalog.schema.bronze_*)
    ├─ bronze_customers
    ├─ bronze_orders
    ├─ bronze_order_payments
    ├─ bronze_order_items
    ├─ bronze_sellers
    ├─ bronze_geolocation
    ├─ bronze_products
    ├─ bronze_order_reviews
    ├─ bronze_product_category
    └─ bronze_chargebacks
    ↓
🥈 Silver Layer (Delta Tables - catalog.schema.silver_*)
    ├─ silver_customers     [limpeza + validação]
    ├─ silver_orders        [conversão temporal]
    ├─ silver_order_payments [agregação]
    ├─ silver_order_items    [agregação]
    ├─ silver_sellers       [padronização]
    └─ silver_geolocation   [deduplicação + validação geoespacial]
    ↓
🥇 Gold Layer (Delta Tables - catalog.schema.gold_*)
    ├─ gold_fato_transacoes      [join orders + payments + items]
    ├─ gold_dim_data             [dimensão temporal]
    ├─ gold_dim_clientes         [dimensão cliente]
    ├─ gold_dim_vendedores       [dimensão vendedor]
    ├─ gold_dim_geolocalizacao   [dimensão geográfica]
    └─ gold_dim_chargebacks      [dimensão chargeback]
    ↓
📊 Dashboards & Análises
```

---

## 🛠️ Scripts de Implementação

### Estrutura de Arquivos (Sugerida)

```
notebooks/
├── 01_bronze_ingestion.py
├── 02_silver_cleaning.py
├── 03_gold_modeling.py
└── 04_business_analytics.sql
```

### Ordem de Execução

1. **01_bronze_ingestion.py** - Carga inicial dos CSVs
2. **02_silver_cleaning.py** - Limpeza e transformações
3. **03_gold_modeling.py** - Criação de dimensões e fatos
4. **04_business_analytics.sql** - Queries analíticas

---

## ✅ Checklist de Qualidade

- [x] Dados brutos preservados (Bronze)
- [x] Tipos de dados corrigidos (Silver)
- [x] Nulos tratados adequadamente (Silver)
- [x] Duplicatas removidas (Silver)
- [x] Relacionamentos validados (Silver → Gold)
- [x] Modelo dimensional implementado (Gold)
- [x] Métricas de negócio calculadas (Gold)
- [x] Linhagem documentada
- [x] Queries de análise testadas

---
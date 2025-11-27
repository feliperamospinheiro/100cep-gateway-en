# Catálogo de Dados — 100cep Gateway

Este catálogo descreve os dados usados nas camadas Bronze, Silver e Gold, incluindo domínios, tipos e valores esperados.

---

# 🟫 Bronze (Raw)

| Tabela | Descrição |
|-------|-----------|
| bronze_orders | Dados brutos de pedidos |
| bronze_payments | Dados brutos de pagamentos |
| bronze_items | Dados brutos de itens do pedido |
| bronze_customers | Dados brutos de clientes |
| bronze_sellers | Dados brutos de sellers |

---

# 🟧 Silver (Cleaned)

## silver_orders
| Coluna | Tipo | Domínio | Descrição |
|--------|------|---------|-----------|
| order_id | string | UUID | Identificador do pedido |
| customer_id | string | UUID | Cliente |
| order_status | string | {delivered, shipped, canceled, ...} | Status |
| order_purchase_timestamp | timestamp | >=2016 | Momento da compra |
| order_delivered_customer_date | timestamp | null ou >= purchase | Entrega |

## silver_payments
| Coluna | Tipo | Domínio | Descrição |
|--------|------|---------|-----------|
| order_id | string | UUID existente | Chave com orders |
| payment_sequential | int | >= 1 | Número do pagamento |
| payment_type | string | {credit_card, boleto, voucher, debit_card} | Método |
| payment_installments | int | 1–24 | Parcelas |
| payment_value | double | >= 0 | Valor pago |

## silver_items
| Coluna | Tipo | Domínio | Descrição |
|--------|------|---------|-----------|
| order_id | string | UUID | Pedido |
| seller_id | string | UUID | Loja |
| price | double | >= 0 | Preço do item |
| freight_value | double | >= 0 | Frete |

## silver_customers
| Coluna | Tipo | Domínio | Descrição |
|--------|------|---------|-----------|
| customer_id | string | UUID | Cliente |
| customer_city | string | A–Z | Cidade |
| customer_state | string | 2 letras | Estado |

## silver_sellers
| Coluna | Tipo | Domínio | Descrição |
|--------|------|---------|-----------|
| seller_id | string | UUID | Loja |
| seller_city | string | A–Z | Cidade |
| seller_state | string | 2 letras | Estado |

---

# 🟨 Gold (Analytics)

## gold_transactions_summary
| Coluna | Descrição |
|--------|-----------|
| date | Data |
| total_orders | Pedidos realizados |
| total_payments | Pagamentos |
| gmv | Valor total processado |
| avg_ticket | Ticket médio |
| chargeback_rate | % Chargeback |

## gold_chargeback_risk
| Coluna | Descrição |
|--------|-----------|
| seller_id | Loja |
| total_orders | Pedidos |
| total_chargebacks | Disputas |
| chargeback_rate | Taxa |
| risk_score | Score calculado (0–1) |

---

# Linhagem de Dados
- Kaggle → Bronze → Silver → Gold  
- Transformações documentadas em `/docs/etl_documentation.md`.  
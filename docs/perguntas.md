<h1 align="center">Perguntas de Negócio — 100cep Gateway</h1>

Este documento lista todas as perguntas de negócio a serem respondidas pelo MVP de Engenharia de Dados. As perguntas estão organizadas por categoria e visam fornecer insights estratégicos para a operação da 100cep Gateway.

---

## 📊 Categorias de Análise

### 1️⃣ Comportamento de Pagamento
**Objetivo**: Entender as preferências de pagamento dos clientes e identificar padrões de uso.

**Pergunta 1**: Qual o método de pagamento mais utilizado pelos clientes da 100cep Gateway?

**Contexto**: Esta análise permite identificar os métodos de pagamento mais populares entre os clientes, auxiliando em decisões estratégicas sobre:
- Negociação de taxas com adquirentes
- Priorização de integrações com bandeiras
- Investimento em infraestrutura de processamento
- Estratégias de marketing focadas em métodos específicos

**Métricas esperadas**:
- Distribuição percentual por tipo de pagamento (credit_card, boleto, debit_card, voucher)
- Volume total de transações por método
- Ticket médio por método de pagamento

---

### 2️⃣ Performance Financeira
**Objetivo**: Monitorar a evolução do faturamento e identificar sazonalidades.

**Pergunta 2**: Qual o histórico de faturamento do ano de 2017?

**Contexto**: O acompanhamento do faturamento ao longo do tempo permite:
- Identificar períodos sazonais de alta/baixa demanda
- Planejar recursos operacionais
- Projetar crescimento futuro
- Avaliar efetividade de campanhas comerciais

**Métricas esperadas**:
- Faturamento mensal agregado
- Taxa de crescimento MoM (Month over Month)
- Comparação entre trimestres
- Análise de tendências

---

### 3️⃣ Risco e Chargebacks
**Objetivo**: Avaliar o nível de exposição a chargebacks e desenvolver estratégias de mitigação.

**Pergunta 3**: Qual a proporção de pedidos com e sem solicitação de chargeback?

**Contexto**: Compreender a taxa de chargeback é fundamental para:
- Avaliar o risco operacional da gateway
- Implementar regras de antifraude
- Provisionar reservas financeiras
- Negociar termos com adquirentes

**Métricas esperadas**:
- Taxa de chargeback (% de pedidos com chargeback)
- Volume absoluto de chargebacks
- Comparação com benchmarks do setor
- Impacto financeiro estimado

---

**Pergunta 4**: Quais métodos de pagamento têm maior risco de chargeback?

**Contexto**: A análise de risco por método de pagamento possibilita:
- Definir regras de análise de risco diferenciadas
- Ajustar precificação (MDR) por método
- Implementar verificações adicionais para métodos de alto risco
- Educar vendedores sobre práticas seguras

**Métricas esperadas**:
- Taxa de chargeback por método de pagamento
- Ranking de risco (do maior para o menor)
- Valor médio de chargeback por método
- Tempo médio até contestação

---

### 4️⃣ Análise Geográfica
**Objetivo**: Identificar regiões com maior risco e oportunidades de expansão.

**Pergunta 5**: Quais estados apresentam as maiores taxas de chargeback?

**Contexto**: A distribuição geográfica de chargebacks permite:
- Identificar regiões de maior risco
- Implementar regras de análise por localização
- Planejar estratégias regionais de expansão
- Compreender padrões de fraude por região

**Métricas esperadas**:
- Taxa de chargeback por estado (UF)
- Mapa de calor de risco
- Top 5 estados com maior/menor taxa
- Correlação com volume de vendas

---

## 🎯 Aplicações Práticas

As respostas a essas perguntas serão utilizadas para:

1. **Dashboard Executivo**: Visualização consolidada dos principais KPIs
2. **Alertas Automáticos**: Notificações quando métricas ultrapassarem limites
3. **Modelo Preditivo**: Base para machine learning em prevenção de fraudes
4. **Relatórios Regulatórios**: Dados para compliance e auditoria
5. **Business Intelligence**: Insights para tomada de decisão estratégica

---

## 📈 Camada de Dados

As análises serão realizadas utilizando as tabelas da **camada Gold**:
- `fato_transacoes`: Dados transacionais consolidados
- `dim_chargebacks`: Informações sobre contestações
- `dim_geolocalizacao`: Dados geográficos para análise regional
- `dim_data`: Dimensão temporal para análises históricas
- `dim_clientes` e `dim_vendedores`: Perfis de clientes e vendedores

---

## 🔄 Próximos Passos

Após responder essas perguntas fundamentais, o MVP poderá expandir para:
- Análise de cohort de clientes
- Previsão de chargeback com ML
- Análise de perfil de fraude
- Otimização de rotas de pagamento
- Segmentação de vendedores por performance

---
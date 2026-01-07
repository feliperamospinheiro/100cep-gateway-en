<h1 align="center">MVP de Engenharia de Dados</h1>

<p align="center">
  <a href="https://shields.io/">
    <img src="https://img.shields.io/badge/status-em%20desenvolvimento-yellow.svg" alt="Status">
  </a>
  <a href="https://www.databricks.com/">
    <img src="https://img.shields.io/badge/Databricks-Data%20Platform-orange?logo=databricks&logoColor=white" alt="Databricks">
  </a>
  <a href="https://en.wikipedia.org/wiki/Data_engineering">
    <img src="https://img.shields.io/badge/Engenharia%20de%20Dados-Data%20Engineering-blue" alt="Engenharia de Dados">
  </a>
  <a href="https://spark.apache.org/">
    <img src="https://img.shields.io/badge/Apache%20Spark-Spark-orange?logo=apachespark&logoColor=white" alt="Apache Spark">
  </a>
  <a href="https://www.postgresql.org/docs/">
    <img src="https://img.shields.io/badge/SQL-Query%20Language-blue?logo=postgresql&logoColor=white" alt="SQL">
  </a>
  <a href="https://pandas.pydata.org/docs/">
    <img src="https://img.shields.io/badge/Pandas-Data%20Analysis-purple?logo=pandas&logoColor=white" alt="Pandas">
  </a>
  <a href="https://seaborn.pydata.org/">
    <img src="https://img.shields.io/badge/Seaborn-Data%20Visualization-lightblue" alt="Seaborn">
  </a>
  <a href="https://geopandas.org/en/stable/">
    <img src="https://img.shields.io/badge/GeoPandas-Geospatial%20Data-green" alt="GeoPandas">
  </a>
</p>

O MVP simula o pipeline transacional da 100cep Gateway, incluindo ingestão, processamento, conciliação e chargebacks, seguindo padrões de adquirência e infraestrutura financeira.

Pipeline de dados construído no Databricks para simular o processamento de pedidos, pagamentos e chargebacks de uma empresa fictícia do setor de pagamentos, a **100cep Gateway**. 

O projeto segue boas práticas de Data Lakehouse, utilizando Delta Lake, Unity Catalog e a arquitetura **Bronze → Silver → Gold**.

---
<h2 align="center">100cep Gateway</h2>

<p align="center"> <img src="./docs/images/logo/100cep-gateway.png" alt="Logo 100cep Gateway" width="100%"></p>

A **100cep Gateway** é uma empresa fictícia de infraestrutura de pagamentos borderless, especializada em processar pagamentos globais de forma rápida, segura e interoperável.

Nosso objetivo é permitir **transações rápidas**, **seguras** e **sem fronteiras** — afinal, somos _100cep_: sem _cidade_, _estado_ ou _país_ limitando o fluxo dos pagamentos.

### 🔑 Principais Características
- 🌍 **Pagamentos Globais**: Processamento sem restrições geográficas
- ⚡ **Alta Performance**: Infraestrutura preparada para alto volume de transações
- 🔒 **Segurança**: Monitoramento de fraude e chargebacks em tempo real
- 📊 **Analytics**: Dashboards e métricas para tomada de decisão

---

<h2 align="center">🛠️ Tecnologias Utilizadas</h2>

### Plataforma e Armazenamento
- **Databricks**: Plataforma de dados unificada
- **Delta Lake**: Formato de armazenamento transacional
- **Unity Catalog**: Governança e catalogação de dados
- **Volumes UC**: Armazenamento de arquivos brutos

### Processamento e Análise
- **Apache Spark**: Engine de processamento distribuído
- **PySpark**: API Python para Spark
- **SQL**: Queries analíticas e transformações
- **Pandas**: Análise exploratória de dados

### Visualização e Modelagem
- **Seaborn**: Visualizações estatísticas
- **Matplotlib**: Gráficos e plots
- **GeoPandas**: Análises geoespaciais
- **dbdiagram.io**: Modelagem de dados

---
<h2 align="center">🎯 Objetivo do Projeto</h2>

Este MVP tem como objetivo construir um pipeline de engenharia de dados completo para:

- ingerir dados transacionais de e-commerce;  
- padronizar, relacionar e organizar entidades (pedidos, pagamentos, itens, clientes, sellers);  
- gerar camadas analíticas para monitoramento de risco, antifraude e chargebacks;  
- responder perguntas de negócio típicas de empresas de pagamentos, adquirentes e gateways.

O foco central é entender:

> **Como a 100cep Gateway pode monitorar, conciliar e antecipar ocorrências de pagamentos e chargebacks utilizando dados transacionais?**

Todas as perguntas de negócio estão documentadas em:  
📄 [/docs/perguntas.md](docs/perguntas.md)

---

<h2 align="center">Coleta dos Dados</h2>

Os dados utilizados foram obtidos no Kaggle (**Brazilian E-Commerce Public Dataset by Olist**), amplamente usado em estudos e projetos educacionais.

Processo adotado:

1. Download manual dos arquivos CSV.
2. Upload para o **Unity Catalog Volumes** no Databricks, garantindo:
   - armazenamento em nuvem,
   - versionamento pelo UC,
   - padronização da ingestão no nível Bronze.

⚠ Não houve uso de web scraping ou dados sensíveis.  
⚠ Nenhum dado interno ou confidencial de empresas reais foi utilizado.

📸 **Evidências**: Screenshots do processo de coleta estão disponíveis na pasta `/docs/images/databricks/`.

---

<h2 align="center">Modelagem de Dados</h2>

Foi adotado um modelo **Lakehouse** com tabelas **flat por conceito**:

### 🥉 Bronze
- Armazenamento dos arquivos *exatamente como chegaram*.
- Sem limpeza, sem inferência, sem padronização.
- Garantia de auditabilidade.

### 🥈 Silver
- Padronização de tipos
- Deduplicação
- Tratamento de nulos
- Correção de colunas derivadas
- Relação entre entidades (join lógico)

### 🥇 Gold
- Tabelas analíticas orientadas ao negócio
- KPIs de chargebacks, GMV, ticket médio
- Modelos por método de pagamento, seller e região

### 📄 Catálogo de Dados
Foi criado um **Data Catalog** contendo:

- Nome da coluna  
- Tipo de dado  
- Domínio esperado  
- Valores mínimos e máximos (numéricos)  
- Categorias possíveis (categóricos)  
- Descrição funcional  
- Camada de origem  
- Linhagem Bronze → Silver → Gold

📄 **Documentação completa**: [/docs/data_catalog.md](docs/data_catalog.md)

---
<h2 align="center">Carga (ETL / ELT)</h2>

A carga foi estruturada em três passos principais:

### 1) Ingestão (Bronze)
- Leitura dos CSVs diretamente do Volume UC  
- Persistência em Delta  
- Normalização de nomes de colunas

### 2) Transformação (Silver)
- Conversão de tipos datetime  
- Correção de colunas categóricas  
- Padronização de campos numéricos  
- Exclusão de duplicadas  
- Consolidação de tabelas relacionadas

### 3) Modelagem Analítica (Gold)
- Tabelas agregadas  
- Métricas de operação e risco  
- Junções entre pedidos, pagamentos e chargebacks

📄 **Documentação completa do ETL**: [/docs/etl.md](docs/etl.md)  
📸 **Evidências de execução**: Screenshots disponíveis em `/docs/images/databricks/`

---
<h2 align="center">Análises Realizadas</h2>

## 🔍 a) Qualidade dos Dados
Foi feita uma análise de:

- valores ausentes  
- valores fora do domínio  
- inconsistências entre tabelas  
- dados duplicados  
- erros de formato  

As correções foram aplicadas na camada Silver, garantindo:
- ✅ Dados consistentes e confiáveis
- ✅ Tipos de dados corretos
- ✅ Valores dentro dos domínios esperados
- ✅ Relacionamentos íntegros entre tabelas

📸 **Evidências**: Screenshots disponíveis em `/docs/images/databricks/`

---

## 🧠 b) Solução do Problema (Perguntas de Negócio)

As análises Gold respondem perguntas como:

- **Qual o método de pagamento mais utilizado pelos clientes da 100cep Gateway?** 
- **Qual o histórico de faturamento do ano de 2017?**  
- **Qual a proporção de pedidos com e sem solicitação de chargeback?**  
- **Quais métodos de pagamento têm maior risco de chargeback?**  
- **Quais estados apresentam as maiores taxas de chargeback?**  

As respostas detalhadas estão em:  
📄 [/docs/perguntas.md](docs/perguntas.md)

### 📊 Principais Insights
- Credit card é o método de pagamento predominante
- Taxa de chargeback varia significativamente por estado
- Correlação entre método de pagamento e risco de chargeback
- Padrões sazonais no faturamento de 2017

---
<h2 align="center">Autoavaliação</h2>

Discussão final sobre:

- objetivos atingidos e não atingidos;  
- dificuldades enfrentadas;  
- limitações naturais do MVP;  
- melhorias e próximos passos (streaming, automação, dashboards, monitoramento).

📄 **Documentação completa**: [/docs/autoavaliação.md](docs/autoavaliação.md)

---

<h2 align="center">Autor</h2>

**Felipe Pinheiro**  

[![Gmail](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:felipervmospinheiro@gmail.com)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/feliperamospinheiro)

<h2 align="center">Creditos</h2>

Dataset: *[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)*

Autor: Olist & André Sionek

DOI Citation: *[DOI](https://doi.org/10.34740/kaggle/dsv/195341)*

Licença: *[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)*

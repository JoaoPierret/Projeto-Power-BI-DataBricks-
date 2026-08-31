# 🌾 Observatório e Análise de Risco do Crédito Rural no RS (2020–2026)

[![Databricks](https://img.shields.io/badge/Databricks-Serverless-FF3621?logo=databricks&logoColor=white)](https://databricks.com/)
[![Apache Spark](https://img.shields.io/badge/Apache_Spark-PySpark-E25A1C?logo=apachespark&logoColor=white)](https://spark.apache.org/)
[![Delta Lake](https://img.shields.io/badge/Storage-Delta_Lake-00ADD8?logo=delta)](https://delta.io/)
[![Power BI](https://img.shields.io/badge/Visualização-Power_BI-F2C811?logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://python.org)

Pipeline de Engenharia e Análise de Dados desenvolvido em nuvem no **Databricks**, implementando a **Arquitetura Medallion** e modelagem dimensional **Star Schema (Kimball)**. O projeto analisa a concessão de crédito de custeio agropecuário, os impactos dos choques climáticos (estiagens severas de 2022/2023 e inundações históricas de 2024) e a exposição a risco nas 14 macrorregiões produtivas do Rio Grande do Sul.

---

## 🏗️ Arquitetura Medallion & Evidências de Execução no Databricks

### 1. Camada Bronze (Ingestão Raw)
* **Fontes:** API Olinda (MDCR v2 / SICOR - Banco Central) e API SGS (Séries Temporais 21086 - Inadimplência Rural PF e 4390 - Selic Mensal).
* **Persistência:** Tabelas Delta `bronze.mdcr_custeio_rs_raw`, `bronze.sgs_inadimplencia_raw` e `bronze.sgs_selic_raw`.

![Execução Bronze](assets/execucoes_databricks/parte1_camada_bronze.png)

---

### 2. Camada Silver (Tratamento, Regras de Negócio e Governança)
* **Transformações PySpark:** Padronização de strings, remoção de acentos, tipagem de dados.
* **Mapeamento Regional:** Agrupamento de municípios nas 14 Macrorregiões do RS (*Vales do Rio Pardo, Vales do Taquari, Missões, Fronteira Oeste, Campanha, Noroeste Colonial, Celeiro, Produção, Alto Jacuí, Baixo Jacuí, Alto Uruguai, Serra, Campos de Cima da Serra, Sul, Grande Porto Alegre e Litoral Norte*).
* **Modelagem de Safras & Risco:** Cálculo de ano-agrícola (julho a junho) e classificação automatizada de causas de risco climático (Estiagem vs. Enchentes).

![Execução Silver](assets/execucoes_databricks/parte2_camada_silver.png)

---

### 3. Camada Gold (Modelagem Dimensional Star Schema)
* **Modelagem Kimball:** Geração de Surrogate Keys (`sk_regiao`, `sk_cultura`, `sk_tempo`) e consolidação da tabela fato.
* **Tabelas Finais:** `gold.dim_regiao_rs`, `gold.dim_cultura`, `gold.dim_tempo_safra` e `gold.fato_custeio_risco_regional_rs`.

![Execução Gold](assets/execucoes_databricks/parte3_camada_gold.png)

---

## 🗄️ Modelagem Star Schema
```mermaid
erDiagram
    dim_regiao_rs ||--o{ fato_custeio_risco_regional_rs : "1 : N"
    dim_cultura ||--o{ fato_custeio_risco_regional_rs : "1 : N"
    dim_tempo_safra ||--o{ fato_custeio_risco_regional_rs : "1 : N"

    dim_regiao_rs {
        int sk_regiao PK
        string nome_regiao
    }

    dim_cultura {
        int sk_cultura PK
        string nome_cultura
    }

    dim_tempo_safra {
        date sk_tempo PK
        int ano
        int mes
        string ano_mes
        string safra_agricola
        string diagnostico_causa_risco
    }

    fato_custeio_risco_regional_rs {
        date fk_tempo FK
        int fk_regiao FK
        int fk_cultura FK
        string diagnostico_causa_risco
        double total_custeio_contratado
        int total_contratos
        double taxa_inadimplencia_referencia_bcb
        double taxa_selic_mensal
    }
```
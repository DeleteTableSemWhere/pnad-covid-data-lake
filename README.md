# Data Lake PNAD-COVID: Inteligência em Saúde Pública

![Status do Projeto](https://img.shields.io/badge/Status-Em_Andamento-yellow)
![Nuvem](https://img.shields.io/badge/Cloud-AWS-orange)
![Arquitetura](https://img.shields.io/badge/Arquitetura-Medallion-blue)
![License](https://img.shields.io/badge/License-MIT-green)

> **Desenvolvido por [DeleteTableSemWhere](https://github.com/DeleteTableSemWhere)**

## Sobre o Projeto

Este projeto é o **Tech Challenge da Fase 3** da Pós-Tech em Data Analytics da FIAP. O objetivo é atuar como consultoria de dados para um grande hospital, analisando a pesquisa **PNAD-COVID-19** do IBGE para construir um **Plano de Contingência Estratégico** em caso de um novo surto da doença.

Diferente de análises locais em planilhas, este projeto implementa uma **Arquitetura de Data Lake em Nuvem (AWS)**, projetada para processar microdados massivos (Big Data) e extrair inteligência acionável através de modelagem SQL e Business Intelligence.

---

## Principais Destaques

* **Arquitetura Medallion na AWS:** Implementação completa de um Data Lake com camadas Raw, Bronze e Silver no Amazon S3.
* **Processamento Distribuído (Big Data):** Utilização de Jobs **PySpark** no AWS Glue para converter e particionar arquivos CSV massivos (milhões de linhas) para o formato colunar `.parquet`.
* **Tratamento de Metadados Complexos:** Limpeza avançada de dicionários do IBGE (arquivos `.xls` legados) usando **Pandas**, resolvendo problemas estruturais (células mescladas, *forward fill*) e inconsistências de tipagem antes da ingestão.
* **Modelagem SQL Escalável:** Criação de *Views* no **Amazon Athena** (motor Presto/Trino) para traduzir códigos numéricos do IBGE em informações de negócio, garantindo performance em consultas ad-hoc.
* **Escopo Analítico Cirúrgico:** Seleção estratégica de apenas **20 variáveis** (Pilares: Demografia, Sintomas, Sobrecarga, Comportamento e Economia), analisando um recorte temporal específico de 3 meses (Julho, Agosto, Setembro).

---

## Engenharia de Dados e Pipeline (AWS)

O projeto segue um pipeline robusto e auditável na nuvem da Amazon Web Services:

### 1. Ingestão e Camada Raw (`Data Ingestion`)
* **Fonte:** Arquivos públicos da pesquisa PNAD-COVID-19 (IBGE).
* **Armazenamento:** Amazon S3 (`data_input/`).
* **Estrutura:** Microdados mensais em `.csv` e Dicionário de Variáveis em `.xls`.

### 2. Processamento e Camada Bronze (`Data Transformation`)
* **Dicionário (Pandas):** Script executado via AWS Glue Interactive Sessions para limpar e padronizar os metadados. Tratamento de *schema enforcement* forçando colunas para `String` para evitar erros de `ArrowInvalid`.
* **Microdados (PySpark):** Job de ETL distribuído para leitura dos arquivos massivos, conversão e particionamento dos dados para `.parquet`, garantindo compressão e otimização de leitura. Salvamento no Amazon S3 (`data_output/bronze/`).

### 3. Catálogo de Dados e Camada Silver (`Data Modeling`)
* **Catálogo:** Configuração de **AWS Glue Crawlers** para inferir os esquemas (schemas) dos arquivos Parquet e registrar as tabelas no AWS Glue Data Catalog (`pnad_bronze`).
* **Modelagem:** Uso do **Amazon Athena** para criar a *View* da camada Silver (`vw_silver_covid_hospital`). Esta camada traduz os microdados brutos (ex: `A004 = 1`) para dimensões de negócio (ex: `cor_raca = 'Branca'`) e aplica o filtro temporal dos 3 meses de análise.

### 4. Análise e Visualização (`Business Intelligence`) - *[Em Andamento]*
* **Ferramenta:** Power BI conectado diretamente ao Amazon Athena via ODBC/DirectQuery.
* **Objetivo:** Responder às 20 perguntas norteadoras definidas no documento de escopo.

---

## O Plano de Contingência (Entregáveis)

A partir da infraestrutura construída, a análise final responderá a perguntas vitais para a diretoria do hospital, divididas em 6 pilares:

1.  **Demografia:** Quais estados e faixas etárias demandaram mais leitos de UTI?
2.  **Sintomas Clínicos:** Quais combinações de sintomas (ex: perda de olfato + dificuldade de respirar) são os maiores preditores de internação?
3.  **Sobrecarga Hospitalar:** Qual a proporção de pacientes que buscaram o SUS vs. Hospitais Privados?
4.  **Testagem:** Qual a taxa real de positividade para calcular o avanço do surto?
5.  **Isolamento:** Como a quebra da restrição de contato prevê picos de demanda hospitalar nas semanas seguintes?
6.  **Impacto Econômico:** Como a impossibilidade de *home office* transformou trabalhadores em vetores primários de transmissão?

> **Nota:** Os resultados detalhados e o Plano de Ação final serão documentados aqui após a conclusão da etapa de Business Intelligence.

---

## Autores

Desenvolvido como parte do Tech Challenge (Fase 3) da Pós-Tech Data Analytics (FIAP).

<div align="center">
<table>
  <tr>
    <td align="center">
      <a href="https://github.com/BrunoAssis12">
        <img src="https://github.com/BrunoAssis12.png" width="100px;" alt=""/>
        <br /><sub><b>Bruno Assis</b></sub>
      </a><br />
      🚀 Data Scientist
    </td>
    <td align="center">
      <a href="https://github.com/gnunes-io">
        <img src="https://github.com/gnunes-io.png" width="100px;" alt=""/>
        <br /><sub><b>Gabriel Nunes</b></sub>
      </a><br />
      📊 Data Analyst
    </td>
    <td align="center">
      <a href="https://github.com/Jonathan-Paixao">
        <img src="https://github.com/Jonathan-Paixao.png" width="100px;" alt=""/>
        <br /><sub><b>Jonathan Paixão</b></sub>
      </a><br />
      📊 Analytic Engineer
    </td>
    <td align="center">
      <a href="https://github.com/rafaelvieiravidal-glitch">
        <img src="https://github.com/rafaelvieiravidal-glitch.png" width="100px;" alt=""/>
        <br /><sub><b>Rafael Vieira</b></sub>
      </a><br />
      📉 Quant
    </td>
    <td align="center">
      <a href="#">
        <img src="https://github.com/ghost.png" width="100px;" alt=""/>
        <br /><sub><b>Wagner da Silva</b></sub>
      </a><br />
      🧠 Data Engineer
    </td>
  </tr>
</table>
</div>

---

<br>

## 🛡️ Aviso Legal
Este projeto tem fins estritamente educacionais e acadêmicos. As análises geradas não constituem recomendações médicas oficiais.

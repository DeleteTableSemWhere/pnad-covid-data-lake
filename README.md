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

* **Arquitetura Medallion na AWS:** Implementação completa de um Data Lake com camadas Raw, Bronze, Silver e Gold no Amazon S3 e Athena.
* **Processamento Distribuído (Big Data):** Utilização de Jobs **PySpark** no AWS Glue para converter e particionar arquivos CSV massivos (milhões de linhas) para o formato colunar `.parquet`.
* **Tratamento de Metadados Complexos:** Limpeza avançada de dicionários do IBGE (arquivos `.xls` legados) usando **Pandas**, resolvendo problemas estruturais (células mescladas, *forward fill*) e inconsistências de tipagem antes da ingestão.
* **Fronteira de Agregação no Athena:** Delegação de todo o processamento de regras de negócio (Camadas Silver e Gold) para o motor do **Amazon Athena**, mantendo a ferramenta de BI estritamente focada na visualização.
* **Escopo Analítico Cirúrgico:** Seleção estratégica de apenas **20 variáveis** (Pilares: Demografia, Sintomas, Sobrecarga, Comportamento e Economia), analisando um recorte temporal específico de 3 meses (Julho, Agosto, Setembro).

---

## Engenharia de Dados e Pipeline (AWS)

O projeto segue um fluxo de dados (pipeline) automatizado e escalável na Amazon Web Services:

### 1. Ingestão e Camada Raw (`Data Ingestion`)
* **Fonte:** Arquivos públicos da pesquisa PNAD-COVID-19 via FTP do IBGE.
* **Processo:** Um script no **AWS Glue (Python)** conecta-se diretamente ao FTP, realiza a extração dos dados brutos e os descarrega no Amazon S3 (`data_input/`).

### 2. Processamento e Camada Bronze (`Data Transformation`)
* **Dicionário (Pandas):** Script executado via AWS Glue Notebook para limpar e padronizar os metadados. Tratamento de *schema enforcement* forçando colunas para `String` para evitar erros de leitura (`ArrowInvalid`).
* **Microdados (PySpark):** Job de ETL distribuído para leitura dos arquivos massivos, conversão e particionamento dos dados para `.parquet`, garantindo compressão e otimização de leitura. Salvamento no Amazon S3 (`data_output/bronze/`).

### 3. Orquestração e Catálogo de Dados (`Data Catalog`)
* **Automação:** Utilização de orquestração baseada em eventos (via **AWS Glue Workflows / EventBridge**) para disparar automaticamente o **AWS Glue Crawler** assim que os Jobs de processamento são finalizados.
* **Catálogo:** O Crawler infere os esquemas dos arquivos Parquet e os registra no banco de dados virtual `pnad_bronze`.

### 4. Modelagem SQL: Camadas Silver e Gold (`Data Modeling`)
Toda a modelagem foi construída em SQL utilizando o motor Presto/Trino do **Amazon Athena**:
* **Silver:** A *View* `vw_silver_covid_hospital` traduz os microdados (ex: `A004 = 1` para `cor_raca = 'Branca'`), aplica o filtro temporal (meses 9, 10 e 11) e padroniza a tipagem.
* **Gold:** Criação de *Views* agregadas adicionais contendo as métricas de negócio e KPIs finais prontas para consumo, otimizando o custo de processamento e volume de dados trafegados.

### 5. Análise e Visualização (`Business Intelligence`) - *[Em Andamento]*
* **Ferramenta:** Microsoft Power BI.
* **Conexão:** Integração com o Amazon Athena realizada via **Driver ODBC**, consumindo diretamente a camada Gold.
* **Objetivo:** Renderizar os gráficos para responder às 20 perguntas norteadoras definidas no documento de escopo.

---

## 📊 O Plano de Contingência (Entregáveis)

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
      🐍 Analytic Engineer
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

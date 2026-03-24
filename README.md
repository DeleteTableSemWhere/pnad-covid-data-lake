# Data Lake PNAD-COVID: Inteligência em Saúde Pública

![Status do Projeto](https://img.shields.io/badge/Status-Em_Andamento-yellow)
![Nuvem](https://img.shields.io/badge/Cloud-AWS-orange)
![Arquitetura](https://img.shields.io/badge/Arquitetura-Medallion-blue)
![License](https://img.shields.io/badge/License-MIT-green)

> **Desenvolvido por [DeleteTableSemWhere](https://github.com/DeleteTableSemWhere)**

## Sobre o Projeto

Este projeto é o **Tech Challenge da Fase 3** da Pós-Tech em Data Analytics da FIAP. O objetivo é atuar como consultoria de dados para um grande hospital, analisando a pesquisa **PNAD-COVID-19** do IBGE para construir um **Plano de Contingência Estratégico** em caso de um novo surto da doença.

Diferente de análises locais em planilhas, este projeto implementa uma **Arquitetura de Data Lake em Nuvem (AWS)**, projetada para processar microdados massivos (Big Data) e extrair inteligência acionável através de processamento distribuído (PySpark) e Business Intelligence.

---

## Principais Destaques

* **Arquitetura Medallion na AWS:** Implementação completa de um pipeline automatizado com camadas Raw, Bronze, Silver e Gold no Amazon S3.
* **ETL Distribuído com PySpark:** Utilização de 4 Jobs no **AWS Glue** para orquestrar desde a extração no FTP governamental até a consolidação final.
* **Desnormalização Dinâmica (Zero Hardcode):** A tradução dos códigos do IBGE (ex: `11` = `Rondônia`) não é feita manualmente. O Job Gold constrói um *lookup dictionary* dinâmico a partir dos metadados e injeta as regras de negócio via PySpark, garantindo resiliência a mudanças no questionário.
* **Escopo Analítico Cirúrgico:** Filtragem semântica focada em responder **20 perguntas norteadoras estratégicas** (Pilares: Demografia, Sintomas, Sobrecarga, Comportamento e Economia), analisando o trimestre de Setembro, Outubro e Novembro de 2020.

---

## Engenharia de Dados e Pipeline (AWS)

O projeto segue um fluxo de dados (pipeline) estruturado em 4 etapas no AWS Glue:

### 1. Ingestão e Camada Raw (`Job 01 - Coleta`)
* **Processo:** Script Python (Boto3/Requests) executado via Glue para extrair os microdados (`.zip`) e a documentação (`.xls`) diretamente do FTP do IBGE, descarregando-os intactos no Amazon S3 (`data_input/`).

### 2. Processamento e Camada Bronze (`Job 02 - Bronze`)
* **Microdados:** O job extrai o CSV de dentro do ZIP dinamicamente, infere o schema e grava no formato colunar `.parquet` (`data_output/bronze/`).
* **Resiliência:** O código possui detecção e adequação automática para garantir a leitura independentemente do separador utilizado pelo IBGE.

### 3. Limpeza e Camada Silver (`Job 03 - Silver`)
* **Dicionário (Pandas):** Uso da biblioteca `xlrd` (injetada no cluster via `%additional_python_modules`) para parsear o XLS. Aplicação de *forward fill* em células mescladas e tratamento de metadados, transformando o dicionário na *fonte da verdade* mapeada em Parquet.
* **Microdados (PySpark):** Seleção de colunas, renomeação semântica (ex: `B0011` → `sintoma_febre`) e padronização de códigos nulos (`9` e `99`).

### 4. Consolidação e Camada Gold (`Job 04 - Gold`)
* **Processo:** O PySpark cruza os microdados (Silver) com o dicionário (Silver), aplicando a desnormalização de forma programática.
* **Entrega:** Consolida os 3 meses de dados em um único diretório Parquet (`pnad_covid_consolidado/`), 100% desnormalizado e pronto para o consumo do BI.
* **Catálogo:** O **AWS Glue Crawler** mapeia essa camada final e a disponibiliza no **Amazon Athena** sob o banco `techchall_covid`.

---

## Qualidade de Dados e Resiliência Analítica

Durante a Análise Exploratória (EDA) e construção do pipeline, implementamos lógicas avançadas de qualidade de dados:
* **Compreensão de Nulos Condicionais:** Identificamos que altas taxas de dados nulos (ex: >95% na variável de resultado de teste Swab) não representam falha técnica, mas sim o **design de saltos do questionário do IBGE**. Variáveis clínicas específicas só eram respondidas se o paciente afirmasse ter buscado a rede de saúde, exigindo tratamento lógico na camada Silver para não distorcer as métricas do hospital.
* **Normalização de Zeros à Esquerda:** Implementação de `lpad` no PySpark para garantir que chaves de cruzamento (ex: `"4"` no dicionário e `"04"` no microdado) realizem o *Join* perfeitamente.

---

## O Plano de Contingência (Entregáveis)

A partir da view consolidada na camada Gold, o painel no Power BI responderá a perguntas vitais para a diretoria do hospital:

1.  **Demografia:** Quais estados e faixas etárias demandaram mais leitos de UTI?
2.  **Sintomas Clínicos:** Quais combinações de sintomas (ex: perda de olfato + dificuldade de respirar) são os maiores preditores de internação?
3.  **Sobrecarga Hospitalar:** Qual a proporção de pacientes que buscaram o SUS vs. Hospitais Privados?
4.  **Testagem e Isolamento:** Como a quebra da restrição de contato prevê picos de demanda hospitalar nas semanas seguintes?
5.  **Impacto Econômico:** Como a impossibilidade de *home office* transformou trabalhadores em vetores primários de transmissão?

> **Nota:** Os dashboards finais e o Plano de Ação Estratégico em PDF serão documentados e anexados aqui após a conclusão da etapa de Business Intelligence.

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

# Data Lake PNAD-COVID: Inteligência em Saúde Pública

![Status do Projeto](https://img.shields.io/badge/Status-Finalizado-success)
![Nuvem](https://img.shields.io/badge/Cloud-AWS-orange)
![Arquitetura](https://img.shields.io/badge/Arquitetura-Medallion-blue)
![License](https://img.shields.io/badge/License-MIT-green)

> **Desenvolvido por [DeleteTableSemWhere](https://github.com/DeleteTableSemWhere)**

## Sobre o Projeto

Este projeto é o **Tech Challenge da Fase 3** da Pós-Tech em Data Analytics da FIAP. [cite_start]O objetivo é atuar como consultoria de dados para um grande hospital, analisando a pesquisa **PNAD-COVID-19** do IBGE para construir um **Plano de Contingência Estratégico** em caso de um novo surto da doença[cite: 100].

Diferente de análises locais em planilhas, este projeto implementa uma **Arquitetura de Data Lake em Nuvem (AWS)**, projetada para processar microdados massivos (Big Data) e extrair inteligência acionável através de processamento distribuído (PySpark) e Business Intelligence. [cite_start]Analisamos uma base consolidada de 1.149.197 registros individuais cobrindo os meses de Setembro, Outubro e Novembro de 2020[cite: 101, 105].

---

## Principais Destaques

* [cite_start]**Arquitetura Medallion na AWS:** Implementação completa de um pipeline automatizado com camadas Raw, Bronze, Silver e Gold no Amazon S3[cite: 152, 155, 157, 159].
* [cite_start]**ELT Distribuído com PySpark:** Utilização de 4 Jobs no **AWS Glue** para orquestrar desde a extração no FTP governamental até a consolidação final[cite: 154].
* **Desnormalização Dinâmica (Zero Hardcode):** A tradução dos códigos do IBGE (ex: `11` = `Rondônia`) não é feita manualmente. [cite_start]O Job Gold constrói um *lookup dictionary* dinâmico a partir dos metadados e injeta as regras de negócio via PySpark, garantindo resiliência a mudanças no questionário[cite: 164, 165].
* [cite_start]**Escopo Analítico Cirúrgico:** Filtragem semântica focada em responder **20 perguntas norteadoras estratégicas**, divididas em 3 vertentes: Características da População, Quadro Clínico e Impacto Econômico[cite: 104].

---

## Engenharia de Dados e Pipeline (AWS)

O projeto segue um fluxo de dados (pipeline) estruturado em 4 etapas no AWS Glue:

### 1. Ingestão e Camada Raw (`Job 01 - Coleta`)
* [cite_start]**Processo:** Script Python executado via Glue para extrair os microdados (`.zip`) e a documentação (`.xls`) diretamente do FTP do IBGE, descarregando-os intactos no Amazon S3 (`data_input/`)[cite: 153, 161].

### 2. Processamento e Camada Bronze (`Job 02 - Bronze`)
* [cite_start]**Microdados:** O job extrai o CSV de dentro do ZIP dinamicamente, infere o schema e grava no formato colunar `.parquet` (`data_output/bronze/microdados/`) com 146 colunas[cite: 162].
* **Documentação:** Os arquivos de dicionário (`.xls`) são copiados para `data_output/bronze/documentacao/`, preservando o insumo original para enriquecimento posterior.
* **Resiliência:** O código trata variações de estrutura dos arquivos de origem para manter robustez na leitura.

### 3. Limpeza, Enriquecimento e Camada Silver (`Job 03 - Silver`)
* [cite_start]**Microdados (PySpark):** Seleção de 42 variáveis semânticas, renomeação (ex: `B0011` → `sintoma_febre`) e padronização de códigos nulos[cite: 163].
* [cite_start]**Lookup de dicionário (Pandas + Spark):** O dicionário é lido diretamente do Bronze (`data_output/bronze/documentacao/`), tratado (*forward fill*, limpeza e normalização) e convertido para lookup em Spark colunar[cite: 164].
* **Join na Silver:** A desnormalização é aplicada na própria Silver via join dinâmico entre microdados e lookup, incluindo normalização de chaves com `lpad`.
* **Saída:** A Silver grava apenas os microdados já enriquecidos em `data_output/silver/microdados/`.

### 4. Consolidação e Camada Gold (`Job 04 - Gold`)
* [cite_start]**Processo:** A Gold passa a focar somente na consolidação dos datasets já enriquecidos na Silver[cite: 165].
* [cite_start]**Entrega:** Consolida os 3 meses em um único diretório Parquet (`pnad_covid_consolidado/`), desnormalizado e pronto para consumo[cite: 165].
* [cite_start]**Catálogo e Consulta:** O **AWS Glue Crawler** mapeia essa camada final no Glue Data Catalog, permitindo consultas via **Amazon Athena** (motor Presto/Trino) para alimentar os dashboards no Power BI / Matplotlib[cite: 166, 167, 168, 171].

---

## 📊 Resultados e Insights Analíticos

A análise dos microdados revelou o verdadeiro comportamento da pandemia e gerou insights cruciais, divididos em 3 vertentes:

### 1. Características da População (Vulnerabilidade)
* [cite_start]**Dupla e Tripla Vulnerabilidade:** Estados do Norte e Nordeste apresentaram maior prevalência de sintomas combinada com menor acesso a EPIs (máscaras e álcool gel)[cite: 194]. [cite_start]Populações pretas e pardas sofreram tripla vulnerabilidade: menor cobertura de plano de saúde, menor possibilidade de isolamento e maior positividade real[cite: 275].
* [cite_start]**Exposição Ativa:** A maior concentração de casos positivos ocorreu na faixa etária de 30 a 59 anos, refletindo a população economicamente ativa exposta pelo trabalho presencial obrigatório[cite: 245].

### 2. Quadro Clínico e Sobrecarga Hospitalar
* [cite_start]**O Funil de Internação:** De mais de 1.1 milhão de entrevistados, apenas 3,57% apresentaram sintomas[cite: 479]. [cite_start]Desses, cerca de 1% buscou atendimento médico, e a taxa final de internação foi de apenas **0,05%**[cite: 482, 485, 856].
* [cite_start]**Market Share Hospitalar:** A dependência do SUS é massiva no Norte e Nordeste (chegando a 82%), enquanto as regiões Sul e Sudeste absorvem grande parcela da demanda via rede privada[cite: 534, 549, 560, 561].
* [cite_start]**Subnotificação:** A taxa real de positividade (unificando os 3 tipos de teste) foi de 24,7% entre os testados, indicando um volume real de contágio muito superior ao divulgado na época[cite: 565].

### 3. Impacto Econômico
* [cite_start]**Paralisia e Endividamento:** Mais de 55% das pessoas afastadas do trabalho não receberam nenhuma remuneração, forçando a quebra do isolamento por necessidade de sobrevivência[cite: 593].
* [cite_start]**Dependência do Estado:** Norte e Nordeste mostraram altíssima dependência do Auxílio Emergencial (chegando a >60% de dependência exclusiva em alguns estados), enquanto o Sul/Sudeste utilizou mais o Seguro-Desemprego, reflexo da formalidade do trabalho[cite: 677, 774].

---

## 🏥 O Plano de Contingência (Ações para o Hospital)

Com base na inteligência extraída, formulamos o seguinte plano de ação para a diretoria hospitalar visando surtos futuros:

1. [cite_start]**Dimensionamento Baseado na Pirâmide Real:** Utilizar a taxa de conversão sintoma-internação de **0,05%** sobre a população exposta para calcular com precisão a necessidade de leitos de UTI, respiradores e equipes, abandonando estimativas teóricas[cite: 856, 858].
2. **Isolamento como Termômetro Preditivo:** Monitorar as taxas de isolamento social da região. [cite_start]A quebra do isolamento demonstrou anteceder os picos de lotação hospitalar em 2 a 3 semanas, permitindo preparação logística antecipada[cite: 863, 864].
3. [cite_start]**Mapeamento do Mix Público-Privado:** Projetar a fatia de atendimento do hospital e preparar acordos de contrarreferência baseados na dependência local do SUS (que varia drasticamente por estado)[cite: 869, 870].
4. [cite_start]**Foco em Grupos Estruturais:** Direcionar campanhas preventivas e protocolos de triagem rigorosos para trabalhadores de setores super-expostos (transporte, comércio e alimentação) e populações com vulnerabilidade sobreposta[cite: 363, 860, 861].
5. [cite_start]**Correção de Subnotificação:** Aplicar multiplicadores de projeção considerando a taxa de 24,7% de positividade em contraste com a baixa cobertura de testagem (11,6%), garantindo margem de segurança no estoque de insumos críticos[cite: 866, 867].

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

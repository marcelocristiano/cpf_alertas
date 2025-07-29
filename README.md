# 🔍 Projeto Databricks – Consulta Automatizada de Antecedentes Criminais

Este projeto foi desenvolvido para automatizar a verificação de antecedentes criminais de CPFs informados por usuários, utilizando o ecossistema do Databricks com suporte a Delta Lake, notebooks automatizados, e envio de relatórios por e-mail.

## ⚙️ Funcionamento Geral

O sistema monitora continuamente uma tabela de controle no Databricks utilizando **Structured Streaming**. Quando um novo CPF é inserido, o Databricks aciona automaticamente um notebook que realiza os seguintes passos:

- Lê o CPF e o e-mail da nova linha na tabela de controle.
- Consulta a tabela `antecedentes_criminais` para verificar se existem registros relacionados ao CPF.
- Gera um **relatório em PDF** com os dados encontrados (ou uma mensagem informando ausência de registros).
- Envia o relatório para o e-mail informado pelo solicitante.
- Atualiza o status da solicitação na tabela de controle para `processado` ou `erro`, conforme o resultado da operação.

## 🧱 Componentes do Projeto

### 🔄 Tabela de Controle: `controle_consultas`

Essa tabela deve estar no formato Delta e ser compatível com leitura contínua via Structured Streaming. Cada nova linha inserida representa uma nova solicitação de consulta.

| Coluna              | Tipo       | Descrição                                   |
|---------------------|------------|---------------------------------------------|
| `cpf`               | string     | CPF a ser verificado                        |
| `email`             | string     | E-mail do solicitante                       |
| `data_solicitacao`  | timestamp  | Momento da inserção da linha                |
| `status`            | string     | Status da solicitação: `pendente`, `processado`, `erro` |

### 📂 Tabela de Base: `antecedentes_criminais`

Contém todos os registros históricos ou ativos de antecedentes.

| Coluna             | Tipo    | Descrição                                |
|--------------------|---------|------------------------------------------|
| `cpf`              | string  | CPF do indivíduo                         |
| `tipo_ocorrencia`  | string  | Natureza da ocorrência (ex: Roubo)       |
| `descricao`        | string  | Detalhamento da infração                 |
| `data_ocorrencia`  | date    | Data do ocorrido                         |
| `local`            | string  | Local do fato                            |

## 🔁 Fluxo do Processo

O fluxo abaixo descreve a execução completa:

🟡 Nova linha na tabela controle_consultas
↓
📘 Notebook é acionado automaticamente
↓
🔍 Consulta antecedentes_criminais usando o CPF
↓
📄 Geração de PDF com os dados (ou mensagem de ausência)
↓
✉️ Envio do PDF por e-mail ao solicitante
↓
✅ Atualização da coluna status para processado ou erro


## 🧰 Tecnologias Utilizadas

- **Databricks (Notebook + Jobs)**
- **Delta Lake**
- **Structured Streaming**
- **Python (PySpark, ReportLab/FPDF, smtplib)**
- **Databricks Jobs para orquestração**
- **Armazenamento em DBFS (Databricks File System)** para PDFs temporários

## 📬 Requisitos para Envio de E-mail

Para o envio de relatórios por e-mail, recomenda-se configurar uma conta SMTP (Gmail, Outlook ou SMTP corporativo). As credenciais devem ser armazenadas em secrets no Databricks.

## 🚀 Como Executar

1. Criar as tabelas `controle_consultas` e `antecedentes_criminais` no seu catálogo Databricks.
2. Desenvolver ou importar o notebook responsável por processar as inserções.
3. Agendar o notebook via Trigger Once ou Streaming com Auto Loader.
4. Configurar secrets para as credenciais SMTP.
5. Testar inserindo uma nova linha na tabela `controle_consultas`.

---

Projeto ideal para automatizar validações sensíveis com integração de dados em tempo real no Databricks.

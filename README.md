# Projeto de Data Warehouse e Análise de Vendas para a Livraria DevSaber
## Integrantes do Mini projeto
Carol Rocha
Bruna Nascimento
Tamires Melo
Raquel Rios 
![Google BigQuery](https://img.shields.io/badge/Google_BigQuery-4285F4?style=for-the-badge&logo=google-bigquery&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-025E8C?style=for-the-badge&logo=sql&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

## 1. 🎯 Missão do Projeto

Este projeto simula um cenário de consultoria real onde a livraria "DevSaber" buscou escalar sua análise de dados. A missão foi migrar o controle de vendas de planilhas frágeis para um mini Data Warehouse robusto no Google BigQuery, construindo um pipeline de dados completo: da modelagem e criação das tabelas à ingestão e, finalmente, à extração de insights estratégicos através de consultas SQL e Views.

## 2. 📖 O Desafio: Do Caos da Planilha à Inteligência de Negócios

Toda empresa em crescimento enfrenta um ponto de inflexão onde planilhas se tornam um gargalo. A Livraria DevSaber estava nesse ponto, enfrentando desafios como:
-   **Falta de Escalabilidade:** Dificuldade para lidar com um volume crescente de vendas.
-   **Risco à Integridade dos Dados:** Erros manuais e falta de padronização.
-   **Incapacidade Analítica:** Impossibilidade de realizar análises complexas e cruzar informações de forma eficiente.

Este projeto resolve esses problemas, criando uma fundação de dados sólida para o futuro da empresa.

## 3. 🛠️ Ferramentas e Conceitos Aplicados

* **Cloud:** Google BigQuery como plataforma de Data Warehouse.
* **Linguagem:** SQL para todas as etapas do projeto (DDL, DML).
* **Conceitos de BigQuery:**
    * Tipos de dados otimizados (`STRING`, `INT64`).
    * Paradigma sem chaves (PK/FK), com relacionamentos lógicos via `JOIN`.
    * Uso de `CREATE OR REPLACE` para scripts idempotentes.
* **Pipeline de Dados (ETL):**
    * **Extract:** Dados extraídos de uma "planilha" conceitual.
    * **Transform:** Modelagem dos dados em um schema relacional (Clientes, Produtos, Vendas).
    * **Load:** Ingestão dos dados com `INSERT INTO`.
* **Análise de Dados:** `SELECT`, `JOIN` para cruzar informações e `SUM/GROUP BY` para agregação.
* **Automação e Reuso:** Criação de `VIEW` para simplificar o acesso a consultas complexas.

## 4. 🚀 O Pipeline de Dados: Passo a Passo da Execução

O projeto foi estruturado em um pipeline claro e sequencial. Todos os scripts estão organizados na pasta `/sql/`.

### Etapa 1: Construindo a Fundação (DDL - `CREATE TABLE`)

O primeiro passo foi modelar e criar as tabelas que serviriam como a "casa" dos dados. Foram criadas 3 tabelas normalizadas: `Clientes`, `Produtos` e `Vendas`. A lógica de não usar chaves primárias/estrangeiras segue o paradigma de performance do BigQuery.

* [Ver script de criação da tabela `Clientes`](sql/1_criacao_tabelas/01_create_clientes.sql)
* [Ver script de criação da tabela `Produtos`](sql/1_criacao_tabelas/02_create_produtos.sql)
* [Ver script de criação da tabela `Vendas`](sql/1_criacao_tabelas/03_create_vendas.sql)

### Etapa 2: Povoando o Data Warehouse (DML - `INSERT INTO`)

Com a estrutura de tabelas pronta, a próxima fase foi a ingestão dos dados. Para este projeto, o comando `INSERT INTO` foi utilizado para carregar os registros iniciais de forma controlada e didática. Em um cenário de produção com maior volume, essa etapa seria automatizada através de um pipeline de dados mais robusto (utilizando ferramentas como Cloud Functions, Dataflow, etc.).

Os dados foram inseridos seguindo a ordem de dependência para garantir a integridade lógica:
1.  Primeiro, os `Clientes` e `Produtos`.
2.  Por último, as `Vendas`, que conectam as duas tabelas anteriores.

* [Ver exemplos de scripts de ingestão de dados](/sql/2_ingestao_dados/)

### Etapa 3: Extraindo Insights (Análise com `SELECT`)

Esta é a etapa onde o valor é gerado. A consulta abaixo unifica os dados das três tabelas para criar um relatório detalhado de vendas, respondendo à principal pergunta de negócio.

**Pergunta: Listar todas as vendas com detalhes do cliente e do produto.**
```sql
SELECT
    C.Nome_Cliente,
    P.Nome_Produto,
    V.Data_Venda
FROM `seu-projeto.LivrariaDevSaber_t3_15.Vendas` AS V
JOIN `seu-projeto.LivrariaDevSaber_t3_15.Clientes` AS C ON V.ID_Cliente = C.ID_Cliente
JOIN `seu-projeto.LivrariaDevSaber_t3_15.Produtos` AS P ON V.ID_Produto = P.ID_Produto
ORDER BY V.Data_Venda;
```

### Etapa 4: Automatizando Análises com `CREATE VIEW`

Uma vez que a principal consulta analítica foi definida, o passo final foi criar uma camada de abstração para simplificar o acesso futuro a esses dados. Reescrever `JOIN`s complexos diariamente é ineficiente e aumenta a chance de erros.

A solução foi encapsular toda a lógica em uma **`VIEW`**. Uma `VIEW` funciona como uma "tabela virtual" que armazena uma consulta, permitindo que qualquer pessoa no time possa acessar os dados já processados de forma simples e segura.


**Código de Criação da View `v_relatorio_vendas_detalhado`:**

```sql
CREATE OR REPLACE VIEW `seu-projeto.LivrariaDevSaber_t3_15.v_relatorio_vendas_detalhado` AS
SELECT
    V.ID_Venda,
    V.Data_Venda,
    C.Nome_Cliente,
    P.Nome_Produto,
    V.Quantidade,
    (V.Quantidade * P.Preco_Produto) AS Valor_Total
FROM `seu-projeto.LivrariaDevSaber_t3_15.Vendas` AS V
JOIN `seu-projeto.LivrariaDevSaber_t3_15.Clientes` AS C ON V.ID_Cliente = C.ID_Cliente
JOIN `seu-projeto.LivrariaDevSaber_t3_15.Produtos` AS P ON V.ID_Produto = P.ID_Produto;
```

Benefício na Prática
Com a VIEW criada, a equipe da "Livraria DevSaber" não precisa mais se preocupar com a complexidade dos JOINs. Para ver as vendas de um cliente específico, a consulta se torna trivial:

```sql
SELECT *
FROM `seu-projeto.LivrariaDevSaber_t3_15.v_relatorio_vendas_detalhado`
WHERE Nome_Cliente = 'Ana Silva';
```
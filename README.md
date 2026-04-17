# Telco Customer Churn Analysis: Engenharia de Dados, UI/UX e Estratégia de Retenção

## 1. O Desafio de Negócio (O Problema)
No setor de Telecomunicações, o Custo de Aquisição de Clientes (CAC) é extremamente alto. Portanto, reter o cliente pelo maior tempo possível (aumentando o LTV - *Lifetime Value*) é o principal motor de lucratividade. 

Este projeto End-to-End visa resolver um problema clássico e custoso: o **Churn (Evasão de Clientes)**. Atuando como uma ponte entre a engenharia de dados e a Diretoria Comercial, o objetivo foi construir um pipeline completo — desde a modelagem no banco de dados até o design de interface (UI/UX) — para identificar onde a empresa está perdendo dinheiro e por quê.

### Perguntas de Negócio respondidas:
1. Qual é o impacto financeiro (Receita Perdida) exato gerado pela evasão atual?
2. Quais perfis demográficos apresentam maior propensão ao cancelamento?
3. O modelo de contrato atual está protegendo a receita ou facilitando a saída do cliente?
4. Existe algum gargalo técnico ou serviço específico "expulsando" a base de usuários?
5. Estratégias de *Cross-sell* são eficazes para reter o cliente no ecossistema?

---

## 2. Arquitetura e Stack Tecnológico
Para garantir governança, performance e uma experiência visual de alto nível, a solução integrou as seguintes ferramentas:

* **SGBD (Banco de Dados):** PostgreSQL.
* **Linguagem de Consulta:** SQL (DDL e DQL).
* **UI/UX Design e Prototipagem:** Figma.
* **Modelagem Analítica:** DAX (Data Analysis Expressions).
* **Visualização:** Power BI Desktop.

---

## 3. UI/UX Design e Prototipagem (Figma)
Um dashboard só gera valor se for intuitivo. Antes da implementação, toda a interface visual foi planejada no **Figma**, aplicando conceitos de redução de carga cognitiva.

* **Wireframing:** Utilização de sistema de Grids para garantir alinhamento e fluidez de leitura (padrão Z). 
* **Custom Backgrounds:** Criação de fundos personalizados em Dark Mode para otimizar a renderização no Power BI.
* **Componentização:** Cartões desenhados com Drop Shadow e bordas arredondadas (10px) para criar profundidade.
* **Psicologia das Cores:** * **Vermelho:** Ofensores (Evasão alta, contratos mensais).
  * **Azul Escuro:** Cenários saudáveis e retenção.

---

## 4. Engenharia de Dados e Modelagem (PostgreSQL)
A base bruta original consistia em uma única tabela desnormalizada (*Flat File*). Para garantir performance, projetei um **Star Schema (Esquema Estrela)** diretamente no PostgreSQL.

### Script de Estruturação (DDL)
```sql
-- 1. Tabela Dimensão Cliente
CREATE TABLE dim_cliente AS
SELECT DISTINCT
    customerID, 
    gender AS genero, 
    SeniorCitizen AS idoso, 
    Partner AS temparceiro, 
    Dependents AS temdependentes
FROM telco_raw_data;

ALTER TABLE dim_cliente ADD CONSTRAINT pk_cliente PRIMARY KEY (customerID);
```

-- 2. Tabela Dimensão Contrato
```
CREATE TABLE dim_contrato AS
SELECT DISTINCT
    customerID, 
    Contract AS tipocontrato, 
    PaperlessBilling AS faturamentodigital, 
    PaymentMethod AS metodopagamento
FROM telco_raw_data;

ALTER TABLE dim_contrato ADD CONSTRAINT pk_contrato PRIMARY KEY (customerID);
```

-- 3. Tabela Dimensão Serviços
```
CREATE TABLE dim_servicos AS
SELECT DISTINCT
    customerID, 
    PhoneService, 
    MultipleLines, 
    InternetService, 
    OnlineSecurity, 
    OnlineBackup, 
    DeviceProtection, 
    TechSupport, 
    StreamingTV, 
    StreamingMovies
FROM telco_raw_data;

ALTER TABLE dim_servicos ADD CONSTRAINT pk_servicos PRIMARY KEY (customerID);
```

-- 4. Tabela Fato Churn
```
CREATE TABLE fato_churn AS
SELECT 
    customerID, 
    tenure AS mesespermanencia, 
    MonthlyCharges AS valormensal, 
    TotalCharges AS valortotal, 
    Churn AS statuschurn
FROM telco_raw_data;
```
## 5. Desenvolvimento Analítico (DAX)
Desenvolvi métricas avançadas para traduzir o banco de dados em KPIs comerciais e entender a sensibilidade ao preço e o engajamento.

Colunas Calculadas (Clusterização)
```
Faixa de Mensalidade = 
SWITCH(
    TRUE(),
    'fato_churn'[valormensal] <= 30, "1. Básico (Até $30)",
    'fato_churn'[valormensal] <= 60, "2. Intermediário ($31 a $60)",
    'fato_churn'[valormensal] <= 90, "3. Avançado ($61 a $90)",
    "4. Premium (Acima de $90)"
)
```

Principais Medidas de Negócio
```
Total de Clientes = COUNTROWS('fato_churn')

Taxa de Churn = DIVIDE(
    CALCULATE([Total de Clientes], 'fato_churn'[statuschurn] = "Yes"), 
    [Total de Clientes], 0
)

Receita Perdida = CALCULATE(SUM('fato_churn'[valormensal]), 'fato_churn'[statuschurn] = "Yes")
```
## 6. Storytelling de Dados e Dashboards
O fluxo analítico foi estruturado em 3 níveis lógicos, guiando o gestor da visão macro para a causa raiz.

Página Inicial: Navegação

Página 1: Resumo Executivo
Impacto global na companhia.

Página 2: Perfil do Cliente
Investigação comportamental por faixa etária e estrutura familiar e árvore de decomposição com IA para identificar caminhos de cancelamento.

Página 3: Serviços e Contratos
Análise técnica de falhas de produto (Fibra Óptica) e impacto do Cross-sell na retenção.

## 7. Conclusão e Plano de Ação
As recomendações estratégicas baseadas nos dados são:

Reestruturação Comercial: Migrar clientes do contrato "Mensal" para planos de fidelidade de 1 a 2 anos.

Estratégia de Cross-sell: Focar na venda de Suporte Técnico e Segurança Online, pois clientes com +4 serviços apresentam evasão irrisória (Efeito Lock-in).

Auditoria Técnica: Investigar a qualidade da infraestrutura de Fibra Óptica, que apresenta taxas de churn acima da média.

## Contato

| 👨‍💻 Autor | **Arthur Mesquita** |
| :--- | :--- |
| **Cargo** | Analista de Dados & BI |
| **LinkedIn** | [linkedin.com/in/arthur-g-mesquita](https://www.linkedin.com/in/arthur-g-mesquita) |
| **Localização** | 📍 Recife, PE |

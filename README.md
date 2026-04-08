## Olist E-commerce Analytics: Engenharia de Dados e Business Intelligence

[Baixe o arquivo do Power BI (.pbix) completo clicando aqui em "Releases"](https://github.com/guiwilliams/olist-ecommerce-analytics/releases/tag/v1.0)

![Capa do Dashboard](https://github.com/guiwilliams/olist-ecommerce-analytics/blob/main/Visão%20Executiva.png)

##  O Problema de Negócio
A Olist é uma grande loja de departamentos atuando em marketplaces. O desafio deste projeto foi construir um produto de dados de ponta a ponta: desde a ingestão de dados brutos espalhados em arquivos CSV até a entrega de um **Painel Executivo, Comercial e Logístico**.

O objetivo principal foi auditar a saúde financeira da empresa, ranquear a força de vendas e, acima de tudo, **encontrar gargalos na malha logística**, analisando o tempo de entrega e os custos de frete em um país de dimensões continentais como o Brasil.

##  Stack Tecnológico
* **PostgreSQL:** Ingestão massiva de dados, tipagem, chaves primárias e modelagem.
* **SQL:** Limpeza, cruzamento de dados (`JOINs`) e criação de `Views` otimizadas.
* **Power BI:** Modelagem de dados, cálculos avançados em DAX e interatividade (Bookmarks/Filtros).
* **Figma:** Prototipação de UI/UX, criação de grids simétricos, backgrounds em Dark Mode e paleta de cores.

---

## 1. Engenharia de Dados (ETL no PostgreSQL)
Para garantir alta performance e não sobrecarregar o Power BI com transformações pesadas, toda a estruturação foi feita no banco de dados. Criei os schemas, defini a tipagem correta e fiz a ingestão dos dados via comando `COPY`.

<details>
<summary><b>Clique aqui para ver o código SQL de criação das tabelas e ingestão (DDL e DML)</b></summary>

```sql
-- CRIANDO A TABELA DE CLIENTES
CREATE TABLE "Projeto Olist Database".customers(
	customer_id VARCHAR PRIMARY KEY,
	customer_unique_id VARCHAR,
	customer_zip_code VARCHAR,
	customer_city VARCHAR,
	customer_state VARCHAR
);

COPY "Projeto Olist Database".customers(customer_id, customer_unique_id, customer_zip_code, customer_city, customer_state)
FROM 'C:\archive\olist_customers_dataset.csv' DELIMITER ',' CSV HEADER;

-- CRIANDO TABELA DE VENDEDORES
CREATE TABLE "Projeto Olist Database".sellers (
	seller_id VARCHAR PRIMARY KEY,
	seller_zip_code VARCHAR NOT NULL,
	seller_city VARCHAR NOT NULL,
	seller_state VARCHAR NOT NULL
);

COPY "Projeto Olist Database".sellers(seller_id, seller_zip_code, seller_city, seller_state)
FROM 'C:\archive\olist_sellers_dataset.csv' DELIMITER ',' CSV HEADER;

-- CRIANDO TABELA DE ORDENS
CREATE TABLE "Projeto Olist Database".orders(
	order_id VARCHAR PRIMARY KEY,
	customer_id VARCHAR NOT NULL,
	order_status VARCHAR NOT NULL,
	order_purchase_timestamp TIMESTAMP,
	order_approved_at TIMESTAMP,
	order_delivered_carrier_date TIMESTAMP,
	order_delivered_customer_date TIMESTAMP,
	order_estimated_delivery_date DATE
);

COPY "Projeto Olist Database".orders(order_id, customer_id, order_status, order_purchase_timestamp, order_approved_at, order_delivered_carrier_date, order_delivered_customer_date, order_estimated_delivery_date)
FROM 'C:\archive\olist_orders_dataset.csv' DELIMITER ',' CSV HEADER;

-- CRIANDO TABELA DE ITENS DAS ORDENS
CREATE TABLE "Projeto Olist Database".order_item(
	order_id VARCHAR, -- Foreign Key
	order_item_id INT,
	product_id VARCHAR NOT NULL,
	seller_id VARCHAR NOT NULL,
	shipping_limit_date TIMESTAMP,
	price FLOAT,
	freight_value FLOAT
);
	
COPY "Projeto Olist Database".order_item(order_id, order_item_id, product_id, seller_id, shipping_limit_date, price, freight_value)
FROM 'C:\archive\olist_order_items_dataset.csv' DELIMITER ',' CSV HEADER;
</details>

Após a estruturação física do banco, criei uma View focada em cruzar os dados de Vendas e Logística, entregando ao Power BI apenas as colunas essenciais para análise:

SQL
-- View Consolidada: O Motor do Power BI
CREATE OR REPLACE VIEW vendas_logisticas AS
SELECT
	o.order_id, o.customer_id, 
	oi.product_id, oi.seller_id, 
	oi.price, oi.freight_value, 
	o.order_purchase_timestamp,
	o.order_approved_at, o.order_delivered_carrier_date, 
	o.order_delivered_customer_date,
	o.order_estimated_delivery_date
FROM "Projeto Olist Database".orders as o
INNER JOIN "Projeto Olist Database".order_item as oi on o.order_id = oi.order_id
WHERE o.order_status = 'delivered';
```

</details>

## 2. Modelagem e DAX (Regras de Negócio)
A regra de negócio precisava espelhar a realidade de uma grande operação. Utilizei funções de transição de contexto (CALCULATE, FILTER) para isolar anomalias.

Higienização de SLA: Para calcular o SLA Médio de Postagem de forma justa, criei uma medida que exclui vendedores com volume irrelevante (menos de 20 vendas), evitando que outliers distorçam o indicador geral da operação.

Taxa de OTIF (On Time In Full): Criação de cálculo de percentual de entregas realizadas dentro da data estimada.

## 3. Telas e Insights do Projeto
* **Visão Executiva e Comercial:**
Foco em densidade de informação limpa. Utilização de gráficos com eixos compartilhados (Dual Label) para cruzar Faturamento e Volume na mesma visualização, além da aplicação de Treemaps com formatação condicional para reduzir a carga cognitiva na análise de Top Vendedores.

![Visão Comercial](https://github.com/guiwilliams/olist-ecommerce-analytics/blob/main/Visão%20Comercial.png)

## 3.1 Visão Logística (O Gargalo Continental):
Esta página foi desenhada para a Diretoria de Operações. Os dados escancaram as dores do SLA e da malha de transportes brasileira:

* Assimetria de Prazos: Identificação clara da lentidão estrutural no Norte e Nordeste, com estados como Roraima e Amazonas ultrapassando a marca de 25 dias de Lead Time.

* Custo de Frete: Correlação direta entre a distância do polo Sudeste e a explosão no custo médio de frete, impactando a conversão de vendas nessas praças.


![Visão Logistica](https://github.com/guiwilliams/olist-ecommerce-analytics/blob/main/Visão%20Logistica.png)

**Projeto concebido do zero com foco na resolução de problemas reais de negócio através da análise de dados.**

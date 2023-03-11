# Sales Analysis for the Top 3 Sellers from Olist, a Brazilian E-commerce platform

*View the [Power BI Dashboard](https://app.powerbi.com/reportEmbed?reportId=1fa7130c-72bb-4038-80bf-452475d41ac6&autoAuth=true&ctid=820acc96-c07b-4b97-a193-90181f5af284)*

*View the [Flat PDF version](https://drive.google.com/file/d/160HimaUsWW6I90kvHY2Vc8AEgzadcveA/view?usp=sharing)*

## I. Introduction
Olist is an e-commerce platform (similar to Shopee) that connects small businesses across Brazil's marketplaces. Merchants are able to sell their products through the Olist Store app and ship them directly to the customers using Olist's logistics partners. With this [dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), my goal is to provide an overall descriptive analysis of Olist's customer base while drilling down on the sales performance of the top performing sellers. 

## II. What we are Given

The [dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) was generally provided by Olist. It has information of 100k orders from 2016 to 2018 made at multiple marketplaces in Brazil. It is real and anonymised commercial data. 

These are the files from the dataset: 
1. **Customers**- a list of Olist's customers with an individually provided customer ID along with location details: Zip Code, City, and State
2. **Geolocation** - information about the latitude and longitude per Zip code prefix. 
3. **Order Items** - a list of every order done on the app. Each order has an order id coupled with product IDs, seller IDs, shipping date and price of product
4. **Payment Types** - description of payment type done per order (credit card, cash or via voucher)
5. **Order Reviews**- customer reviews of the orders they made. Review scores range from 1 to 5. 
6. **Order status**- status of every order made on the app (approved, invoiced, delivered, etc.) along with timestamps for each.
7. **Products** - product IDs and their categories. Product dimensions are also included
8. **Sellers**- Sellers and their location.

Out of these 8 files, three of them were not explored further: 
1. Geolocation - External data was used to validate the Zip Code's states
2. Payments - Orders used multiple payment types and it is not necessary in evaluating the sales performance for the three sellers. The amount for every order was already included in the order items list so price was based on that file. 
3. Order reviews - This could have been explored further but not all orders have reviews.

### Limitations

Here are some of the limitations that were discovered when analyzing the files : 
1. Incomplete details for financial analysis such as Cost of Goods Sold (COGS) and Tax - Profit, Gross Margin Revenue, and Gross income would have been analyzed as well if COGS wasn't missing. What is left is just an analysis of total revenue made per product by every seller
2. Inventory details were also not included. Analysis could also be done on inventory turnover, out of stock rate and carrying cost of inventory 

## III. The Ask

From the dataset, here are the things that can be explored: 

1. Overall
- Number of products per category
- Customer count per Brazilian state
- Top Sellers
- Delivered vs Canceled orders 

2. Per Seller
- New and Returning customers
- Completed orders
- Annual sales growth rate
- Sales per Month
- Sales per Category
- Sales per State
- Product Performance (Units sold, Total Sales, and Rank)

## IV. Data Import

``` sql
CREATE TABLE olist_customers_dataset (
customer_id VARCHAR(50),
customer_unique_id VARCHAR(50),
customer_zip_code_prefix MEDIUMINT UNSIGNED,
customer_city VARCHAR(50),
customer_state CHAR(2),
CONSTRAINT pk_id PRIMARY KEY (customer_id)
);

LOAD DATA LOCAL INFILE 'C:/Program Files/MySQL/MySQL Server 8.0/Olist/olist_customers_dataset.csv' INTO TABLE olist_customers_dataset
FIELDS TERMINATED BY ','
ENCLOSED by '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

CREATE TABLE order_items_dataset (
order_id VARCHAR(50),
order_item DOUBLE PRECISION,
product_id VARCHAR(50),
seller_id VARCHAR(50),
shipping_limit_date DATETIME,
price DOUBLE PRECISION,
freight_value DOUBLE PRECISION
);

LOAD DATA LOCAL INFILE 'C:/Program Files/MySQL/MySQL Server 8.0/Olist/olist_order_items_dataset.csv' INTO TABLE order_items_dataset
FIELDS TERMINATED BY ','
ENCLOSED by '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

CREATE TABLE orders_time_dataset (
order_id VARCHAR(50) ,
customer_id VARCHAR(50),
order_status VARCHAR(50),
order_purchase_timestamp VARCHAR(50),
order_approved_at VARCHAR(50),
order_delivered_carrier_date VARCHAR(50),
order_delivered_customer_date VARCHAR(50),
order_estimated_delivery_date VARCHAR(50)
);

LOAD DATA LOCAL INFILE 'C:/Program Files/MySQL/MySQL Server 8.0/Olist/olist_orders_dataset.csv' INTO TABLE orders_time_dataset
FIELDS TERMINATED BY ','
ENCLOSED by '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

CREATE TABLE product_dataset (
product_id VARCHAR(50) ,
product_category_name VARCHAR(50),
product_name_lenght TINYINT UNSIGNED,
product_description_lenght SMALLINT UNSIGNED ,
product_photos_qty TINYINT UNSIGNED ,
product_weight_g SMALLINT UNSIGNED ,
product_length_cm TINYINT UNSIGNED ,
product_height_cm TINYINT UNSIGNED ,
product_width_cm TINYINT UNSIGNED 
);

LOAD DATA LOCAL INFILE 'C:/Program Files/MySQL/MySQL Server 8.0/Olist/olist_products_dataset.csv' INTO TABLE product_dataset
FIELDS TERMINATED BY ','
ENCLOSED by '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(product_id,product_category_name,@product_name_lenght,@product_description_lenght,@product_photos_qty,
@product_weight_g,@product_length_cm,@product_height_cm,@product_width_cm)
SET
product_name_lenght = NULLIF(@product_name_lenght,''),
product_description_lenght = NULLIF(@product_description_lenght,''),
product_photos_qty = NULLIF(@product_photos_qty,''),
product_weight_g = NULLIF(@product_weight_g,''),
product_length_cm = NULLIF(@product_length_cm,''),
product_height_cm = NULLIF(@product_height_cm,''),
product_width_cm = NULLIF(@product_width_cm,'');

CREATE TABLE sellers_dataset (
seller_id VARCHAR(50) ,
seller_zip_code_prefix MEDIUMINT UNSIGNED,
seller_city VARCHAR(50),
seller_state CHAR(2)
);

LOAD DATA LOCAL INFILE 'C:/Program Files/MySQL/MySQL Server 8.0/Olist/olist_sellers_dataset.csv' INTO TABLE sellers_dataset
FIELDS TERMINATED BY ','
ENCLOSED by '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

``` sql
-- additional data to check brazilian states
CREATE TABLE brazil_states (
state_abbrev CHAR(2) ,
state VARCHAR(50),
postcode_range VARCHAR(50)
);

INSERT INTO brazil_states
  (state_abbrev,state,postcode_range)
VALUES
	('AC','Acre','69900-000 to 69990-000'),
  ('AL','Alagoas','57000-000 to 57995-000'),
  ('AM','Amazonas','69000-000 to 69280-000'),
  ('AM','Amazonas','69400-000 to 69895-000'),
  ('AP','Amapá','68900-000 to 68997-000'),
  ('BA','Bahia','40000-000 to 48990-000'),
  ('CE','Ceará','60000-000 to 63970-000'),
  ('DF','Distrito Federal','70000-000 to 73300-000'),
  ('ES','Espírito Santo','29000-000 to 29980-000'),
  ('GO','Goiás','72800-000 to 76740-000'),
  ('MA','Maranhão','65000-000 to 65995-000'),
  ('MG','Minas Gerais','30000-000 to 39998-000'),
  ('MS','Mato Grosso do Sul','79000-000 to 79995-000'),
  ('MT','Mato Grosso','78000-000 to 78890-000'),
  ('PA','Pará','66000-000 to 68890-000'),
  ('PB','Paraíba','58000-000 to 58995-000'),
  ('PE','Pernambuco','50000-000 to 56980-000'),
  ('PI','Piauí','64000-000 to 64995-000'),
  ('PR','Paraná','80000-000 to 87990-000'),
  ('RJ','Rio de Janeiro','20000-000 to 28990-000'),
  ('RN','Rio Grande do Norte','59000-000 to 59995-000'),
  ('RO','Rondônia','78900-000 to 78999-000'),
  ('RR','Roraima','69300-000 to 69390-000'),
  ('RS','Rio Grande do Sul','90000-000 to 99990-000'),
  ('SC','Santa Catarina','88000-000 to 89998-000'),
  ('SE','Sergipe','49000-000 to 49995-000'),
  ('SP','São Paulo','01000-000 to 19990-000'),
  ('TO','Tocantins','77000-000 to 77995-000');

CREATE TABLE brazil_states_abbrev (
state_abbrev CHAR(2) ,
state VARCHAR(50));
INSERT INTO brazil_states_abbrev (state_abbrev, state)
VALUES
  ('AC','Acre'),
  ('AL','Alagoas'),
  ('AM','Amazonas'),
  ('AM','Amazonas'),
  ('AP','Amapá'),
  ('BA','Bahia'),
  ('CE','Ceará'),
  ('DF','Distrito Federal'),
  ('ES','Espírito Santo'),
  ('GO','Goiás'),
  ('MA','Maranhão'),
  ('MG','Minas Gerais'),
  ('MS','Mato Grosso do Sul'),
  ('MT','Mato Grosso'),
  ('PA','Pará'),
  ('PB','Paraíba'),
  ('PE','Pernambuco'),
  ('PI','Piauí'),
  ('PR','Paraná'),
  ('RJ','Rio de Janeiro'),
  ('RN','Rio Grande do Norte'),
  ('RO','Rondônia'),
  ('RR','Roraima'),
  ('RS','Rio Grande do Sul'),
  ('SC','Santa Catarina'),
  ('SE','Sergipe'),
  ('SP','São Paulo'),
  ('TO','Tocantins');
  
CREATE TABLE brazil_states_abbrev2 AS SELECT DISTINCT state_abbrev, state
FROM brazil_states_abbrev;
DROP TABLE brazil_states_abbrev;
ALTER TABLE brazil_states_abbrev2 RENAME brazil_states_abbrev;
```
## V. Data Cleaning

### A. Customers Dataset

1. Checking for Duplicates

``` sql
SELECT DISTINCT customer_id FROM olist_customers_dataset;
SELECT DISTINCT customer_unique_id FROM olist_customers_dataset;

-- there are duplicates from customer_unique_id so column should be dropped

-- remove column from table: 
ALTER TABLE olist_customers_dataset DROP COLUMN customer_unique_id;
-- renaming table name : 
ALTER TABLE olist_customers_dataset RENAME customers_dataset;

-- check data types of customers dataset : everything seems correct for data wrangling
```

2. Check for validity of Brazilian states

``` sql
CREATE TABLE brazil_states_cln AS 
SELECT x.state_abbrev,x.state,x.postcode_range,CAST(x.zipcode_max AS UNSIGNED) AS zipcode_max, CAST(x.zipcode_min AS UNSIGNED) AS zipcode_min
FROM (
SELECT state_abbrev, state, postcode_range, 
CASE WHEN state_abbrev = 'SP' THEN substr(postcode_range,2,4)
ELSE substr(postcode_range,1,5)
END AS zipcode_min, 
substr(postcode_range,-9,5) AS zipcode_max
FROM brazil_states) x ;

DROP TABLE brazil_states;
ALTER TABLE brazil_states_cln RENAME brazil_states;

-- checking if zip codes from customer data is consistent wtih state codes from brazil states table
SELECT cd.customer_id, cd.customer_zip_code_prefix, cd.customer_state, bs.postcode_range, bs.zipcode_min , bs.zipcode_max , bs.state_abbrev
FROM customers_dataset cd CROSS JOIN brazil_states bs
WHERE cd.customer_zip_code_prefix BETWEEN bs.zipcode_min AND bs.zipcode_max
AND cd.customer_state <> bs.state_abbrev;

-- 311 rows did not have matching zipcodes and state names. 
-- Reason: Zip code ranges for Goias and Distrito Federal are overlapping
-- solution: remove Goias and Distrito Federal states from brazil_states table and create new table for just Goias and Distrito Federal

DELETE FROM brazil_states WHERE state_abbrev = 'DF' OR state_abbrev = 'GO';

CREATE TABLE DF_GO ( 
state_abbrev CHAR(2),
state_city VARCHAR(50)
);

INSERT INTO DF_GO (state_abbrev,state_city) 
VALUES  ('DF','70000-000 - Brasilia'),('DF','70640-000 - Cruzeiro'),('DF','71000-000 - Guara'),('DF','71500-000 - Lago Norte'),('DF','71570-000 - Paranoa'),
('DF','71600-000 - Lago Sul'),('DF','71690-000 - Sao Sebastiao'),('DF','71700-000 - Nucleo Bandeirante'),('DF','71725-000 - Candangolandia'),('DF','71800-000 - Riacho Fundo'),('DF','72000-000 - Taguatinga'),('DF','72200-000 - Ceilandia'),('DF','72300-000 - Samambaia'),('DF','72400-000 - Gama'),('DF','72500-000 - Santa Maria'),('DF','72600-000 - Recanto Das Emas'),('DF','72700-000 - Brazlandia'),('DF','73000-000 - Sobradinho'),('DF','73300-000 - Planaltina'),('GO','72800-000 - Luziania'),('GO','72860-000 - Novo Gama'),('GO','72870-000 - Valparaiso De Goias'),('GO','72880-000 - Cidade Ocidental'),('GO','72900-000 - Santo Antonio Do Descoberto'),('GO','72910-000 - Aguas Lindas De Goias'),('GO','72920-000 - Alexania'),('GO','72940-000 - Abadiania'),('GO','72960-000 - Corumba De Goias'),
('GO','72975-000 - Cocalzinho De Goias'),('GO','72980-000 - Pirenopolis'),('GO','72990-000 - Vila Propicio'),('GO','73700-000 - Padre Bernardo'),('GO','73730-000 - Mimoso De Goias'),('GO','73740-000 - Colinas Do Sul'),('GO','73750-000 - Planaltina'),('GO','73760-000 - Sao Joao D''alianca'),('GO','73770-000 - Alto Paraiso De Goias'),('GO','73780-000 - Agua Fria De Goias'),('GO','73790-000 - Cavalcante'),('GO','73795-000 - Teresina De Goias'),('GO','73800-000 - Formosa'),('GO','73820-000 - Nova Roma'),('GO','73825-000 - Vila Boa'),('GO','73830-000 - Monte Alegre De Goias'),('GO','73840-000 - Campos Belos'),
('GO','73850-000 - Cristalina'),('GO','73860-000 - Sao Domingos'),('GO','73865-000 - Divinopolis De Goias'),('GO','73870-000 - Cabeceiras'),('GO','73890-000 - Flores De Goias'),('GO','73900-000 - Posse'),('GO','73910-000 - Guarani De Goias'),('GO','73920-000 - Iaciara'),('GO','73930-000 - Simolandia'),
('GO','73950-000 - Alvorada Do Norte'),('GO','73970-000 - Mambai'),('GO','73975-000 - Buritinopolis'),('GO','73980-000 - Damianopolis'),('GO','73990-000 - Sitio D''abadia'),('GO','74000-000 - Goiania'),('GO','74900-000 - Aparecida De Goiania'),('GO','75000-000 - Anapolis'),('GO','75165-000 - Ouro Verde De Goias'),
('GO','75170-000 - Goianapolis'),('GO','75175-000 - Terezopolis De Goias'),('GO','75180-000 - Silvania'),('GO','75185-000 - Sao Miguel Do Passa Quatro'),
('GO','75190-000 - Leopoldo De Bulhoes'),('GO','75195-000 - Bonfinopolis'),('GO','75200-000 - Pires Do Rio'),('GO','75210-000 - Palmelo'),('GO','75220-000 - Santa Cruz De Goias'),('GO','75230-000 - Cristianopolis'),('GO','75240-000 - Bela Vista De Goias'),('GO','75245-000 - Caldazinha'),
('GO','75250-000 - Senador Canedo'),('GO','75260-000 - Vianopolis'),('GO','75280-000 - Orizona'),('GO','75340-000 - Hidrolandia'),
('GO','75345-000 - Abadia De Goias');

-- create cleaned table for df_go
CREATE TABLE df_go_cln AS 
SELECT state_abbrev, state_city, substr(state_city,1,5) as state_code, substr(state_city,13) as city
FROM df_go;

-- match customers from DF or GO with df_go_cln and check for mismatch
CREATE VIEW x AS SELECT cd.customer_id, cd.customer_zip_code_prefix, cd.customer_city, cd.customer_state , dg.state_abbrev, dg.city
FROM customers_dataset cd LEFT JOIN df_go_cln dg ON cd.customer_zip_code_prefix=dg.state_code
WHERE cd.customer_state IN ('DF','GO') AND dg.state_abbrev IS NULL;

SELECT x.customer_id, x.customer_zip_code_prefix, x.customer_city, x.customer_state, dg.city, dg.state_abbrev
FROM x LEFT JOIN df_go_cln dg ON x.customer_city = lower(dg.city)
WHERE x.customer_state <> dg.state_abbrev;

-- it seems that the initially created table for df_go didnt capture all zip codes. 
-- the customer_city column in customers_dataset however, matched the cities from df_go with only 10 rows remaining;
-- remaining Zip code prefix with no match: 73751, 73752, 73753, 73754, 73755. Seems like both GO and DF has a city called Planaltina
-- upon checking, these prefixes belong to GO

-- conclusion: customer_zip_code column from customers dataset matches each state
```

### B. Products Dataset

1. Check for duplicate Product IDs

``` sql
SELECT DISTINCT product_id
FROM product_dataset;
--- 32951 row(s) returned
-- number of rows equal to distinct product_ids. : no duplicates
```

2. Check for Nulls
``` sql
SELECT product_category_name, COUNT(product_id) as cnt
FROM product_dataset
GROUP BY product_category_name;
-- nulls were not detected but a blank field has a count of 610 total products
-- solution: change this field to 'NULL'

UPDATE product_dataset
SET product_category_name = CASE WHEN product_category_name = '' THEN NULL 
ELSE product_category_name END;
-- to check: 
SELECT product_category_name, COUNT(product_id) as cnt FROM product_dataset GROUP BY product_category_name;
```
![image](https://user-images.githubusercontent.com/118359623/224473106-0190bdf6-9adc-4b09-a6fb-ba42cb8be66c.png)

``` sql
-- join products_dataset with category_name_translation to see categories in English
SELECT DISTINCT pd.product_category_name, pc.product_category_name_english
FROM product_dataset pd LEFT JOIN product_category_name_translation pc ON pd.product_category_name = pc.product_category_name;

-- 'portateis_cozinha_e_preparadores_de_alimentos' and 'pc_gamer' has no translation. 
-- In English it is kitchen_portables_and_food_preparers and pc game, respectively,  according to google translate
-- it would be best to replace category names with generic names to shorten our list of categories 
```
3. Reclassify Categories

``` sql

CREATE VIEW ctgry AS SELECT product_category_name,product_category_name_english,
CASE WHEN product_category_name IN ('pc_gamer','pcs','informatica_acessorios') THEN 'Computer and computer accessories'
WHEN product_category_name = 'agro_industria_e_comercio' THEN 'Agriculture'
WHEN product_category_name = 'consoles_games' THEN 'Gaming'
WHEN product_category_name IN ('construcao_ferramentas_construcao','construcao_ferramentas_iluminacao','construcao_ferramentas_seguranca','construcao_ferramentas_jardim',
'construcao_ferramentas_ferramentas','automotivo','ferramentas_jardim') THEN 'Construction tools and General house tools'
WHEN product_category_name = 'cool_stuff' THEN 'Cool Stuff'
WHEN product_category_name IN ('artes','artes_e_artesanato') THEN 'Art'
WHEN product_category_name IN ('cds_dvds_musicais','musica','instrumentos_musicais') THEN 'Music'
WHEN product_category_name IN ('audio','eletronicos','telefonia_fixa','telefonia') THEN 'Electronics'
WHEN product_category_name IN ('fashion_roupa_feminina','fashion_bolsas_e_acessorios','fashion_roupa_infanto_juvenil','fashion_roupa_masculina',
'fashion_calcados','fashion_esporte','fashion_underwear_e_moda_praia','malas_acessorios','perfumaria','relogios_presentes') THEN 'Fashion and Accessories'
WHEN product_category_name = 'pet_shop' THEN 'Pet Shop'
WHEN product_category_name IN ('bebes','fraldas_higiene') THEN 'Baby items and hygiene'
WHEN product_category_name IN ('cama_mesa_banho','casa_conforto_2','casa_conforto','casa_construcao','utilidades_domesticas',
'moveis_cozinha_area_de_servico_jantar_e_jardim','portateis_cozinha_e_preparadores_de_alimentos','la_cuisine') THEN 'Home and Living'
WHEN product_category_name IN ('livros_interesse_geral','livros_importados','livros_tecnicos','papelaria') THEN 'Books and Stationery'
WHEN product_category_name IN ('artigos_de_natal','artigos_de_festas') THEN 'Party supplies'
WHEN product_category_name IN ('cine_foto','tablets_impressao_imagem','dvds_blu_ray') THEN 'Photography and Cinema'
WHEN product_category_name IN ('bebidas','alimentos','alimentos_bebidas') THEN 'Food and Beverage'
WHEN product_category_name = 'flores' THEN 'Flowers'
WHEN product_category_name IN ('moveis_quarto','moveis_decoracao','moveis_sala','moveis_colchao_e_estofado','moveis_escritorio') THEN 'Furniture'
WHEN product_category_name IN ('beleza_saude','esporte_lazer') THEN 'Health and Beauty'
WHEN product_category_name IN ('eletrodomesticos','eletrodomesticos_2','eletroportateis','portateis_casa_forno_e_cafe','climatizacao') THEN 'Appliances'
WHEN product_category_name IN ('industria_comercio_e_negocios','market_place') THEN 'Business'
WHEN product_category_name IN ('seguros_e_servicos','sinalizacao_e_seguranca') THEN 'Security'
WHEN product_category_name = 'brinquedos' THEN 'Toys'
END AS new_category
FROM (SELECT DISTINCT pd.product_category_name, pc.product_category_name_english
FROM product_dataset pd LEFT JOIN product_category_name_translation pc ON pd.product_category_name = pc.product_category_name) x;

-- count the number of products for each category
SELECT ctgry.new_category, COUNT(pd.product_id) as new_count 
FROM product_dataset pd LEFT JOIN ctgry ON pd.product_category_name=ctgry.product_category_name
GROUP BY ctgry.new_category WITH ROLLUP
ORDER BY 2 DESC;

-- From 74 categories, we reduced the number of categories to 24
-- Top 3 categories with highest count are: 'home_and_living','health_and_beauty' and 'fashion_and_accessories'

-- create a new products table with new category names
CREATE TABLE product_cln AS SELECT pd.product_id, pd.product_category_name, ctgry.new_category
FROM product_dataset pd LEFT JOIN ctgry ON pd.product_category_name=ctgry.product_category_name;
```
Nulls pertain to the rollup amount:

![image](https://user-images.githubusercontent.com/118359623/224473520-29e07de5-c68b-4b3f-94f2-93fd536ef09e.png)

### C. Order Status Dataset

1. Change Data Set Name
``` sql
ALTER TABLE orders_time_dataset RENAME order_status;
```
2. Replace blank values with NULL and Change data types.

``` sql
CREATE TABLE order_status_cln AS SELECT order_id, customer_id, order_status, 
CAST(order_purchase_timestamp AS DATETIME) order_purchase_timestamp, 
CAST(order_approved_at AS DATETIME) order_approved_at, 
CAST(order_delivered_carrier_date AS DATETIME) order_delivered_carrier_date, 
CAST(order_delivered_customer_date AS DATETIME) order_delivered_customer_date, 
CAST(order_estimated_delivery_date AS DATETIME) order_estimated_delivery_date
FROM (SELECT order_id, 
CASE WHEN customer_id = '' THEN NULL ELSE customer_id END AS customer_id, 
CASE WHEN order_status = '' THEN NULL ELSE order_status END AS order_status,
CASE WHEN order_purchase_timestamp = '' THEN NULL ELSE order_purchase_timestamp END AS order_purchase_timestamp,
CASE WHEN order_approved_at= '' THEN NULL ELSE order_approved_at END AS order_approved_at,
CASE WHEN order_delivered_carrier_date= '' THEN NULL ELSE order_delivered_carrier_date END AS order_delivered_carrier_date,
CASE WHEN order_delivered_customer_date= '' THEN NULL ELSE order_delivered_customer_date END AS order_delivered_customer_date ,
CASE WHEN order_estimated_delivery_date= '' THEN NULL ELSE order_estimated_delivery_date END AS order_estimated_delivery_date
FROM order_status) x;
```

3. Check if each order id from the orders view matches the order status dataset 

``` sql
SELECT oc.order_id, osc.order_id
FROM orders_cln oc LEFT JOIN order_status_cln osc ON oc.order_id=osc.order_id
WHERE osc.order_id IS NULL;
-- all rows from the orders data set has values
```

4. Replace NULL values in the order_delivered_customer_date where order_status is delivered

``` sql
SELECT order_id, order_status,order_approved_at,order_estimated_delivery_date,order_delivered_carrier_date, order_delivered_customer_date
FROM order_status_cln
WHERE order_status = 'delivered' AND order_delivered_customer_date IS NULL; 
-- 8 rows have no delivered date with 1 row missing an order_delivered_carrier_date; assuming that the order_delivered_carrier_date is approximately equal to order_delivered_customer_date, values from
-- order_delivered_carrier_date will be copied to order_delivered_customer_date for the 7 rows. 
-- As for the remaining row, order_estimated_delivery_date will be copied 

UPDATE order_status_cln
SET order_delivered_customer_date = order_delivered_carrier_date
WHERE order_status = 'delivered' AND order_delivered_customer_date IS NULL;

UPDATE order_status_cln
SET order_delivered_customer_date = order_estimated_delivery_date
WHERE order_status = 'delivered' AND order_delivered_customer_date IS NULL;
```
### D. Orders Dataset

1. Check for Duplicate Values and Explain

``` sql
SELECT order_id, count FROM (
SELECT order_id, order_item, product_id, shipping_limit_date, price , COUNT(order_id) OVER (PARTITION BY order_id) as count
FROM order_items_dataset
ORDER BY order_id) x
WHERE count > 1;

-- There are 23787 order ids from the data set that appeared more than one. 
-- Explanation: multiple orders of the same item causes the duplication of order ids. 
```

2. Look for NULLS in the data set

``` sql
SELECT order_id, product_id, price FROM order_items_dataset WHERE order_id IS NULL OR product_id IS NULL 
OR price IS NULL OR seller_id IS NULL OR shipping_limit_date IS NULL ; 
-- findings: no rows were found with null values.
```

3. Check if each product id has only one price

``` sql
SELECT DISTINCT product_id, min_price, max_price FROM 
(SELECT product_id, MAX(price) OVER (PARTITION BY product_id ORDER BY price) AS max_price,
MIN(price) OVER (PARTITION BY product_id ORDER BY price) AS min_price
FROM order_items_dataset) x
WHERE max_price <> min_price ;
-- There are 9457 product ids with markup prices over time; 
```

4. Check if all products in orders dataset has a match in products dataset
``` sql
SELECT oid.order_id, oid.product_id, pd.product_id, oid.price
FROM order_items_dataset oid LEFT JOIN product_cln pd ON oid.product_id=pd.product_id
WHERE pd.product_id IS NULL;
-- all products have a match
```

5. Check for outliers in the price and create new table for wrangling
``` sql
CREATE TABLE orders_cln AS SELECT oid.order_id, oid.seller_id, oid.product_id,pd.new_category,oid.price
FROM order_items_dataset oid LEFT JOIN product_cln pd ON oid.product_id=pd.product_id;
-- it would be difficult to say that a certain product_id has a price that is an outlier since prices vary a lot depending on the product being sold
-- The extremes however, fall on home_and_living (6735 BRL) and contruction tools (0.85 BRL)
```
## VI. Data Wrangling (Overall)

### A. Summary of count of customers to each Brazilian state
``` sql
SELECT DISTINCT bsa.state as State, x.count as Count
FROM 
(SELECT cd.customer_state as state, count(customer_state) as count
FROM customers_dataset cd
GROUP BY customer_state) x INNER JOIN brazil_states_abbrev bsa ON x.state=bsa.state_abbrev
ORDER BY 2 DESC;
```
### B. How many products had a markup of >20%

``` sql
SELECT DISTINCT product_id, min_price, max_price FROM 
(SELECT product_id, MAX(price) OVER (PARTITION BY product_id ORDER BY price) AS max_price,
MIN(price) OVER (PARTITION BY product_id ORDER BY price) AS min_price
FROM order_items_dataset) x
WHERE max_price <> min_price AND max_price >= 1.2*min_price ;
-- 3129 products increased in price by more than 20%; 
-- check what are the categories of these products:

WITH cte AS  (SELECT DISTINCT product_id, min_price, max_price FROM 
(SELECT product_id, MAX(price) OVER (PARTITION BY product_id ORDER BY price) AS max_price,
MIN(price) OVER (PARTITION BY product_id ORDER BY price) AS min_price
FROM order_items_dataset) x
WHERE max_price <> min_price AND max_price >= 1.2*min_price)
SELECT product_cln.new_category,COUNT(*) AS count
FROM cte LEFT JOIN product_cln ON cte.product_id = product_cln.product_id
GROUP BY product_cln.new_category WITH ROLLUP
ORDER BY 2 DESC ;

-- FINDING : The top 4 categories that had the most markup of product prices are: 'Fashion and Accessories' (492), 'Health and Beauty' (416), 'Home and Living' (392) and 'Computer and computer accessories' (320) 
```

### C. Overall Sales by State

``` sql
SELECT oc.order_id, osc.customer_id, oc.seller_id, oc.product_id, oc.new_category, oc.price, osc.order_status, osc.order_delivered_customer_date
FROM orders_cln oc LEFT JOIN order_status_cln osc ON oc.order_id=osc.order_id
WHERE osc.order_status = 'delivered'; -- out of 112650 orders, only 110,197 were delivered 

SELECT state, SUM(price) as price
FROM 
(SELECT oc.order_id, osc.customer_id, oc.seller_id, oc.product_id, oc.new_category, oc.price, osc.order_status, osc.order_delivered_customer_date,bsa.state
FROM orders_cln oc LEFT JOIN order_status_cln osc ON oc.order_id=osc.order_id
INNER JOIN customers_dataset cd ON osc.customer_id=cd.customer_id
INNER JOIN brazil_states_abbrev bsa ON cd.customer_state=bsa.state_abbrev
WHERE osc.order_status = 'delivered') x
GROUP BY state 
ORDER BY 2 DESC;

-- Sao Paulo had the greatest revenue. This is expected since this state had the most number of customers
```
### D. Overall Sales by Top 100 Vendors

``` sql
SELECT seller_id,SUM(price) as price
FROM 
(SELECT oc.order_id, osc.customer_id, oc.seller_id, oc.product_id, oc.new_category, oc.price, osc.order_status,bsa.state
FROM orders_cln oc LEFT JOIN order_status_cln osc ON oc.order_id=osc.order_id
INNER JOIN customers_dataset cd ON osc.customer_id=cd.customer_id
INNER JOIN brazil_states_abbrev bsa ON cd.customer_state=bsa.state_abbrev
WHERE osc.order_status = 'delivered') x
GROUP BY seller_id
ORDER BY 2 DESC
LIMIT 3;

-- From our results: Top 1: '4869f7a5dfa277a7dca6462dcf3b52b2', Top 2: '53243585a1d6dc2643021fd1853d8905' Top 3: '4a3ca9315b744ce9f8e9374361493884'
-- Let's focus on these sellers only and perform our analysis
```
## VII. Data Wrangling (Per Supplier)
``` sql
-- Create view for analysis: 
CREATE VIEW vw AS SELECT oc.order_id, osc.customer_id, oc.seller_id, oc.product_id, oc.new_category, oc.price, osc.order_status,osc.order_delivered_customer_date,bsa.state
FROM orders_cln oc LEFT JOIN order_status_cln osc ON oc.order_id=osc.order_id
INNER JOIN customers_dataset cd ON osc.customer_id=cd.customer_id
INNER JOIN brazil_states_abbrev bsa ON cd.customer_state=bsa.state_abbrev
WHERE osc.order_status = 'delivered';

-- create new view for overall orders regardless of order_status
CREATE VIEW vw2 AS SELECT oc.order_id, osc.customer_id, oc.seller_id, oc.product_id, oc.new_category, oc.price, osc.order_status,osc.order_purchase_timestamp,osc.order_approved_at,
osc.order_delivered_carrier_date,osc.order_delivered_customer_date,bsa.state
FROM orders_cln oc LEFT JOIN order_status_cln osc ON oc.order_id=osc.order_id
INNER JOIN customers_dataset cd ON osc.customer_id=cd.customer_id
INNER JOIN brazil_states_abbrev bsa ON cd.customer_state=bsa.state_abbrev;
```
### A. Sales per category per month 
``` sql
-- S1 '4869f7a5dfa277a7dca6462dcf3b52b2':
-- categnumber was added for Power BI sorting
-- There are no sales for May 2017 so I inserted a row 
SELECT monthnumber, month , year, category, SUM(revenue) as revenue,
CASE WHEN category = 'Fashion and Accessories' THEN 1
WHEN category = 'Electronics' THEN 2
WHEN category = 'Cool Stuff' THEN 3
WHEN category = 'Health and Beauty' THEN 4
WHEN category = 'Gaming' THEN 5
WHEN category = 'Construction tools and General house tools' THEN 6
WHEN category = 'Computer and computer accessories' THEN 7
END AS categnumber
FROM (
SELECT monthname(order_delivered_customer_date) AS month, MONTH(order_delivered_customer_date) as monthnumber, year(order_delivered_customer_date) AS year, new_category AS category, price AS revenue
FROM vw
WHERE seller_id = '4869f7a5dfa277a7dca6462dcf3b52b2') x
GROUP BY month,monthnumber,year,category
UNION ALL
SELECT 5, 'May', 2017,'Fashion and Accessories', 0 , 1
ORDER BY 3,1,5;

-- S2 '53243585a1d6dc2643021fd1853d8905':
-- There are no sales for June 2018 so I inserted a row 
SELECT monthnumber, month , year, category, SUM(revenue) as revenue,
CASE WHEN category = 'Electronics' THEN 1
WHEN category = 'Computer and computer accessories' THEN 2
END AS categnumber
FROM (
SELECT monthname(order_delivered_customer_date) AS month, MONTH(order_delivered_customer_date) as monthnumber, year(order_delivered_customer_date) AS year, new_category AS category, price AS revenue
FROM vw
WHERE seller_id = '53243585a1d6dc2643021fd1853d8905') x
GROUP BY month,monthnumber,year,category
UNION ALL
SELECT 6, 'June', 2018,'Computer and computer accessories', 0 , 2
ORDER BY 3,1,5;

-- S3 '4a3ca9315b744ce9f8e9374361493884'
SELECT monthnumber, month , year, category, SUM(revenue) as revenue,
CASE WHEN category = 'Home and Living' THEN 1
WHEN category = 'Furniture' THEN 2
WHEN category = 'Cool Stuff' THEN 5
WHEN category = 'Toys' THEN 4
WHEN category = 'Baby items and hygiene' THEN 3
END AS categnumber
FROM (
SELECT monthname(order_delivered_customer_date) AS month, MONTH(order_delivered_customer_date) as monthnumber, year(order_delivered_customer_date) AS year, new_category AS category, price AS revenue
FROM vw
WHERE seller_id = '4a3ca9315b744ce9f8e9374361493884') x
GROUP BY month,monthnumber,year,category
ORDER BY 3,1,5;
```
### B. Sales per location

``` sql
-- S1 '4869f7a5dfa277a7dca6462dcf3b52b2' 
SELECT year, state,SUM(price) as revenue
FROM (
SELECT year(order_delivered_customer_date) AS year, state,price
FROM vw
WHERE seller_id = '4869f7a5dfa277a7dca6462dcf3b52b2') x
GROUP BY year,state
ORDER BY 1,2;

-- S2 '53243585a1d6dc2643021fd1853d8905'
SELECT year, state,SUM(price) as revenue
FROM (
SELECT year(order_delivered_customer_date) AS year, state,price
FROM vw
WHERE seller_id = '53243585a1d6dc2643021fd1853d8905') x
GROUP BY year,state
ORDER BY 1,3;

-- S3 '4a3ca9315b744ce9f8e9374361493884'
SELECT year, state,SUM(price) as revenue
FROM (
SELECT year(order_delivered_customer_date) AS year, state,price
FROM vw
WHERE seller_id = '4a3ca9315b744ce9f8e9374361493884') x
GROUP BY year,state
ORDER BY 1,3 DESC;
```
### C. New Customers and Returning Customers

``` sql
-- S1 '4869f7a5dfa277a7dca6462dcf3b52b2' 

SELECT 2018 AS year, COUNT(*)  as new_customers_2018
FROM(
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' AND YEAR(order_delivered_customer_date)=2018
EXCEPT
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' AND YEAR(order_delivered_customer_date)=2017) x;

SELECT 2018 as year, COUNT(*)  as RETURNING_CUSTOMER
FROM(
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' AND YEAR(order_delivered_customer_date)=2018
INTERSECT
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' AND YEAR(order_delivered_customer_date)=2017) x;

-- S2 '53243585a1d6dc2643021fd1853d8905'

SELECT 2018 AS year, COUNT(*)  as new_customers_2018
FROM(
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='53243585a1d6dc2643021fd1853d8905' AND YEAR(order_delivered_customer_date)=2018
EXCEPT
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='53243585a1d6dc2643021fd1853d8905' AND YEAR(order_delivered_customer_date)=2017) x;

SELECT 2018 as year, COUNT(*)  as RETURNING_CUSTOMER
FROM(
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='53243585a1d6dc2643021fd1853d8905' AND YEAR(order_delivered_customer_date)=2018
INTERSECT
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='53243585a1d6dc2643021fd1853d8905' AND YEAR(order_delivered_customer_date)=2017) x;

-- S3 '4a3ca9315b744ce9f8e9374361493884'

SELECT 2018 AS year, COUNT(*)  as new_customers_2018
FROM(
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884' AND YEAR(order_delivered_customer_date)=2018
EXCEPT
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884' AND YEAR(order_delivered_customer_date)=2017) x;

SELECT 2018 as year, COUNT(*)  as RETURNING_CUSTOMER
FROM(
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884' AND YEAR(order_delivered_customer_date)=2018
INTERSECT
SELECT DISTINCT customer_id
FROM vw
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884' AND YEAR(order_delivered_customer_date)=2017) x;
```

### D. Product Rankings and Sales volume per product

``` sql
-- S1 '4869f7a5dfa277a7dca6462dcf3b52b2' 

SELECT year, product_id, new_category as category, units_sold, total_revenue, DENSE_RANK() OVER (PARTITION BY year ORDER BY total_revenue DESC) AS product_ranking
FROM (SELECT year(order_delivered_customer_date) AS year, product_id, new_category, COUNT(order_id) as units_sold, SUM(price) as total_revenue
FROM vw
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' 
GROUP BY year,product_id,new_category
ORDER BY 3 DESC) x;

-- S2 '53243585a1d6dc2643021fd1853d8905'

SELECT year, product_id, new_category as category, units_sold, total_revenue, DENSE_RANK() OVER (PARTITION BY year ORDER BY total_revenue DESC) AS product_ranking
FROM (SELECT year(order_delivered_customer_date) AS year, product_id, new_category, COUNT(order_id) as units_sold, SUM(price) as total_revenue
FROM vw
WHERE seller_id='53243585a1d6dc2643021fd1853d8905'
GROUP BY year,product_id,new_category
ORDER BY 3 DESC) x;

-- S3 '4a3ca9315b744ce9f8e9374361493884'

SELECT year, product_id, new_category as category, units_sold, total_revenue, DENSE_RANK() OVER (PARTITION BY year ORDER BY total_revenue DESC) AS product_ranking
FROM (SELECT year(order_delivered_customer_date) AS year, product_id, new_category, COUNT(order_id) as units_sold, SUM(price) as total_revenue
FROM vw
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884'
GROUP BY year,product_id,new_category
ORDER BY 3 DESC) x;

```

### E. Percent completed orders

``` sql
-- S1 '4869f7a5dfa277a7dca6462dcf3b52b2' 
SELECT x.year,y.order_status, y.total_orders,x.order_status, x.total_orders,y.total_orders/x.total_orders AS sales_orders_ratio
FROM (SELECT year(order_purchase_timestamp) AS year, 'all_orders' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' 
GROUP BY year(order_purchase_timestamp)
UNION ALL 
SELECT year(order_purchase_timestamp) AS year, 'delivered' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' AND order_status = 'delivered' 
GROUP BY year(order_purchase_timestamp))x CROSS JOIN
(SELECT year(order_purchase_timestamp) AS year, 'all_orders' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' 
GROUP BY year(order_purchase_timestamp)
UNION ALL 
SELECT year(order_purchase_timestamp) AS year, 'delivered' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4869f7a5dfa277a7dca6462dcf3b52b2' AND order_status = 'delivered' 
GROUP BY year(order_purchase_timestamp))y
WHERE x.year=y.year AND x.order_status='all_orders' AND y.order_status='delivered'
ORDER BY x.year;

-- S2 '53243585a1d6dc2643021fd1853d8905'
SELECT x.year,y.order_status, y.total_orders,x.order_status, x.total_orders,y.total_orders/x.total_orders AS sales_orders_ratio
FROM (SELECT year(order_purchase_timestamp) AS year, 'all_orders' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='53243585a1d6dc2643021fd1853d8905'
GROUP BY year(order_purchase_timestamp)
UNION ALL 
SELECT year(order_purchase_timestamp) AS year, 'delivered' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='53243585a1d6dc2643021fd1853d8905' AND order_status = 'delivered' 
GROUP BY year(order_purchase_timestamp))x CROSS JOIN
(SELECT year(order_purchase_timestamp) AS year, 'all_orders' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='53243585a1d6dc2643021fd1853d8905'
GROUP BY year(order_purchase_timestamp)
UNION ALL 
SELECT year(order_purchase_timestamp) AS year, 'delivered' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='53243585a1d6dc2643021fd1853d8905' AND order_status = 'delivered' 
GROUP BY year(order_purchase_timestamp))y
WHERE x.year=y.year AND x.order_status='all_orders' AND y.order_status='delivered'
ORDER BY x.year;

-- S3 '4a3ca9315b744ce9f8e9374361493884'
SELECT x.year,y.order_status, y.total_orders,x.order_status, x.total_orders,y.total_orders/x.total_orders AS sales_orders_ratio
FROM (SELECT year(order_purchase_timestamp) AS year, 'all_orders' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884'
GROUP BY year(order_purchase_timestamp)
UNION ALL 
SELECT year(order_purchase_timestamp) AS year, 'delivered' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884' AND order_status = 'delivered' 
GROUP BY year(order_purchase_timestamp))x CROSS JOIN
(SELECT year(order_purchase_timestamp) AS year, 'all_orders' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884'
GROUP BY year(order_purchase_timestamp)
UNION ALL 
SELECT year(order_purchase_timestamp) AS year, 'delivered' as order_status, COUNT(DISTINCT order_id) AS total_orders
FROM vw2
WHERE seller_id='4a3ca9315b744ce9f8e9374361493884' AND order_status = 'delivered' 
GROUP BY year(order_purchase_timestamp))y
WHERE x.year=y.year AND x.order_status='all_orders' AND y.order_status='delivered'
ORDER BY x.year;
``` 
### F. Annual Growth Rate

``` sql
-- S1 '4869f7a5dfa277a7dca6462dcf3b52b2' 
SELECT Year, Sales, LAG(Sales) OVER (ORDER BY Year) as lag_sales,
COALESCE((Sales-LAG(Sales) OVER (ORDER BY Year))/LAG(Sales) OVER (ORDER BY Year),"") AS Growth_Rate
FROM (SELECT YEAR(order_delivered_customer_date) AS Year, SUM(price) as Sales
FROM vw
WHERE seller_id = '4869f7a5dfa277a7dca6462dcf3b52b2' 
GROUP BY YEAR(order_delivered_customer_date)) x;

-- S2 '53243585a1d6dc2643021fd1853d8905'
SELECT Year, Sales, LAG(Sales) OVER (ORDER BY Year) as lag_sales,
COALESCE((Sales-LAG(Sales) OVER (ORDER BY Year))/LAG(Sales) OVER (ORDER BY Year),"") AS Growth_Rate
FROM (SELECT YEAR(order_delivered_customer_date) AS Year, SUM(price) as Sales
FROM vw
WHERE seller_id = '53243585a1d6dc2643021fd1853d8905'
GROUP BY YEAR(order_delivered_customer_date)) x;

-- S3 '4a3ca9315b744ce9f8e9374361493884'
SELECT Year, Sales, LAG(Sales) OVER (ORDER BY Year) as lag_sales,
COALESCE((Sales-LAG(Sales) OVER (ORDER BY Year))/LAG(Sales) OVER (ORDER BY Year),"") AS Growth_Rate
FROM (SELECT YEAR(order_delivered_customer_date) AS Year, SUM(price) as Sales
FROM vw
WHERE seller_id = '4a3ca9315b744ce9f8e9374361493884'
GROUP BY YEAR(order_delivered_customer_date)) x;

``` 
## VIII. Data Visualization and Findings

Please check the Power BI Dashboard (on top of this report)

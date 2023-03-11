# Sales Analysis for the Top 3 Sellers from Olist, an E-commerce dataset

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






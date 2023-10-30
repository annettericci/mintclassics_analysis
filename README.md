# mintclassics_analysis
My analysis process for the Coursera project "Analyze Data in a Model Car Database with MySQL Workbench"

# Agenda for this case study  

This exploratory analysis will cover the following deliverables:  

1.  Project description  
2.  Description of data sources  
3.  Data exploration + analysis  
4.  Suggestions for inventory reduction supported by SQL queries  


# 1) Project description  

## 1.1) The company and stakeholders, situation and business problem  

Mint Classics Company, a retailer of classic model cars and other vehicles, is looking at closing one of their storage facilities. 

To support a data-based business decision, they are looking for suggestions and recommendations for reorganizing or reducing inventory, while still maintaining timely service to their customers. For example, they would like to be able to ship a product to a customer within 24 hours of the order being placed.

As a data analyst, you have been asked to use MySQL Workbench to familiarize yourself with the general business by examining the current data. You will be provided with a data model and sample data tables to review. You will then need to isolate and identify those parts of the data that could be useful in deciding how to reduce inventory.

The overall objective is to give recommendations for reducing inventory with the goal of closing one of the storage facilities.  

## 1.2) The business task(s)

1. Explore products currently in inventory.  

2. Determine important factors that may influence inventory reorganization/reduction.  

3. Provide analytic insights and data-driven recommendations.  

## 1.3) Guiding questions  

* Where are items stored and if they were rearranged, could a warehouse be eliminated?  

* How are inventory numbers related to sales figures? Do the inventory counts seem appropriate for each item?  

* Are we storing items that are not moving? Are any items candidates for being dropped from the product line?  

## 1.4) Key solution metrics  

In order to be able to answer above business question(s), the following key metrics are to be calculated.  

1. Most and least popular types of models  

2. Summary of inventory at each warehouse location  

3. Correlation between sales and amount of inventory in stock  


# 2) Description of data sources  

In this analysis, I am using the provided "mintclassicsDB" from the Coursera project "Analyze Data in a Model Car Database with MySQL Workbench".   
This is a fictional database for learning purposes.  

Previewing the included tables:  

* warehouses - lists the 4 warehouses  
* products - contains information about different model products  
* productlines - descriptions of different product lines  
* orderdetails - order id's, products, quantity, and price  
* orders - also contains order id's, relevant dates, status, and customer id's  
* customers - customer id's with contact information  
* payments - customer id, check id, payment date, amount  
* employees - list of employee information  
* offices - list of 7 offices  

The relevant tables for this project will be:  

- warehouses  

- products  

- orderdetails (more financial information than orders, more descriptive)  


# 3) Data exploration and analysis  

## Most and least popular models  

I will take a look at the 5 most profitable and the 5 least profitable products.  

Most popular models:  

- S18_3232 1992 Ferrari 360 Spider red  
- S18_1342 1937 Lincoln Berline  
- S700_4002 American Airlines: MD-11S  
- S18_3856 1941 Chevrolet Special Deluxe Cabriolet  
- S50_1341 1930 Buick Marquette Phaeton  

Least popular models:  

- S18_4933 1957 Ford Thunderbird  
- S24_1046 1970 Chevy Chevelle SS 454  
- S24_3969 1936 Mercedes Benz 500k Roadster  
- S18_2248 1911 Ford Town Car  
- S18_2870 1999 Indy 500 Monte Carlo SS  

There is no obvious trend here as you have both classic and vintage cars making up the majority of both the highest and lowest sellers.  


## Summary of inventory at each warehouse:  

Warehouse a (North): planes + motorcycles  
  12 planes, 13 motorcycles = 25 total products  
  
Warehouse b (East): classic cars  
  38 total products for classic cars  
  
Warehouse c (West): vintage cars  
  24 total products for vintage cars  
  
Warehouse d (South): Trucks + buses, ships, trains  
  11 trucks + buses, 9 ships, 3 trains = 23 total  

  
## Correlation between sales and amount of inventory in stock   
For this part, I chose to use R because I prefer R for finding relationships between variables.  

I imported the "inv" table that I created in SQL to RStudio.  

install.packages("tidyverse")  
library(tidyverse)  
mintclassics_inv <- read.csv("C:/Users/Annette/OneDrive/Documents/R Files/mintclassics r files/mintclassics_inv.csv")  

ggplot(data = mintclassics_inv) +  
  geom_point(mapping = aes(x = quantityInStock, y = TotalOrdered))  

![Rplot1](https://github.com/annettericci/mintclassics_analysis/assets/149432800/59cbbb74-91ac-44a0-893b-5cf81612d98d)

This graph shows an interesting trend, however I would like to remove the one far outlier to make sure the trend still holds when I "zoom in".  

x <- mintclassics_inv %>% filter(TotalOrdered < 1808)

ggplot(data = x) +  
  geom_point(mapping = aes(x = quantityInStock, y = TotalOrdered)) +  
  geom_smooth(mapping = aes(x = quantityInStock, y = TotalOrdered))  

![Rplot2](https://github.com/annettericci/mintclassics_analysis/assets/149432800/9eb0e52f-0c69-4a2f-b357-099b18ab2980)

So, we can see even more clearly that there is no correlation between quantity in stock and the amount needed for orders.    

Ideally, you would want more in stock for more popular items, and less in stock for less popular items.  

In the case of the outlier, the "S18_3232, 1992 Ferrari 360 Spider red" model, they sell 1808 units, but carry 8347 units in stock. That's about 4.6x greater than what they need!  

So, it makes sense to carry a lot of their most popular product, but should it be that much?  

Some products are even worse in their stock-to-needed ratio. For example, the "S12_2823 2002 Suzuki XREO" model has 9997 units in stock, even though it only sells 1028 units, a difference of about 9.7x!  

So there is clearly a need to get rid of some excess products. Before we get into that, I wanted to check if there is any relationship between the amount in stock and any other variables:  

ggplot(data = mintclassics_inv) +  
  geom_point(mapping = aes(x = quantityInStock, y = productLine))  

![Rplot3](https://github.com/annettericci/mintclassics_analysis/assets/149432800/5ee20edc-a49a-4c27-96b2-0a823f94d0e5)

While some product lines obviously have more unique items than others, the spread of quantity in stock is pretty even across the board.  

ggplot(data = mintclassics_inv) +  
  geom_point(mapping = aes(x = quantityInStock, y = buyPrice))  

  ![Rplot4](https://github.com/annettericci/mintclassics_analysis/assets/149432800/93b513c2-547d-4759-8485-bfc72366378a)

There also seems to be no relationship between the price of the item and how many they keep in stock.  

ggplot(data = mintclassics_inv) +  
  geom_point(mapping = aes(x = quantityInStock, y = TotalProfit))  
  
  ![Rplot5](https://github.com/annettericci/mintclassics_analysis/assets/149432800/4c0b137e-7245-4a67-a044-b4af755b3bbb)

Even when looking at total profit, you see just as many in stock for less profitable products as the more profitable products.  

So, after checking for relationships between the quantity in stock and many other variables, it seems there is not a good system in place for determining the amount to keep in stock. That means there's a lot of room for reducing inventory.  

## Checking the total capacity of each warehouse  

> Note: I could not find any legend provided in the Coursera lesson that gave the definition of the "PctCap" variable in the warehouse table. However, after searching online, the only other instance I found using this abbreviation was "Percent Capacity", and that seems to make sense for this scenario, so that is how I'm defining it moving forward.  

a <- mintclassics_inv %>% filter(warehouseCode == "a")  
b <- mintclassics_inv %>% filter(warehouseCode == "b")  
c <- mintclassics_inv %>% filter(warehouseCode == "c")  
d <- mintclassics_inv %>% filter(warehouseCode == "d")  

Warehouse A (North):  
sum(a$quantityInStock)  
In Stock: 131,688  
Pct Capacity: 72%  
At 100% Capacity: 182,900  

Warehouse B (East):  
sum(b$quantityInStock)  
In Stock: 211,450  
Pct Capacity: 67%  
At 100% Capacity: 315,597  

Warehouse C (West):  
sum(c$quantityInStock)  
In Stock: 124,880  
Pct Capacity: 50%  
At 100% Capacity: 249,760  

Warehouse D (South):  
sum(d$quantityInStock)  
In Stock: 79,380  
Pct Capacity: 75%  
At 100% Capacity: 105,840  

So, as we can see warehouse D (South) has the least capacity for storage.  
So, we could easily move the inventory from warehouse D to warehouse C, which is only at 50% capacity and the second largest warehouse.  

If we added all of warehouse D's inventory to warehouse C, it would amount to 204,260 items, which would then bring warehouse C up to about 82% capacity.  

That still leaves a good amount of wiggle room. 

### Going back to our second business task:   
## Determine important factors that may influence inventory reorganization/reduction.  

We need to figure out a better system for determining how much stock to keep in inventory. Some items are kept in great excess to what is needed. 

I did some research on best practices for inventory management as this is an area I'm not personally familiar with.  

> Ideally, your business should hold enough inventory to meet customer demand, and enough safety stock to tide you over until you replenish, but not so much that you’re left with deadstock.  
> To avoid overstocking, [try] calculating safety stock levels.  
[Via www.shipbob.com](https://www.shipbob.com/inventory-kpis/holding-costs/#6)  

Calculating safety stock usually requires a lot of variables for max accuracy, such as supply and demand variability, lead time, seasonality, etc. But, since I am lacking these variables in this practice dataset, I will stick with a basic guideline:

> Some inventory managers use rules of thumb to govern safety stock amounts, such as the guideline that safety stock levels should equal 10-20% of cycle stock.  
[Via ascm.org](https://www.ascm.org/ascm-insights/safety-stock-a-contingency-plan-to-keep-supply-chains-flying-high/)  

Therefore, I recommend keeping enough inventory on hand to meet a full cycle of orders, plus 20% more, or 120% of the amount needed overall.  

I will go back to SQL for this next part as I want to do some calculations. I prefer SQL for filtering data and doing calculations, while I prefer R for finding relationships among variables.  

In SQL, I created a new table, IdealOnHand, with a calculated field showing the ideal number of each item to keep on hand according to the above guideline. I also added a field to show how much difference there was between Ideal OH and actual OH.  

What this table tells us is that there are many items that should be reduced in quantity, and some items that even need to be ordered up on a little.  

# 4) Suggestions for inventory reduction  
## Based on these findings, my suggestions would be one or a combination of the following:  

1. Adopt an inventory management software  

2. Set reorder points to the IdealOnHand values, reassess this number yearly  

3. Reduce stock of items with the largest differences between IdealOnHand and actual on hand by donating, bundling, offering discounts, etc.

4. Close warehouse D and move the remaining inventory to warehouse C to reduce maintenance costs  

5. After reducing on hand stock closer to ideal levels, reassess warehouse capacities and consider merging warehouses again if there is still a lot of free capacity  

### Limitations and further research:  

The data did not include holding costs or lead times, which would be helpful to determine a more accurate IdealOnHand number.  

As mentioned earlier, the definition of the "PctCap" variable was not clear, so I assumed it meant "Percent Capacity" based on searching for other instances of this variable being used.  

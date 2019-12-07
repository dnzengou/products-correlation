# Products Correlation
Optimizing the products price for physical and online store; proactively recommending adequate actions on promotions and order amounts


## Problem statement


### Context
We present a set of products in which we are trying to determine actions to perform on products that present a correlation with each other, in particular negative correlation. For example, sales promotion or order amounts readjustment; further on, possibly which products should continue to be sold, which products to remove from the inventory.
The data files contain historical sales data and active inventory. More details below.


* **Objective:**
We want to determine which products from the store inventory should be retained to sell (in particular, applying promotional actions) and the ones to discard (or to decrease the orders).

* **Goal:** Among possible ways forward, we intend to build a ***binary classifier*** with a list of products ID which could be retained in the inventory or list of products for which further actions need to be done.

**Dataset**
In addition to all publicly available data, we have daily sales and associated waste for each product reaching back 1.5 year. We also have the physical location of a subset around 15% of products.

Dataset looks like this:

***EAN 	BEN	  VGR	          DATUM	  Ord_Akt	FSG	  ANTAL	  ANTAL_KG***<br>
  64	  3123	Lösviktsgodis	159	2019-11-05	11	  3913.86	106	48.760
  65	  3123	Lösviktsgodis	159	2019-11-06	11	  3549.05	98	44.215

We also have two months worth of receipts, e.g. for each transaction what products sold together along with timestamps (date) in a DataFrame with the following columns:<br>
***transaction_id	date	subtotal	number_of_items	ean	product	quantity	value	is_discount	incomplete_shopping***


A few comments about the attributes included, as we realize we may have some attributes that are unnecessary or may need to be explained.

    SKU_number: This is the unique identifier for each product.
    EAN = standardized barcode and marked on most commercialized products currently available at the stores. EAN is a universal code throughout the world. ※ EAN=European Article Number
    Ord_Akt = indicator of promotions
    Order: Just a sequential counter. Can be ignored
    SoldFlag: 1 = sold in past 6 mos. 0 = Not sold
    MarketingType = Two categories of how we market the product. This should probably be ignored, or better yet, each type should be considered independently.

    New_Release_Flag = Any product that has had a future release (i.e., Release Number > 1)

<br>with</br>






### Reference
[AlibabaCloudDocs/kvstore](https://github.com/AlibabaCloudDocs/kvstore/blob/master/intl.en-US/Best%20Practices/Build%20an%20e-commerce%20short-term%20sales%20promotion%20system%20by%20using%20ApsaraDB%20for%20Redis.md)<br>
[Solving Case study : Optimize the Products Price for an Online Vendor](https://www.analyticsvidhya.com/blog/2016/07/solving-case-study-optimize-products-price-online-vendor-level-hard/)<br>
[Finding Correlations in Non-Linear Data](https://www.freecodecamp.org/news/how-machines-make-predictions-finding-correlations-in-complex-data-dfd9f0d87889/)<br>
[Exploratory Data Analysis (EDA) and Data Visualization with Python](https://kite.com/blog/python/data-analysis-visualization-python/)<br>




<hr>



Use the data control module to cache the promotion commodity data to the read/write splitting instance in advance, and specify the flag for starting the promotion as follows:

``` {#codeblock_g99_9xh_n6s}
"goodsId_count": 100 //The total number of commodities.
"goodsId_start": 0   //The flag to indicate that the promotion starts.
"goodsId_access": 0  //The number of orders that the promotion system accepts.
```

1.  Before the promotion starts, the server cluster for the short-term sales promotion reads goodsId\_start as 0, and indicates in the response that the promotion has not started.

2.  When the data control module changes goodsId\_start to 1, the promotion starts.

3.  The server cluster for the short-term sales promotion caches the promotion start flag and accepts order requests. The cluster records the requests to goodsId\_access. The number of remaining commodities is the result of the value of goodsId\_count minus the value of goodsId\_access.

4.  After the number of orders that the short-term sales promotion system accepts reaches the value of goodsId\_count, the short-term sales promotion system intercepts all requests. The number of remaining commodities is 0.


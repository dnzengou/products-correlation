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

***EAN 	BEN	  VGR	          DATUM	  Ord_Akt	FSG	  ANTAL	  ANTAL_KG***
  64	  3123	Lösviktsgodis	159	2019-11-05	11	  3913.86	106	48.760
  65	  3123	Lösviktsgodis	159	2019-11-06	11	  3549.05	98	44.215

We also have two months worth of receipts, e.g. for each transaction what products sold together along with timestamps (date) in a DataFrame with the following columns:
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



# Build an e-commerce short-term sales promotion system by using ApsaraDB for Redis {#concept_pmq_cgx_wdb .concept}

## Background {#section_6ft_4th_b85 .section}

The short-term sales promotion is a common method for low-price promotion and brand marketing in the e-commerce industry. This method can bring more traffic and increase the visibility of your platform. An excellent short-term sales promotion system can improve the stability and fairness of the platform, provide a better user experience, and enhance the reputation of the platform. Therefore, this system can maximize the value of short-term sales promotions.

This topic describes the short-term sales promotion system that processes high-concurrency requests based on the cache feature of ApsaraDB for Redis.

## Characteristics of a short-term sales promotion {#section_hfo_xmr_v1g .section}

A short-term sales promotion is intended to sell scarce or special commodities for specified quantities in a timed manner, and attract a large number of buyers. However, only a few buyers place orders during the promotion. Therefore, the short-term sales promotion brings visits and order requests dozens or hundreds of times that in normal sales on your platform in a short period.

A short-term sales promotion is divided into three phases:

-   Before the promotion: buyers constantly refresh the commodity details page, and the number of requests of the page reaches the instantaneous peak.
-   The promotion starts: buyers click the promotion button, and the number of order requests reaches the instantaneous peak.
-   After the promotion: some buyers that have successfully placed orders continue to refresh their orders or return the orders. Most buyers continue to refresh the commodity details page and wait for returning orders.

A database processes the requests based on row-level rocking when buyers submit orders. The database allows requests that hold the lock to query inventories and place orders. However, if the database cannot process high-concurrency requests, the service may be blocked. This causes server downtime to buyers.

## Short-term sales promotion system {#section_1dj_syv_v2k .section}

The traffic to the short-term sales promotion system is high, but the valid traffic is low. Based on the system hierarchy, if you verify and intercept invalid traffic in advance in each phase, you can reduce large amounts of invalid traffic to the database.

**Use the browser cache and Content Delivery Network \(CDN\) to process traffic that comes to static pages**

Before the short-term sales promotion, buyers constantly refresh the commodity details page. This results in a large number of requests to the page. Therefore, you can separate the commodity details page for the short-term sales promotion from that for normal sales. For the commodity details page in the short-term sales promotion, the Web server processes the clicks on the promotion button in real time, and the short-term sales promotion system caches static elements to the browser and CDN. In this way, only a fraction of the traffic caused by refreshing the page before the promotion goes to the Web server.

**Use a read/write splitting instance of ApsaraDB for Redis to cache and intercept traffic**

You can first use the CDN service to intercept traffic, and then use a read/write splitting instance of ApsaraDB for Redis to intercept more traffic. The read/write splitting instance can support more than 600,000 queries per second \(QPS\), and process a large number of read requests.

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


In this way, the short-term sales promotion system accepts only a small fraction of the order requests. In the case of high concurrency, you can allow a bit more traffic to the system. Therefore, you can control the percentage of orders that the system accepts.

**Use a master-replica instance of ApsaraDB for Redis to accelerate inventory deduction**

After accepting an order, the short-term sales promotion system checks the order information and deducts the inventory. To avoid direct access to a database, you can use a master-replica instance of ApsaraDB for Redis to deduct the inventory. The instance supports more than 100,000 QPS. The ApsaraDB for Redis instance optimizes the inventory query, intercepts invalid order requests in advance, and increases the overall throughput of the short-term sales promotion system.

You can use the data control module to cache the inventory to the ApsaraDB for Redis instance in advance. The instance stores the commodity data for promotion in a hash table.

``` {#codeblock_thm_coz_v86}
"goodsId" : {
    "Total": 100
    "Booked": 100
}
```

To deduct the inventory, the short-term sales promotion server runs the following Lua script and connects to the ApsaraDB for Redis instance to obtain the order permission. Due to Redis single-thread model, the Lua script ensures the atomicity of multiple commands.

``` {#codeblock_aqo_m2g_u13}
local n = tonumber(ARGV[1])
if not n  or n == 0 then
    return 0       
end                
local vals = redis.call("HMGET", KEYS[1], "Total", "Booked");
local total = tonumber(vals[1])
local blocked = tonumber(vals[2])
if not total or not blocked then
    return 0       
end                
if blocked + n <= total then
    redis.call("HINCRBY", KEYS[1], "Booked", n)                                   
    return n;   
end                
return 0
```

Run the `SCRIPT LOAD` command to cache the Lua script to the ApsaraDB for Redis instance in advance, and then run the `EVALSHA` command to call the script. This method requires less network bandwidth than you directly run the `EVAL` command.

``` {#codeblock_554_dq3_23s}
redis 127.0.0.1:6379>SCRIPT LOAD "lua code"
"438dd755f3fe0d32771753eb57f075b18fed7716"
redis 127.0.0.1:6379>EVAL 438dd755f3fe0d32771753eb57f075b18fed7716 1 goodsId 1 
```

If the ApsaraDB for Redis instance returns the value n as the number of commodities that buyers have ordered, the short-term sales promotion system determines that the current inventory deduction is successful.

**Use a master-replica instance of ApsaraDB for Redis to asynchronously write order data to the database based on message queues**

The short-term sales promotion system writes order data to the database after successful inventory deduction. For a few commodities, the system can directly perform operations in the database. If the number of commodities for promotion is more than 10,000 or 100,000, lock conflicts may occur and cause performance bottlenecks in the database. Therefore, to avoid direct operations in the database, the short-term sales promotion system writes order data to message queues to complete the order process.

1.  The ApsaraDB for Redis instance provides message queues in a list structure.

    ``` {#codeblock_m1h_x49_56f}
     orderList {
         [0] = {Order content} 
         [1] = {Order content}
         [2] = {Order content}
         ...
     }
    ```

2.  The short-term sales promotion system writes order content to the ApsaraDB for Redis instance.

    ``` {#codeblock_7or_pie_hsu}
    LPUSH orderList {Order content}
    ```

3.  The asynchronous order module sequentially retrieves order data from the ApsaraDB for Redis instance and writes order data to the database.

    ``` {#codeblock_080_yd7_135}
     BRPOP orderList 0
    ```


The ApsaraDB for Redis instance provides message queues and asynchronously writes order data to the database to efficiently complete the order process.

**The data control module manages synchronization of promotion data**

At the start, the short-term sales promotion system uses the read/write splitting instance of ApsaraDB for Redis to intercept traffic and allows a fraction of valid traffic to continue the order process. Afterward, the short-term sales promotion system has to process more traffic caused by order authentication failures and returning orders. Therefore, the data control module regularly computes data in the database, and synchronizes the data to the master-replica instance and then to the read/write splitting instance.


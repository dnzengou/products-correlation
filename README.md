# Products Correlation
Optimizing the products price for physical and online store; proactively recommending adequate actions on promotions and order amounts


## Problem statement


### Context
We present a set of products in which we are trying to determine actions to perform on products that present a correlation with each other, in particular negative correlation. For example, sales promotion or order amounts readjustment; further on, possibly which products should continue to be sold, which products to remove from the inventory.
The data files contain historical sales data and active inventory. More details below.


 * **Goal:**
1. We want to determine which products from the store inventory are (negatively) correlated, so that we can splitt them into the ones that should be retained for sell (in particular, applying promotional actions) and the ones to discard (or to decrease the amount of order requests).
2. Among possible ways forward, we intend to build a ***binary classifier*** with a list of products ID which could be retained in the inventory or list of products for which further actions need to be done.

* **Dataset**
In addition to all publicly available data, we have daily sales and associated waste for each product reaching back 1.5 year. We also have the physical location of a subset around 15% of products.

Dataset looks like this:

***EAN 	BEN	  VGR	          DATUM	  Ord_Akt	FSG	  ANTAL	  ANTAL_KG***<br>
  64	  3123	Lösviktsgodis	159	2019-11-05	11	  3913.86	106	48.760
  65	  3123	Lösviktsgodis	159	2019-11-06	11	  3549.05	98	44.215

We also have two months worth of receipts, e.g. for each transaction what products sold together along with timestamps (date) in a DataFrame with the following columns:<br>

***transaction_id	date	subtotal	number_of_items	ean	product	quantity	value	is_discount	incomplete_shopping***


A few comments about the attributes included, as we realize we may have some attributes that are unnecessary or may need to be explained.

    EAN : standardized barcode and marked on most commercialized products currently available at the stores. EAN is a universal code throughout the world. (EAN=European Article Number) Type int
    Ord_Akt : indicator of promotions (type date)
    date: timestamp variable of date type
    value	is_discount : Type boolean

<hr>


## Methods

To proceed to Goal 1., we can either determine whether or not there is correlation within products data and, if negative value (eg. Pearson correlation coefficient PCC = -1), then identify specific products that meet this condition<br>
OR, preferably, let's proceed in a sequential manner with the Exploratory Data Analysis: the first order of operations needed to get a grasp of the what, why, and how of the problem statement, i.e. analyzing the data set by summarizing its main characteristics then visualize them. 

### Finding Correlations within products data

**A bit of theory**

***Correlation***
Between two variables, this concept generally refers to to their ‘relatedness’. It allows for predictions about one variable based upon another.<br>
Nevertheless, beware that “Correlation does not imply causation”. Spurious statistical associations can be found in a multitude of quantities, simply due to chance. Often, a relationship may appear to be causal through high correlation due to some unobserved variables.

Assuming we deal with linear data,<br>
the ***Pearson Correlation coefficient*** (also known as Pearson' r, most common measure of correlation) helps quantifying the degree to which a relationship between two variables can be described by a line. Mathematically, it is defined as “the covariance between two vectors, normalized by the product of their standard deviations”.<br>

Let's briefly introduce the concept of covariance, that is of a statistical measure of association between two variables X and Y. It is therefore related to Pearson's r.




### Exploratory Data Analysis on products data





### Reference
[AlibabaCloudDocs/kvstore](https://github.com/AlibabaCloudDocs/kvstore/blob/master/intl.en-US/Best%20Practices/Build%20an%20e-commerce%20short-term%20sales%20promotion%20system%20by%20using%20ApsaraDB%20for%20Redis.md)<br>
[Solving Case study : Optimize the Products Price for an Online Vendor](https://www.analyticsvidhya.com/blog/2016/07/solving-case-study-optimize-products-price-online-vendor-level-hard/)<br>
[Finding Correlations in Non-Linear Data](https://www.freecodecamp.org/news/how-machines-make-predictions-finding-correlations-in-complex-data-dfd9f0d87889/)<br>
[Exploratory Data Analysis (EDA) and Data Visualization with Python](https://kite.com/blog/python/data-analysis-visualization-python/)<br>
[Introduction to Correlation with pandas](https://blogs.oracle.com/datascience/introduction-to-correlation)




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


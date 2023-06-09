---
title: "An Example of the Exploratory Data Analysis (EDA) Process in R"
author: "Chris Barnett"
date: 2023-05-25
subtitle: ""
excerpt: "Exploratory Data Analysis (EDA) is a critical initial step in data analysis that allows you to understand your data by asking questions, searching for answers, and using those insights to ask additional questions, and the cycle repeats."
output:
  html_document:
    toc: yes
    toc_float: yes
---

<style type="text/css">
h1.title { /* Header 4 - and the author and date headers use this too  */
  font-size: 40px;
  font-style: normal;
  font-weight: bold;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
}

h4.author { /* Header 4 - and the author and date headers use this too  */
  font-size: 20px;
  font-style: normal;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;

}
h4.date { /* Header 4 - and the author and date headers use this too  */
  font-size: 20px;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
}

h2 {/* Header 2 */
  font-size: 24px;
  font-family: Tahoma, Verdana, sans-serif;
  color: Black;
  background-color: WhiteSmoke;
  text-indent: 5px;
}
</style>

---



![](https://d3i6fh83elv35t.cloudfront.net/static/2023/03/2017-01-16T120000Z_2044270320_RC11FA7C4E70_RTRMADP_3_CHINA-AUTOS-SALES-1024x620.jpg)

## Exploratory Data Analysis
Exploratory Data Analysis (EDA) is a critical initial step in data analysis that allows you to understand your data by asking questions, searching for answers, and using those insights to ask additional questions, and the cycle repeats. Much of this information is significantly influenced by Chapter 7 of the book [R for Data Science, first addition](https://r4ds.had.co.nz/exploratory-data-analysis.html) and other sources that I will cite along the way.

## Data acquisition and loading:
The data we will be exploring is the [Used Cars data set](https://www.kaggle.com/austinreese/craigslist-carstrucks-data) from Kaggle, which includes every used vehicle entry within the United States on Craigslist, a personal online advertisements website in the US, in 2021.

First, we will read in the data and take our initial look "glimpse" at the data.

```r
# Reading in the data
used_cars1 <- read_csv('/Users/emmacoleman/Downloads/vehicles.csv')

# Taking a quick glimpse at the used_cars dataframe
glimpse(used_cars1)
```

```
## Rows: 426,880
## Columns: 26
## $ id           <dbl> 7222695916, 7218891961, 7221797935, 7222270760, 721038403…
## $ url          <chr> "https://prescott.craigslist.org/cto/d/prescott-2010-ford…
## $ region       <chr> "prescott", "fayetteville", "florida keys", "worcester / …
## $ region_url   <chr> "https://prescott.craigslist.org", "https://fayar.craigsl…
## $ price        <dbl> 6000, 11900, 21000, 1500, 4900, 1600, 1000, 15995, 5000, …
## $ year         <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ manufacturer <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ model        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ condition    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ cylinders    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ fuel         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ odometer     <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ title_status <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ transmission <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ VIN          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ drive        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ size         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ type         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ paint_color  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ image_url    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ description  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ county       <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ state        <chr> "az", "ar", "fl", "ma", "nc", "ny", "ny", "ny", "or", "pa…
## $ lat          <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ long         <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ posting_date <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
```

This data appears to have many missing values in the early rows of the data set, making it hard to tell if the variable class was read in correctly (many R import functions look at the first ~100 rows of data to guess the variable types). In this situation, you can add a random sample of the data to the **glimpse()** function.    


```r
# Taking another look
glimpse(sample_n(used_cars1, 50))
```

```
## Rows: 50
## Columns: 26
## $ id           <dbl> 7312701380, 7307183005, 7311208938, 7315783291, 731277714…
## $ url          <chr> "https://westmd.craigslist.org/ctd/d/atlanta-2019-chevy-c…
## $ region       <chr> "western maryland", "mobile", "maine", "atlanta", "jackso…
## $ region_url   <chr> "https://westmd.craigslist.org", "https://mobile.craigsli…
## $ price        <dbl> 32990, 18778, 15500, 0, 16599, 18999, 8300, 10500, 18590,…
## $ year         <dbl> 2019, 2016, 2011, 2017, 2013, 2015, 1995, 2012, 2018, 199…
## $ manufacturer <chr> "chevrolet", "buick", "nissan", NA, "lexus", "ford", "toy…
## $ model        <chr> "silverado 1500 ld", "enclave", "titan", "WE SAY YES", "e…
## $ condition    <chr> "good", NA, NA, NA, "excellent", NA, "fair", "like new", …
## $ cylinders    <chr> NA, "6 cylinders", "8 cylinders", NA, "6 cylinders", NA, …
## $ fuel         <chr> "other", "gas", "gas", "gas", "gas", "gas", "gas", "gas",…
## $ odometer     <dbl> 6897, 116431, 99057, 50000, 100343, 194887, 185000, 74000…
## $ title_status <chr> "clean", "clean", "clean", "clean", "clean", "clean", "cl…
## $ transmission <chr> "other", "automatic", "automatic", "manual", "automatic",…
## $ VIN          <chr> "2GCVKNEC8K1187169", NA, NA, NA, "JTHBK1GG2D2081250", "1F…
## $ drive        <chr> NA, "fwd", "4wd", NA, "fwd", "4wd", "4wd", "fwd", "fwd", …
## $ size         <chr> NA, "full-size", NA, NA, "full-size", NA, "mid-size", "co…
## $ type         <chr> "pickup", "SUV", NA, NA, "sedan", NA, "pickup", "sedan", …
## $ paint_color  <chr> "black", "white", "black", NA, "black", "black", "red", "…
## $ image_url    <chr> "https://images.craigslist.org/00Q0Q_1KOmPdt9wZzz_0gw0co_…
## $ description  <chr> "Carvana is the safer way to buy a car During these uncer…
## $ county       <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ state        <chr> "md", "al", "me", "ga", "fl", "mn", "or", "az", "az", "tn…
## $ lat          <dbl> 33.77921, 30.66879, 37.13284, 33.94960, 30.28667, 45.4708…
## $ long         <dbl> -84.41181, -88.10587, -95.78558, -83.99420, -81.54638, -9…
## $ posting_date <dttm> 2021-04-26 10:30:45, 2021-04-15 15:03:37, 2021-04-23 03:…
```

The random sample within the glimpse() function makes it much easier to see that many character variables might work better as factor variables.

## Data cleaning and preprocessing

```r
# Changing some character variables to factors
used_cars2 <- used_cars1 %>%
  mutate(across(.cols = c(region, model, condition, cylinders, 
                          fuel, title_status, transmission, drive, size, type, 
                          paint_color, state), factor))
```

Great! Now we can leverage some of the simplest plots of the [DataExplorer package](https://cran.r-project.org/web/packages/DataExplorer/vignettes/dataexplorer-intro.html) to get a better understanding of this dataframe.

```r
introduce(used_cars2)
```

```
## # A tibble: 1 × 9
##     rows columns discrete_columns continuous_columns all_missing_columns
##    <int>   <int>            <int>              <int>               <int>
## 1 426880      26               19                  6                   1
## # ℹ 4 more variables: total_missing_values <int>, complete_rows <int>,
## #   total_observations <int>, memory_usage <dbl>
```

```r
plot_missing(used_cars2)
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-4-1.png" width="1440" />
With these two plots, we see that most of our variables are discrete (categorical), and significant missing values exist in many columns, including one, the completely empty country variable.

Another helpful package to understand the extent of missing data in your data set is the [naniar package](https://cran.r-project.org/web/packages/naniar/vignettes/getting-started-w-naniar.html). The predominant visualization package in R, [ggplot2](https://ggplot2.tidyverse.org/) often excludes missing values. The *naniar package* works with *ggplot2* to specifically visualize missing values in many interesting ways, one of which we'll explore here.

The plot from the *naniar package* we will use is the **gg_miss_upset()** plot. This plot creates groups (or sets) of variables with missing values and orders each group from largest to smallest. 

```r
library(naniar)
used_cars2 %>%
  select(type, paint_color, drive, VIN, condition, cylinders, size) %>%
gg_miss_upset()
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-5-1.png" width="1440" />

These groups are mutually exclusive (no overlap of missing values), and we can see that the largest group with missing values is the VIN variable. This differs from our previous plot showing the percent of missing observations per variable, where size (71.77%) showed the highest proportion of missing data. However, our **gg_miss_upset()** plot is consistent with our **plot_missing()** graph because the following nine groups in our upset plot all include size as a missing variable. Perhaps the most interesting finding from this plot is that 36,012 rows are missing all of these variables with many combinations missing more than one variable.

We start the EDA process by looking at the variation within each data feature. Here, it is important to pay attention to the most common and least common values and any patterns or distributions that exist. Do these values make sense? If not, what questions can we come up with that we can explore with the data that might help us understand? 

The easiest way to quickly comprehend variation is by visualizing the data. Continues and categorical values usually have vastly different distributions and are visualized differently. We will start by exploring the variation of the categorical variables first.  

Bar plots are a standard method of visualizing categorical data distribution (or counts). However, before making numerous bar plots, I like to get an idea of the number of categories (or levels) in each factor variable. The first loop performed by the **map()** function within the [purrr package](https://purrr.tidyverse.org/index.html) gives us the number of levels in each categorical variable. Here we can see that region (region of the US) and model (model of car) have way more levels than the other variables. We will remove these two before visualizing these data. The second **map()** loop is only sometimes necessary. Still, you will often find redundant levels of a categorical variable that may need to be combined. The second map() loop looks just at the names of each level, and we see that both "land rover" and "rover" are listed as distinct categories. There is also a manufacturer category of "morgan" with zero data that we will remove.

```r
# Calculating the number of levels in each categorical varible
map(fac_used_cars <- used_cars2 %>%
  select_if(.predicate = is.factor), .f = nlevels)
```

```
## $region
## [1] 404
## 
## $model
## [1] 29668
## 
## $condition
## [1] 6
## 
## $cylinders
## [1] 8
## 
## $fuel
## [1] 5
## 
## $title_status
## [1] 6
## 
## $transmission
## [1] 3
## 
## $drive
## [1] 3
## 
## $size
## [1] 4
## 
## $type
## [1] 13
## 
## $paint_color
## [1] 12
## 
## $state
## [1] 51
```

```r
# Listing the labels of each level within each categorical variable (except for
# a few variables the have many [51+] levels)
map(fac_used_cars <- used_cars2 %>%
  select_if(.predicate = is.factor) %>%
    select(-region, -model, -state), .f = levels)
```

```
## $condition
## [1] "excellent" "fair"      "good"      "like new"  "new"       "salvage"  
## 
## $cylinders
## [1] "10 cylinders" "12 cylinders" "3 cylinders"  "4 cylinders"  "5 cylinders" 
## [6] "6 cylinders"  "8 cylinders"  "other"       
## 
## $fuel
## [1] "diesel"   "electric" "gas"      "hybrid"   "other"   
## 
## $title_status
## [1] "clean"      "lien"       "missing"    "parts only" "rebuilt"   
## [6] "salvage"   
## 
## $transmission
## [1] "automatic" "manual"    "other"    
## 
## $drive
## [1] "4wd" "fwd" "rwd"
## 
## $size
## [1] "compact"     "full-size"   "mid-size"    "sub-compact"
## 
## $type
##  [1] "bus"         "convertible" "coupe"       "hatchback"   "mini-van"   
##  [6] "offroad"     "other"       "pickup"      "sedan"       "SUV"        
## [11] "truck"       "van"         "wagon"      
## 
## $paint_color
##  [1] "black"  "blue"   "brown"  "custom" "green"  "grey"   "orange" "purple"
##  [9] "red"    "silver" "white"  "yellow"
```
Right away, we see a few abnormalities within the manufacturer variable. First, we find the car manufacturer label *"morgan"* and see it only has 3 instances in the data. Second, the manufacturer categories "rover" and *"land rover"* exist. This is where domain knowledge can be helpful. Still, it is a fair assumption to add the "rover" category to *"land rover"* and filter out the rows with *"morgan"* as their manufacturer.   

```r
# Investigating the manufacturer category "morgan"
used_cars2 %>%
  filter(manufacturer == "morgan") %>%
  nrow()
```

```
## [1] 3
```

```r
# Combining manufacturer variable categories "rover" and "land rover"
used_cars3 <- used_cars2 %>%
  mutate(manufacturer = ifelse(manufacturer == "rover", 
                               "land rover", manufacturer)) %>%
  filter(manufacturer != "morgan")
```
## Visualizing Data
### Categorical Variables
Now we can again utilize the [DataExplorer package](https://cran.r-project.org/web/packages/DataExplorer/vignettes/dataexplorer-intro.html) to quickly visualize our categorical variables. With the plot_bar() function, we instantly see a lot of useful information. For example, this data set's three most frequently advertised car manufacturers are *Ford*, *Chevrolet*, and *Toyota*. Additionally, (as we saw in our first plot), we will have to deal with significant missing data as the top category for many of these variables is NA.

```r
plot_bar(used_cars3 %>%
            select(-region, -model), maxcat = 51, nrow = 1)
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-8-1.png" width="1440" /><img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-8-2.png" width="1440" /><img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-8-3.png" width="1440" /><img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-8-4.png" width="1440" />
As we look over all these plots, we must ask ourselves if these distributions make sense. Having some domain knowledge is helpful, but keeping in mind the purpose of the data is critical. Often data is not collected and curated with your specific project in mind. Understanding the purpose and the incentives of those creating the data is necessary to understand the data itself. For example, the bar plots of categorical variables "*size*," "*drive*," and "*cylinders*" display that the most common categories for each are "full-size," "4wd," and "6 cylinders" (they also show a significant amount of missing data). Should we believe that the most common vehicles in this data are full-size, 6-cylinder four-wheel drive cars, or is there reporting bias within these variables because advertisers feel these attributes add value to the car they are trying to sell? These questions will be even more important when we decide how to address the missingness in these variables. 

### Continuous variables
Next, we can look at the distributions of the half-dozen continuous variables in our data set. However, three continuous variables (id, lat, and long) are features that likely don't have informative distributions, so we won't plot them. Histograms are a classic plot for quickly plotting the distribution of continuous variables. Density plots **plot_density()** also work well for visualizing continuous variables.

```r
used_cars3 %>% 
   select(odometer, price, year) %>%
ExpNumStat(by = "A", round = 2)
```

```
##      Vname Group     TN nNeg nZero   nPos NegInf PosInf NA_Value Per_of_Missing
## 1 odometer   All 409231    0  1660 403420      0      0     4151           1.01
## 2    price   All 409231    0 31434 377797      0      0        0           0.00
## 3     year   All 409231    0     0 409225      0      0        6           0.00
##           sum  min        max     mean  median          SD     CV      IQR
## 1 39200864352    0   10000000 96773.14 85879.5   200545.31   2.07 95180.25
## 2 31509426900    0 3736928711 76996.68 13988.0 12439203.34 161.56 20020.00
## 3   823159582 1900       2022  2011.51  2014.0        8.94   0.00     9.00
##   Skewness Kurtosis
## 1    40.49  1920.28
## 2   249.27 66406.81
## 3    -3.61    20.91
```

```r
used_cars3 %>%
  select(odometer, price, year) %>%
plot_histogram()
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-9-1.png" width="1440" />

```r
used_cars3 %>%
  select(odometer, price, year) %>%
plot_density()
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-9-2.png" width="1440" />
Hmm... Both the odometer and price seem to contain some severe outliers that we are going to have to investigate further. From the looks of our histogram above, there are many cars with a price of zero dollars and some vehicles with a suspiciously expensive price listed.


```r
# Looking at the cheapest priced cars
used_cars3 %>%
  filter(price == 0) %>%
  nrow()
```

```
## [1] 31434
```

```r
# Looking at the top 30 priciest cars
used_cars3 %>%
  arrange(desc(price)) %>%
  select(price) %>%
  head(30)
```

```
## # A tibble: 30 × 1
##         price
##         <dbl>
##  1 3736928711
##  2 3736928711
##  3 3024942282
##  4 3024942282
##  5 3009548743
##  6 1410065407
##  7 1234567890
##  8 1111111111
##  9 1111111111
## 10  987654321
## # ℹ 20 more rows
```
There are over 30 thousand cars with a price listing of zero dollars and dozens of vehicles with dollar amounts so high that they seem improbable. 

Looking at the odometer variable, we see only a few thousand listings with a mileage of zero and about the same amount with an odometer listing of over 400,000

```r
# Looking at the cars with the least miles on the odometer
used_cars3 %>%
  filter(odometer == 0) %>%
  nrow()
```

```
## [1] 1660
```

```r
# Looking at the cars with the most miles on the odometer
used_cars3 %>%
  arrange(desc(odometer)) %>%
  filter(odometer > 400000) %>%
  nrow()
```

```
## [1] 1015
```
Based on what we have discovered so far, it might be worth considering what we hope to get out of this data. Perhaps your goal is to develop a model that predicts used car prices on Craigslist, so you can price your own car online or compare the price of a vehicle you are considering buying to other similar listings.

We would likely not need to include the most expensive cars in this data to do this. Additionally, we could remove the vehicles priced at zero dollars. Maybe we don't want our model to consider a car unless it is priced above 1,000 dollars. 

There likely are cars sold on Craigslist with zero miles, and the number of those cars is minimal, so we can keep those low-mileage vehicles. However, cars with high mileage may not be desirable to you. The magazine and website [Car and Driver](https://www.caranddriver.com/research/a32758625/how-many-miles-does-a-car-last/#) states that standard cars are only expected to last about 200,000 miles. Therefore, we will limit this data to vehicles with 200,000 miles or less.

Whenever you consider removing extreme values (outliers) in a data set, it is important to perform your analysis with and without those unusual values. This is a form of [sensitivity analysis](https://en.wikipedia.org/wiki/Sensitivity_analysis), a handy tool for dealing with uncertainty. If the effects of dropping your extreme values are minimal and the values don't make sense, it may be okay to remove them. However, if their impact is substantial, it is crucial to justify and document (as we did above) before removing any observations.


```r
#library(flextable)
# Filtering out price outliers and high mileage vehicles
used_cars4 <- used_cars3 %>%
  filter(price > 1000,
         price <= 349999, 
          odometer <= 200000)

# ExpNumStat(used_cars4, by = "A", round = 2) %>%
#   flextable::flextable()

# used_cars4 %>%
#   select(odometer, price, year) %>%
#   plot_histogram()
# 
# used_cars4 %>%
#   select(odometer, price, year) %>%
#   plot_qq()
```

### Bivariate Analysis
The next thing to investigate is the relationships between variables (covariation). Again, different strategies are used to explore the relationships between different types of variables. For example, you may want to see how two continuous variables change with respect to one other, or you may want to look at the distribution of a continuous variable broken down by a particular categorical variable. Lastly, you can visualize the covariation between two categorical variables.

So far, we've utilized the [DataExplorer package](https://cran.r-project.org/web/packages/DataExplorer/vignettes/dataexplorer-intro.html) to perform many of our visualizations. DataExplorer is excellent, but it can't always be tailored to specific questions you might have about your data. The ggplot2 package is the workhorse package for data visualizations in R. ggplot2 can be customized to seemingly no end, making it a powerful tool for conveying meaning from your data. However, ggplot2's flexibility comes at the cost of a steep learning curve. Luckily, using ggplot2 for EDA is a great way to build your data visualization skills in R.

Here are a few examples of bivariate EDA with ggplot2. 

### Continues and Categorical Variable
Here is a simple boxplot with ggplot2 looking at the price of cars stratified by the listed condition.

```r
ggplot(data = used_cars4, mapping = aes(x = condition, 
                                        y = price, 
                                        group = condition)) +
  geom_boxplot()
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-13-1.png" width="1440" />
This boxplot isn't much to look at, and that limits what we can take away from the data. Let us expand on this boxplot to see if we can make something more useful. The things we will change from the first boxplot are

- Flip the x and y axes.
- Distinguish the variable groupings with color.
- Plot the y-axis on the log scale to reduce the effect of this highly skewed data.
- Reorder the condition variable into descending values of price.
- Increase the size (of the lines) of the boxplots themselves.
- Add a more descriptive y-axis (the x-axis is self-explanatory here)
- Overlay a [*ggbeeswarm package*](https://cran.r-project.org/web/packages/ggbeeswarm/index.html) layer 


```r
library(ggbeeswarm)
ggplot(data = used_cars4, mapping = aes(y = fct_reorder(condition, desc(price)), 
                                        x = price, 
                                        fill = condition, 
                                        color = condition)) +
  geom_boxplot(alpha = 0.2, size = 1.5, 
               show.legend = FALSE, 
               outlier.colour = NA) +
  scale_x_log10(labels = scales::dollar_format(prefix = "$")) +
  geom_quasirandom(data = sample_frac(tbl = used_cars4, size = 0.10), 
                   alpha = 0.4, show.legend = FALSE) +
  labs(y = NULL, x = "Price of Car in Dollars ($)") + 
  theme_bw()
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-14-1.png" width="1440" />
This boxplot obviously looks much different. The color helps convey that these are distinct groups of data. The log scale allows us to compare differences between these groups, and the ordered x-axis, along with the increased plot size and descriptive axis labels, makes this much easier to digest. Lastly, I added a **geom_quasirandom()** layer from the [*ggbeeswarm package*](https://cran.r-project.org/web/packages/ggbeeswarm/index.html). The **geom_quasirandom()** layer is usually better with data that has a smaller number of observations (n < 500). Still, I sometimes take a random sample for larger data to understand each category's distribution and the relative number of observations within each group. The ordered x-axis and the increased plot and descriptive axis labels make this much easier to digest.

### Two Continues Variables
Next, we will explore the covariation of the two continuous variables **odometer** and **price.** To do this, we will use the ggplot2 **geom_point()** layer, and because this is such a large data set to visualize, we will plot a fraction of the data using **sample_frac()**.

```r
used_cars4 %>% sample_frac(size = 0.05) %>%
  ggplot(aes(x = odometer, y = price)) + 
  geom_point (color = "steel blue", 
              size = 2, 
              alpha = 0.7,
              na.rm = TRUE) +
  scale_y_continuous(labels = scales::dollar_format(prefix = "$")) +
  labs(y = "Price of Car in Dollars ($)", x = "Miles on Odometer") + 
  theme_bw()
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-15-1.png" width="1440" />
From the plot above, we can see a gradual decline in car price as the number of miles on the odometer increase.

### Two Categorical Variables
Lastly, we will explore how two categorical variables change with respect to one another. To do this, we need counts of each category in either variable. Luckily the ggplot2 geom layer geom_count() can help us do this.

```r
used_cars4 %>%
  group_by(manufacturer) %>%
  mutate(manu_count = n()) %>%
  filter(manu_count > 10000) %>%
  ggplot() +
  geom_count(mapping = aes(x = manufacturer, 
                           y = paint_color,
                           na.rm = TRUE))
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-16-1.png" width="1440" />
We can see from the plot above that for the top represented manufacturers in this data set, the most common car colors for all manufacturers are white, black, silver, red, blue, and grey.

## Context-Specific EDA
Often we are working with data where we are interested in how one variable (our outcome) is influenced by all others (our predictors). With this in mind, it is common to ask specific questions and create visualizations that provide a better understanding of the predictor's effect on our outcome. We can incorporate more than two variables and create new features from the data to include in our visualizations.

Price is an obvious potential outcome in this data set. When doing EDA with a specific outcome in mind, it often useful to ask; what kind of predictor variables do I have? 

- Do you have a time variable? Can you trend your outcome over time? 
- Do you have place-based data? Can you look at your outcome geographically?
- If you are working with data collected on people, can you stratify by variables like age, sex, and other demographics that might be insightful?

We don't have much in the way of demographic data since we're working with cars. However, we can explore price more with some of the most descriptive variables in the data set.

```r
used_cars4 %>% 
  group_by(manufacturer) %>%
  mutate(manu_count = n()) %>%
  filter(manu_count > 10000) %>%
  ggplot(aes(x = price, y = fct_reorder(manufacturer, price)), 
         group = manufacturer,
         color = manufacturer) + 
  geom_boxplot(na.rm = TRUE, 
               show.legend = FALSE) +
  #scale_x_log10(labels = scales::dollar_format(prefix = "$")) +
  labs(y = NULL, x = "Price of Car in Dollars ($)") +
  theme_bw() + 
  ggtitle("The Average Car Asking Price by Manufacturer and the Number of Years Old")
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-17-1.png" width="1440" />
In the plot above, we have taken the ten most commonly listed manufacturers in the data set and plotted their price distribution. We can see that Honda and *Nissan* cars have the smallest pricing variation. Meanwhile, manufacturers like *Ford*, *GMC*, and *Cherolet* have the most significant variation in listing prices.

We also don't have an obvious way to see how price changes over time because we only have one year (2021) of data. However, we do have the year each car was manufactured!

```r
# used_cars4 %>%
#   select_if(function(var) is.factor(var) |
#               all(var == used_cars4$price) |
#               all(var == used_cars4$year)) %>%
#      select(-region, -model, -VIN, -title_status, -state)
#   plot_boxplot(by = c("condition", "cylinders", "fuel", "title_status", "transmission", "drive"))

# 
# library(ggbeeswarm)
# ggplot(data = used_cars4, mapping = aes(y = fct_reorder(condition, desc(price)), 
#                                         x = price, 
#                                         fill = condition, 
#                                         color = condition)) +
#   geom_boxplot(alpha = 0.2, size = 1.5, 
#                show.legend = FALSE, 
#                outlier.colour = NA) +
#   scale_x_log10(labels = scales::dollar_format(prefix = "$")) +
#   geom_quasirandom(data = sample_frac(tbl = used_cars4, size = 0.10), 
#                    alpha = 0.4, show.legend = FALSE) +
#   labs(y = NULL, x = "Price of Car in Dollars ($)") + 
#   theme_bw()
  
used_cars4 %>% 
  mutate(years_old = 2021 - year) %>% 
  filter(year > 1990 & year < 2020) %>%
  group_by(manufacturer) %>%
  mutate(manu_count = n()) %>%
  ungroup() %>%
  group_by(manufacturer, year) %>%
  mutate(manu_yr_price_avg = mean(price)) %>%
  filter(manu_count > 10000) %>%
  ggplot(aes(x = years_old, y = manu_yr_price_avg, group = manufacturer,
                              color = manufacturer)) + 
  geom_line(size = 2, 
            alpha = 0.7,
            na.rm = TRUE, 
            show.legend = TRUE) +
  scale_color_brewer(palette="Paired") +
  scale_y_continuous(labels = scales::dollar_format(prefix = "$")) +
  labs(y = "Average Price of Car in Dollars ($)", x = "Year") + 
  theme_bw() + 
  ggtitle("The Average Car Asking Price by Manufacturer and the Number of Years Old")
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-18-1.png" width="1440" />
In the graph above, we have plotted the average yearly car price in this data set by the age of the vehicle (in years), stratified by the 10 most frequently represented manufacturers. From this plot, we can see a spike in asking price for Toyota's from the early 90s and a relatively high (and steady overtime) asking price for Dodge Ram vehicles until they reach ~20 years old. 

Next, it may be interesting to investigate geographic differences since we have data from all 50 states. To do this, we will use box plots again to display the distribution of car prices in each state.

```r
ggplot(data = used_cars4, aes(x = price, y =  fct_reorder(state, price))) +
                              #color = fct_reorder(state, price))) + 
  geom_boxplot(na.rm = TRUE, show.legend = FALSE) +
  scale_x_log10(labels = scales::dollar_format(prefix = "$")) +
  labs(y = NULL, x = "Price of Car in Dollars ($)") 
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-19-1.png" width="1440" />
Here we see the price distribution for all cars stratified by all fifty states. Interestingly, West Virginia has the highest median listing price, and Delaware has the lowest. However, this level of stratification may be too much to make valuable inferences. Let's see if we can find a more appropriate geographic stratification by adding to this data set.

Fortunately, R has inherent data sets that include information on all 50 states in the U.S. We can join these U.S states data sets with our Craigslist data set with the state abbreviation variables that exist in both. We will also do some feature engineering and create a new category within the country region variable because it seems inaccurate to classify Alaska and Hawaii simply *"West."*


```r
states <- tibble(state.abb, state.region) %>%
  mutate(state.region = as.character(state.region)) %>%
  mutate(state.region = ifelse(state.abb == "AK" |
                                 state.abb == "HI", 
                               "AK/HI", state.region)) %>%
  rename("state" = state.abb, 
         "country_region" = state.region)

used_cars5 <- used_cars4 %>%
  mutate(state = toupper(state)) %>%
  left_join(states, by = "state")
```

Now that we have added U.S. regions to our data set, we can look at regional price differences.

```r
ggplot(data = used_cars5, aes(x = price, y =  fct_reorder(country_region, price))) +
                              #color = fct_reorder(state, price))) + 
  geom_boxplot(na.rm = TRUE, show.legend = FALSE) +
  scale_x_log10(labels = scales::dollar_format(prefix = "$")) +
  labs(y = NULL, x = "Price of Car in Dollars ($)") 
```

<img src="/blog/EDA in R/eda_in_R_files/figure-html/unnamed-chunk-21-1.png" width="1440" />
The plot above shows that the lowest median listed vehicle price is in the Northeast region. In addition, our decision to create a new category for Alaska and Hawaii also seems validated, as these states have the highest listing price.


## Conclusion
![](https://img.freepik.com/premium-vector/filtering-icon-with-funnel-flat-vector_116137-5693.jpg)

This has been an example of how an EDA process might progress with a given data set. Unfortunately, there is no template on how to approach EDA. Instead, you should start exploring a new data set with a funnel-shaped framework where you begin by asking and investigating broad questions about the data, then iteratively progress to more specific and complex questions.

Even if you are given a data set with a specific task, EDA can be a valuable tool to uncover specifics in data quality or relationships between previously unknown variables. 

The EDA process is just the beginning. We might have to revisit this data set to explore other concepts of data analysis. Thank you for reading to the end, and good luck navigating your next data set!


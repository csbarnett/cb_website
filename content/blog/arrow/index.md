---
title: "Working With Large Data in R With Apache Arrow and DuckDB"
author: "Chris Barnett"
date: "2023-08-31"
excerpt: Some times we have data files that are larger than R can store in memory. Apache Arrow and DuckDB allow the user to manipulate and query these files while they remain on disk, much like working with an SQL database.
---


![](https://images.unsplash.com/photo-1523274620588-4c03146581a1?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1470&q=80)%20Big%20data%20is%20watching%20you:%20Free%20to%20use%20under%20the%20[Unsplash%20License](https://unsplash.com/license)


## TLDR
Some times we have data files that are larger than R can store in memory. Apache Arrow and DuckDB allow the user to manipulate and query these files while they remain on disk, much like working with an SQL database.

## Environment

```r
knitr::opts_chunk$set(echo = TRUE)
options(scipen=999) 

library(tidyverse)
library(duckdb)
library(arrow)
library(janitor)
library(tictoc)
```

## Data

![](https://images.unsplash.com/photo-1532938911079-1b06ac7ceec7?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1632&q=80) Doctor with a stethoscope: Free to use under the [Unsplash License](https://unsplash.com/license)

We will manipulate the [Doctors and clinicians data archive](https://data.cms.gov/provider-data/archived-data/doctors-clinicians) from the Centers for Medicare & Medicaid Services. More specifically, we will be using the [National Downloadable File](https://data.cms.gov/provider-data/dataset/mj5m-pzi6), which lists all U.S physicians enrolled in Medicare and information on their enrollment group, practice, or hospital. 


## Big Data
Analytic tools like R and Python are designed to analyze data in memory, or RAM (random access memory). Working with data in memory has the advantage of fast and seamless modification and copying of data. However, modifying and copying large amounts of data can quickly outpace your machine's memory. Big Data has many definitions, but when the capacity of your computer is the limiting factor, big data is anything you can't process in memory.


```r
csv_files <- list.files("/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac/", 
                        recursive=TRUE, 
                        full.names = TRUE)
data.frame(file = "CSV Files", 
  Megabytes_MB = (file.size(csv_files) / 1024^2) %>% 
             round(digits = 2))
```

```
##        file Megabytes_MB
## 1 CSV Files       635.99
## 2 CSV Files       638.01
## 3 CSV Files       638.07
## 4 CSV Files       640.10
## 5 CSV Files       637.39
## 6 CSV Files       636.60
```

## Arrow

![](https://arrow.apache.org/img/arrow-logo_chevrons_black-txt_white-bg.svg)


[Apache Arrow](https://arrow.apache.org/) is a software platform for processing and transporting large data sets. [The arrow package](https://arrow.apache.org/docs/r/index.html) provides a standard way to use Apache Arrow in R.

Instead of bringing large data sets onto your machine's RAM, *arrow* can scan a folder and "connect" or "point" to the data set without loading it into memory. We can do this using **open_dataset()**:

It often helps to divide large data sets into obvious segments so you only download and store the data you need. The Doctors and Clinicians data is divided into years and months. Fortunately, *arrow* can quickly connect all these disparate data sets into one *arrow* object with the *partitioning* argument.

```r
# Pointing to the csv data with arrow
  arrow_files <- open_dataset("/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac/", 
                         format = "csv",
                         partitioning = c("year", "month"))
arrow_files
```

```
## FileSystemDataset with 6 csv files
## NPI: int64
## Ind_PAC_ID: int64
## Ind_enrl_ID: string
## lst_nm: string
## frst_nm: string
## mid_nm: string
## suff: string
## gndr: string
## Cred: string
## Med_sch: string
## Grd_yr: int64
## pri_spec: string
## sec_spec_1: string
## sec_spec_2: string
## sec_spec_3: string
## sec_spec_4: null
## sec_spec_all: string
## Telehlth: string
## org_nm: string
## org_pac_id: int64
## num_org_mem: int64
## adr_ln_1: string
## adr_ln_2: string
## ln_2_sprs: string
## cty: string
## st: string
## zip: int64
## phn_numbr: int64
## ind_assgn: string
## grp_assgn: string
## adrs_id: string
## year: int32
## month: int32
```
Here, we can see we have combined six files into one arrow object with 32 variables, including year and month variables that *arrow* has added.

We can also see that *arrow* detected only empty values in the first 1000 rows of *sec_spec_4* variable and has set the data type to **NULL**. We can perform simple data manipulations with *arrow*, including removing empty variables. 

```r
# Subsetting the  dataset
arrow_files_2 <- arrow_files |>
  select(-sec_spec_4)



glimpse(arrow_files_2)
```

```
## FileSystemDataset with 6 csv files (query)
## 14,852,597 rows x 32 columns
## $ NPI           <int64> 1003000126, 1003000126, 1003000126, 1003000126, 10030001…
## $ Ind_PAC_ID    <int64> 7517003643, 7517003643, 7517003643, 7517003643, 75170036…
## $ Ind_enrl_ID  <string> "I20130530000085", "I20130530000085", "I20130530000085",…
## $ lst_nm       <string> "ENKESHAFI", "ENKESHAFI", "ENKESHAFI", "ENKESHAFI", "ENK…
## $ frst_nm      <string> "ARDALAN", "ARDALAN", "ARDALAN", "ARDALAN", "ARDALAN", "…
## $ mid_nm       <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "L", "L", "L",…
## $ suff         <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ gndr         <string> "M", "M", "M", "M", "M", "M", "M", "M", "M", "M", "M", "…
## $ Cred         <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ Med_sch      <string> "OTHER", "OTHER", "OTHER", "OTHER", "OTHER", "OTHER", "O…
## $ Grd_yr        <int64> 1994, 1994, 1994, 1994, 1994, 1994, 1994, 2003, 2003, 20…
## $ pri_spec     <string> "INTERNAL MEDICINE", "INTERNAL MEDICINE", "INTERNAL MEDI…
## $ sec_spec_1   <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ sec_spec_2   <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ sec_spec_3   <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ sec_spec_all <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ Telehlth     <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "NA", "N…
## $ org_nm       <string> "ADFINITAS HEALTH AT UMCAP LLC", "EMERGENCY MEDICINE ASS…
## $ org_pac_id    <int64> 840677738, 8022914522, 8022914522, 8325943707, 832594370…
## $ num_org_mem   <int64> 44, 361, 361, 514, 514, 83, 361, 1427, 1427, 1427, 1427,…
## $ adr_ln_1     <string> "901 HARRY S TRUMAN DR", "1850 TOWN CTR PKWY", "1701 N G…
## $ adr_ln_2     <string> "NA", "NA", "NA", "SUITE 102", "NA", "NA", "NA", "NA", "…
## $ ln_2_sprs    <string> "NA", "NA", "NA", "NA", "NA", "NA", "NA", "Y", "Y", "NA"…
## $ cty          <string> "LARGO", "RESTON", "ARLINGTON", "LAUREL", "BETHESDA", "F…
## $ st           <string> "MD", "VA", "VA", "MD", "MD", "VA", "VA", "IL", "IL", "I…
## $ zip           <int64> 207745477, 201903204, 222053610, 207075255, 208141422, 2…
## $ phn_numbr     <int64> 2406771000, 2406862300, 7035586161, 3016045254, NA, 5407…
## $ ind_assgn    <string> "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "…
## $ grp_assgn    <string> "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "Y", "…
## $ adrs_id      <string> "MD207745477LA901XXDRXX500", "VA201903204RE1850XPKWY400"…
## $ year          <int32> 2023, 2023, 2023, 2023, 2023, 2023, 2023, 2023, 2023, 20…
## $ month         <int32> 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4,…
## Call `print()` for query details
```
What is the size of this large (~15 million rows) arrow object?

```r
object.size(arrow_files_2)
```

```
## 18224 bytes
```
1 MB = 1,024,000 bytes

## Parquet Files

![](https://images.unsplash.com/photo-1629976828074-c248d94c82ea?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1470&q=80) Jakarta Parquet: Free to use under the [Unsplash License](https://unsplash.com/license)

Data manipulation in *arrow* is somewhat limited, so what do you do if you need to perform more in-depth analysis? For large datasets, CSV files become difficult to store and work with. Luckily, *arrow* allows quick and easy data format transformations. Parquet files are a column-oriented data format for efficient storage, fast reading, and writing. 

The main benefits of Parquet files are: 
 - Their efficient storage.
 - They can handle seemingly any data type.
 - Their speed and ease of use with big data.
 
 
We can create a new folder and populate it with transformed parquet files using *arrow*. 

```r
# Creating a home for converted parquet files amd writing 
if(!dir.exists("/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac_pq/converted_parquet")) {
  
  dir.create("/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac_pq/converted_parquet")
  
  # this reads each csv file in the csv_ds dataset and converts it to 
  # a .parquet file
  write_dataset(arrow_files_2, 
                path = "/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac_pq/converted_parquet", 
                format = "parquet",
                partitioning = c("year", "month"))
}
```


How do the parquet file sizes compare?

```r
pq_files <- list.files("/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac_pq/converted_parquet", 
                        recursive=TRUE, 
                        full.names = TRUE)
data.frame(file = "Parquet Files", 
  size_Mb = (file.size(pq_files) / 1024^2) %>% 
             round(digits = 2))
```

```
##            file size_Mb
## 1 Parquet Files  220.16
## 2 Parquet Files  220.78
## 3 Parquet Files  220.98
## 4 Parquet Files  222.07
## 5 Parquet Files  221.31
## 6 Parquet Files  221.09
```
In this example, our parquet files are about a third the size of the CSV files.

We can directly read in these smaller parquet data sets into memory with *arrow* and the **read_parquet()** function.

```r
# Reading in March 2023 Doctors and clinicians data
dac_pq_2023_03 <- arrow::read_parquet(file = "/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac_pq/converted_parquet/year=2023/month=3/part-0.parquet")
(as.character(object.size(dac_pq_2023_03)) %>% parse_number() / 1024^2) %>% 
             round(digits = 2) %>% paste("MB")
```

```
## [1] "684.55 MB"
```


Or, we can "point" arrow at these newly created parquet files the same way we accessed the CSV data.

```r
# Pointing to the parquet data with arrow
  arrow_pq_files <- open_dataset("/Users/emmacoleman/Library/Mobile Documents/com~apple~CloudDocs/Documents/R Projects/cb_website/data/dac_pq/converted_parquet", 
                         format = "parquet",
                         partitioning = c("year", "month"))
arrow_pq_files
```

```
## FileSystemDataset with 6 Parquet files
## NPI: int64
## Ind_PAC_ID: int64
## Ind_enrl_ID: string
## lst_nm: string
## frst_nm: string
## mid_nm: string
## suff: string
## gndr: string
## Cred: string
## Med_sch: string
## Grd_yr: int64
## pri_spec: string
## sec_spec_1: string
## sec_spec_2: string
## sec_spec_3: string
## sec_spec_all: string
## Telehlth: string
## org_nm: string
## org_pac_id: int64
## num_org_mem: int64
## adr_ln_1: string
## adr_ln_2: string
## ln_2_sprs: string
## cty: string
## st: string
## zip: int64
## phn_numbr: int64
## ind_assgn: string
## grp_assgn: string
## adrs_id: string
## year: int32
## month: int32
```

But if manipulating data with *arrow* is limited, how can we analyze all these data sets without bringing them into memory?


## DuckDB

![](https://miro.medium.com/v2/resize:fit:1112/0*rPqrr9iDtilrIbb0.png)

[*DuckDB*](https://duckdb.org/) is an analytical data management system that can run SQL, R, or Python. *DuckDB* can query *arrow* datasets directly and stream query results back to *arrow*. This allows us to query *arrow* data using *DuckDB's* interface, without extra data copying.

The main advantages of *DuckDB* are 
 - It has full SQL, R, and Python support, 
 - It allows for data manipulation without bringing it into memory or creating extra copies of the data itself.
 
Therefore, you can use *arrow* for reading and writing data fast and efficiently. While *arrow* has some built-in analysis capabilities, *DuckDB* is a database engine with more complete data manipulation and aggregation capabilities.

We can couple the [duckdb package](https://cran.r-project.org/web/packages/duckdb/index.html) with the [DBI package](https://cran.r-project.org/web/packages/DBI/index.html) to create a connection object that we can use to query the *arrow* object.


```r
library(duckdb)
library(DBI)

con <- DBI::dbConnect(duckdb())

pq_db <- to_duckdb(.data = arrow_pq_files, 
                   con = con, 
                   table_name = "arrow_pq")
```

The duckdb "**database**" can be queried using SQL.


```r
# Querying all Alaska phyisicans in the data
ak_docs <- DBI::dbSendQuery(con, "SELECT NPI, frst_nm, lst_nm, org_nm, st
                                  FROM arrow_pq
                                  WHERE st = 'AK'
                                  LIMIT 20;")
DBI::dbFetch(ak_docs)
```

```
##           NPI  frst_nm    lst_nm                                    org_nm st
## 1  1003156738  HILLARY   MONSOUR           NORTH POLE PHYSICAL THERAPY INC AK
## 2  1003159021  MATTHEW    DAWSON PROVIDENCE HEALTH AND SERVICES WASHINGTON AK
## 3  1003173824   ANDREA CABALLERO                                        NA AK
## 4  1003000753    JAMES    DEITLE             EASTERN ALEUTIAN TRIBES, INC. AK
## 5  1003000753    JAMES    DEITLE             EASTERN ALEUTIAN TRIBES, INC. AK
## 6  1003064999  DOUGLAS    LARSON                    IMAGING ASSOCIATES LLC AK
## 7  1003070780    JERRY     FLYNN                                        NA AK
## 8  1003070780    JERRY     FLYNN                            CITY OF SEWARD AK
## 9  1003073560      JOE    LLENOS              SOUTH PENINSULA HOSPITAL INC AK
## 10 1003075292 FERNANDO     TOVAR             ANESTHESIA CARE ASSOCIATES PC AK
## 11 1003076571      AMY     SAUCK      SELECT PHYSICAL THERAPY HOLDINGS INC AK
## 12 1003076571      AMY     SAUCK      SELECT PHYSICAL THERAPY HOLDINGS INC AK
## 13 1003085218   CECILY  REYNOLDS      ALASKA EMERGENCY MEDICINE ASSOCIATES AK
## 14 1003260167  THERESA   CHIHULY    CENTRAL PENINSULA GENERAL HOSPITAL INC AK
## 15 1003264029    LOGAN      HUFF           NORTON SOUND HEALTH CORPORATION AK
## 16 1003285222 CHRISTIE     BENTZ                BARTLETT REGIONAL HOSPITAL AK
## 17 1003285222 CHRISTIE     BENTZ                BARTLETT REGIONAL HOSPITAL AK
## 18 1003301755    PARIS    TAYLOR             MAT - SU HEALTH SERVICES INC. AK
## 19 1003313222    DEVON   KIENZLE       BRISTOL BAY AREA HEALTH CORPORATION AK
## 20 1003313222    DEVON   KIENZLE       BRISTOL BAY AREA HEALTH CORPORATION AK
```

However, if you don't know SQL, the *DuckDB* "**database**" can be queried tidyverse style verbs using [dplyr package](https://dplyr.tidyverse.org/).

```r
# Querying all Alaska phyisicans in the data
ak_docs_dplyr <- pq_db %>%
  select(NPI, frst_nm, lst_nm, org_nm, st) %>%
  filter(st == "AK")

glimpse(ak_docs_dplyr)
```

```
## Rows: ??
## Columns: 5
## Database: DuckDB 0.8.1 [root@Darwin 19.6.0:R 4.1.0/:memory:]
## $ NPI     <dbl> 1003156738, 1003159021, 1003173824, 1003000753, 1003000753, 10…
## $ frst_nm <chr> "HILLARY", "MATTHEW", "ANDREA", "JAMES", "JAMES", "DOUGLAS", "…
## $ lst_nm  <chr> "MONSOUR", "DAWSON", "CABALLERO", "DEITLE", "DEITLE", "LARSON"…
## $ org_nm  <chr> "NORTH POLE PHYSICAL THERAPY INC", "PROVIDENCE HEALTH AND SERV…
## $ st      <chr> "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK", "AK", "A…
```




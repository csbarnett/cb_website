---
title: "Himalayan climbing expeditions"
author: "Chris Barnett"
date: "2023-06-29"
excerpt: "The Himalayan Database is a compilation of mountaineering climbing records for all expeditions in the Himalayas of Nepal, started by American journalist Elizabeth Hawley. The database has been supplemented by information from books, alpine journals, and correspondence with Himalayan climbers. In this post, we explore the database and ask, what are the most challenging peaks to climb in the Himalayas? Also, we see if we can develop a simple model to predict a climber's success in summiting."
output: html_document
---

![](https://www.forbesindia.com/media/images/2022/Oct/img_195311_himalaya.jpg)


I came across [this article](https://www.nytimes.com/2018/01/26/obituaries/elizabeth-hawley-who-chronicled-everest-treks-dies-at-94.html) about the passing of journalist Elizabeth Hawley who chronicled Himalayan mountaineering expeditions for more than 50 years and was one of the founders of the Himalayan Database. The [Himalayan Database](https://www.himalayandatabase.com) is a compilation of mountaineering climbing records for all expeditions in the Himalayas of Nepal. The database has been supplemented by information from books, alpine journals, and correspondence with Himalayan climbers. Elizabeth Hawley took the accuracy of the database seriously. She would often interview mountaineers (more than 15,000 interviews) before and after summit attempts and ask about specific details to gauge authenticity. Elizabeth Hawley died in January 2018 in a small hospital in Nepal. She was 94.

The data includes expeditions from 1905 to (currently) Spring 2022. Expeditions on peaks that span the border of Nepal and China, such as Everest, Cho Oyu, Makalu, and Kangchenjunga, and some smaller border peaks are also included. Each expedition record contains detailed information, including dates, routes, camps, use of supplemental oxygen, successes, deaths, and accidents. Each expedition record includes biographical information for all members listed on the permit and for hired members (e.g., Sherpas) for which there are significant events such as a summit success, death, accident, or rescue. We will explore the database and see if we can develop a simple model to predict a climber's success in summiting.



## EDA
What are the hardest peaks to climb in the Himalayas? To find out, we will sort each peak by the **success rate** (the number of ascents divided by the number of attempts). We will also restrict the data to peaks with at least 100 attempts to summit.

```
## # A tibble: 51 × 4
## # Groups:   peak_name [51]
##    peak_name         ascents attempts success_rate
##    <chr>               <int>    <int>        <dbl>
##  1 Peak 29                 2      104         1.92
##  2 Gaurishankar            6      201         2.99
##  3 Nemjung                 8      133         6.02
##  4 Nuptse                 20      303         6.6 
##  5 Annapurna II           16      224         7.14
##  6 Lhotse Shar            24      276         8.7 
##  7 Dhaulagiri IV          13      129        10.1 
##  8 Himalchuli East        27      246        11.0 
##  9 Annapurna III          31      272        11.4 
## 10 Churen Himal West      14      103        13.6 
## # ℹ 41 more rows
```

```
## # A tibble: 2 × 5
## # Groups:   oxygen_used [2]
##   oxygen_used success     n attempts success_rate
##   <lgl>       <lgl>   <int>    <int>        <dbl>
## 1 FALSE       TRUE    15054    58271         25.8
## 2 TRUE        TRUE    14131    18233         77.5
```

```
## # A tibble: 2 × 5
## # Groups:   solo [2]
##   solo  success     n attempts success_rate
##   <lgl> <lgl>   <int>    <int>        <dbl>
## 1 FALSE TRUE    29108    76383         38.1
## 2 TRUE  TRUE       77      121         63.6
```
![](https://www.caingram.info/Nepalpeaks/Pix/Peak_29.jpg)

Peak 29 or "Ngadi Chuli"


```r
# Filtering himal_df by only peaks with 100 or more attempts
himal_100 <- himal %>%
  count(peak_id, success) %>%
  group_by(peak_id) %>%
  mutate(attempts = sum(n)) %>%
  filter(success == TRUE) %>%
  filter(attempts > 100)

himal_100 %>% distinct(peak_id)
```

```
## # A tibble: 51 × 1
## # Groups:   peak_id [51]
##    peak_id
##    <chr>  
##  1 AMAD   
##  2 ANN1   
##  3 ANN2   
##  4 ANN3   
##  5 ANN4   
##  6 ANNS   
##  7 APIM   
##  8 BARU   
##  9 BHRI   
## 10 CHAM   
## # ℹ 41 more rows
```

The filter join **semi_join()** of himal using himal_100 gives us only himal observations with peaks that are present in himal_100.


```r
himal_100_df <- himal %>%
  semi_join(himal_100, by = "peak_id")

himal_100_df %>% distinct(peak_id)
```

```
## # A tibble: 51 × 1
##    peak_id
##    <chr>  
##  1 AMAD   
##  2 ANN1   
##  3 ANN2   
##  4 ANN3   
##  5 ANN4   
##  6 ANNS   
##  7 APIM   
##  8 BARU   
##  9 BHRI   
## 10 CHAM   
## # ℹ 41 more rows
```
Taking a quick look at the himal_100_df data with the *skimr package*

```r
skimr::skim(himal_100_df)
```


Table: Table 1: Data summary

|                         |             |
|:------------------------|:------------|
|Name                     |himal_100_df |
|Number of rows           |68436        |
|Number of columns        |21           |
|_______________________  |             |
|Column type frequency:   |             |
|character                |10           |
|logical                  |6            |
|numeric                  |5            |
|________________________ |             |
|Group variables          |None         |


**Variable type: character**

|skim_variable   | n_missing| complete_rate| min| max| empty| n_unique| whitespace|
|:---------------|---------:|-------------:|---:|---:|-----:|--------:|----------:|
|expedition_id   |         0|          1.00|   9|   9|     0|     9019|          0|
|member_id       |         0|          1.00|  12|  12|     0|    68435|          0|
|peak_id         |         0|          1.00|   4|   4|     0|       51|          0|
|peak_name       |         0|          1.00|   5|  17|     0|       51|          0|
|season          |         0|          1.00|   6|   6|     0|        4|          0|
|sex             |         0|          1.00|   1|   1|     0|        2|          0|
|citizenship     |         3|          1.00|   2|  23|     0|      210|          0|
|expedition_role |        14|          1.00|   4|  25|     0|      486|          0|
|death_cause     |     67429|          0.01|   3|  27|     0|       12|          0|
|injury_type     |     66832|          0.02|   3|  27|     0|       11|          0|


**Variable type: logical**

|skim_variable | n_missing| complete_rate| mean|count                  |
|:-------------|---------:|-------------:|----:|:----------------------|
|hired         |         0|             1| 0.21|FAL: 53813, TRU: 14623 |
|success       |         0|             1| 0.39|FAL: 41957, TRU: 26479 |
|solo          |         0|             1| 0.00|FAL: 68334, TRU: 102   |
|oxygen_used   |         0|             1| 0.26|FAL: 50335, TRU: 18101 |
|died          |         0|             1| 0.01|FAL: 67429, TRU: 1007  |
|injured       |         0|             1| 0.02|FAL: 66831, TRU: 1605  |


**Variable type: numeric**

|skim_variable        | n_missing| complete_rate|    mean|      sd|   p0|  p25|  p50|    p75| p100|hist  |
|:--------------------|---------:|-------------:|-------:|-------:|----:|----:|----:|------:|----:|:-----|
|year                 |         0|          1.00| 2001.34|   13.83| 1905| 1993| 2005| 2012.0| 2019|▁▁▁▃▇ |
|age                  |      2513|          0.96|   37.34|   10.18|    7|   30|   36|   44.0|   85|▁▇▅▁▁ |
|highpoint_metres     |     18704|          0.73| 7578.45| 1009.08| 4000| 6814| 7678| 8500.0| 8850|▁▁▅▃▇ |
|death_height_metres  |     67460|          0.01| 6671.63| 1295.21|  400| 5800| 6700| 7600.0| 8830|▁▁▂▇▆ |
|injury_height_metres |     67486|          0.01| 7111.95| 1200.64|  400| 6300| 7200| 8037.5| 8880|▁▁▁▆▇ |
Now, we will look at the possible values for each character value (minus member ID and expedition ID). This will give us an idea of how many levels we will deal with if we transform these variables into factors.

```r
# Selecting character variables and looping levels function over each variable
himal_100_df %>% select_if(.predicate = is.character) %>%
  select(-member_id, -expedition_id) %>%
  map(~levels(factor(.x)))
```

```
## $peak_id
##  [1] "AMAD" "ANN1" "ANN2" "ANN3" "ANN4" "ANNS" "APIM" "BARU" "BHRI" "CHAM"
## [11] "CHOL" "CHOY" "CHRW" "CTSE" "DHA1" "DHA2" "DHA4" "DHAM" "DORJ" "EVER"
## [21] "GANG" "GAUR" "GIMM" "GLAC" "GYAC" "GYAJ" "HIME" "HIML" "JANU" "KANG"
## [31] "KGUR" "KTEG" "LANG" "LHOT" "LSHR" "MAK2" "MAKA" "MANA" "NEMJ" "NUMB"
## [41] "NUPT" "PK29" "PUMO" "PUTH" "RATC" "SAIP" "SARI" "TAWO" "TILI" "TUKU"
## [51] "YALU"
## 
## $peak_name
##  [1] "Ama Dablam"        "Annapurna I"       "Annapurna II"     
##  [4] "Annapurna III"     "Annapurna IV"      "Annapurna South"  
##  [7] "Api Main"          "Baruntse"          "Bhrikuti"         
## [10] "Chamlang"          "Changtse"          "Cho Oyu"          
## [13] "Cholatse"          "Churen Himal West" "Dhampus"          
## [16] "Dhaulagiri I"      "Dhaulagiri II"     "Dhaulagiri IV"    
## [19] "Dorje Lhakpa"      "Everest"           "Gangapurna"       
## [22] "Gaurishankar"      "Gimmigela Chuli"   "Glacier Dome"     
## [25] "Gyachung Kang"     "Gyajikang"         "Himalchuli East"  
## [28] "Himlung Himal"     "Jannu"             "Kang Guru"        
## [31] "Kangchenjunga"     "Kangtega"          "Langtang Lirung"  
## [34] "Lhotse"            "Lhotse Shar"       "Makalu"           
## [37] "Makalu II"         "Manaslu"           "Nemjung"          
## [40] "Numbur"            "Nuptse"            "Peak 29"          
## [43] "Pumori"            "Putha Hiunchuli"   "Ratna Chuli"      
## [46] "Saipal"            "Saribung"          "Tawoche"          
## [49] "Tilicho"           "Tukuche"           "Yalung Kang"      
## 
## $season
## [1] "Autumn" "Spring" "Summer" "Winter"
## 
## $sex
## [1] "F" "M"
## 
## $citizenship
##   [1] "Albania"                 "Algeria"                
##   [3] "Andorra"                 "Argentina"              
##   [5] "Argentina/Canada"        "Armenia"                
##   [7] "Australia"               "Australia/Greece"       
##   [9] "Australia/Ireland"       "Australia/New Zealand"  
##  [11] "Australia/Sweden"        "Australia/UK"           
##  [13] "Australia/USA"           "Austria"                
##  [15] "Austria/Brazil"          "Azerbaijan"             
##  [17] "Azerbaijan/Russia"       "Bahrain"                
##  [19] "Bangladesh"              "Belarus"                
##  [21] "Belgium"                 "Bhutan"                 
##  [23] "Bolivia"                 "Bosnia-Herzegovina"     
##  [25] "Bosnia-Herzegovina/USA"  "Botswana"               
##  [27] "Brazil"                  "Bulgaria"               
##  [29] "Canada"                  "Canada/Ireland"         
##  [31] "Canada/Macedonia"        "Canada/Russia"          
##  [33] "Canada/UK"               "Canada/USA"             
##  [35] "Chile"                   "Chile/Sweden"           
##  [37] "China"                   "China/Japan"            
##  [39] "China/USA"               "Colombia"               
##  [41] "Colombia/USA"            "Costa Rica"             
##  [43] "Croatia"                 "Cyprus"                 
##  [45] "Czech Republic"          "Czechoslovakia"         
##  [47] "Denmark"                 "Denmark/Australia"      
##  [49] "Dominican Republic"      "Ecuador"                
##  [51] "Ecuador/Germany"         "Egypt"                  
##  [53] "Egypt/UK"                "Estonia"                
##  [55] "Finland"                 "France"                 
##  [57] "France/Algeria"          "France/Switzerland"     
##  [59] "France/USA"              "Georgia"                
##  [61] "Germany"                 "Germany/Iran"           
##  [63] "Germany/S Africa"        "Germany/Switzerland"    
##  [65] "Germany/USA"             "Greece"                 
##  [67] "Guatemala"               "Hong Kong"              
##  [69] "Hungary"                 "Hungary/Romania"        
##  [71] "Iceland"                 "India"                  
##  [73] "India/Nepal"             "Indonesia"              
##  [75] "Iran"                    "Ireland"                
##  [77] "Ireland/UK"              "Israel"                 
##  [79] "Italy"                   "Japan"                  
##  [81] "Japan/China"             "Jordan"                 
##  [83] "Kazakhstan"              "Kenya"                  
##  [85] "Kosovo"                  "Kuwait"                 
##  [87] "Kyrgyz Republic"         "Kyrgyzstan"             
##  [89] "Latvia"                  "Lebanon"                
##  [91] "Liechtenstein"           "Lithuania"              
##  [93] "Lithuania/USA"           "Luxembourg"             
##  [95] "Macedonia"               "Macedonia/Australia"    
##  [97] "Malaysia"                "Malta"                  
##  [99] "Mauritius"               "Mexico"                 
## [101] "Moldova"                 "Mongolia"               
## [103] "Montenegro"              "Morocco"                
## [105] "Myanmar"                 "N Korea"                
## [107] "Nepal"                   "Nepal/Australia"        
## [109] "Nepal/Canada"            "Nepal/India"            
## [111] "Nepal/India?"            "Netherlands"            
## [113] "Netherlands/Switzerland" "New Zealand"            
## [115] "New Zealand/Australia"   "New Zealand/UK"         
## [117] "Norway"                  "Oman"                   
## [119] "Pakistan"                "Palestine"              
## [121] "Panama"                  "Paraguay/Germany"       
## [123] "Peru"                    "Philippines"            
## [125] "Poland"                  "Poland/Canada"          
## [127] "Poland/USA"              "Portugal"               
## [129] "Qatar"                   "Romania"                
## [131] "Romania/USA"             "Russia"                 
## [133] "Russia/Kazakhstan"       "S Africa"               
## [135] "S Africa/UK"             "S Africa/Zambia"        
## [137] "S Korea"                 "San Marino"             
## [139] "Saudi Arabia"            "Saudi Arabia/USA"       
## [141] "Serbia"                  "Serbia/Jordan"          
## [143] "Singapore"               "Singapore/Malaysia"     
## [145] "Slovakia"                "Slovakia/USA"           
## [147] "Slovenia"                "Spain"                  
## [149] "Spain/Brazil"            "Spain/USA"              
## [151] "Sri Lanka"               "Sweden"                 
## [153] "Switzerland"             "Switzerland/France"     
## [155] "Switzerland/Germany"     "Switzerland/Greece"     
## [157] "Switzerland/UK"          "Switzerland/USA"        
## [159] "Syria"                   "Taiwan"                 
## [161] "Tanzania"                "Thailand"               
## [163] "Tunisia"                 "Turkey"                 
## [165] "Turkey/USA"              "Turkmenistan"           
## [167] "UAE"                     "UK"                     
## [169] "UK/Argentina"            "UK/Australia"           
## [171] "UK/Bangladesh"           "UK/Canada"              
## [173] "UK/Egypt"                "UK/Hong Kong"           
## [175] "UK/Iceland"              "UK/India"               
## [177] "UK/Italy"                "UK/Kenya"               
## [179] "UK/Lebanon"              "UK/Nepal"               
## [181] "UK/New Zealand"          "UK/Russia"              
## [183] "UK/S Africa"             "UK/USA"                 
## [185] "Ukraine"                 "Uruguay"                
## [187] "USA"                     "USA/Austria"            
## [189] "USA/Brazil"              "USA/Canada"             
## [191] "USA/China"               "USA/Dominican Republic" 
## [193] "USA/Ethiopia"            "USA/Iran"               
## [195] "USA/Ireland"             "USA/Israel"             
## [197] "USA/Jamaica"             "USA/Poland"             
## [199] "USA/Romania"             "USA/Russia"             
## [201] "USA/Switzerland"         "USA/UK"                 
## [203] "USSR"                    "Uzbekistan"             
## [205] "Venezuela"               "Venezuela/Spain"        
## [207] "Vietnam"                 "W Germany"              
## [209] "W Germany/Iran"          "Yugoslavia"             
## 
## $expedition_role
##   [1] "2nd BC Manager"            "2nd Deputy Leader"        
##   [3] "2nd Exp Doctor"            "2nd Sirdar"               
##   [5] "ABC Cook"                  "ABC Manager"              
##   [7] "ABC Manager (Tawoche)"     "ABC Mgr (Tawoche)"        
##   [9] "ABC Staff"                 "ABC support"              
##  [11] "Acting Leader"             "Admin support"            
##  [13] "Advance BC Manager"        "Advance Team Leader"      
##  [15] "Advisor"                   "Advisor/Leader"           
##  [17] "Airdrop Team"              "Army Leader (S)"          
##  [19] "Assistant"                 "Assistant BC Mgr"         
##  [21] "Assistant Cameraman"       "Assistant Coach"          
##  [23] "Assistant Guide"           "Assistant Leader"         
##  [25] "Assistant Sirdar"          "Assoc Member (N)"         
##  [27] "Assoc Member (S)"          "Asst BC Manager"          
##  [29] "Asst Chief Instructor"     "Asst Scientist"           
##  [31] "Asst Video Man"            "Base Camp Leader"         
##  [33] "BBC Producer"              "BC Administration"        
##  [35] "BC Administrator"          "BC Assistant"             
##  [37] "BC Chef"                   "BC Communications Mgr"    
##  [39] "BC Cook"                   "BC Food Manager"          
##  [41] "BC Kitchen Staff"          "BC Manager"               
##  [43] "BC Manager (C1 only)"      "BC Manager (until C1)"    
##  [45] "BC Manager & Cook"         "BC Manager?"              
##  [47] "BC Manager/Cameraman"      "BC Manager/Climber"       
##  [49] "BC Medical Officer"        "BC Member"                
##  [51] "BC Mgr"                    "BC Mgr (for Klein)"       
##  [53] "BC Mgr/Cameraman"          "BC Mgr/Coach"             
##  [55] "BC Nurse"                  "BC Photographer"          
##  [57] "BC Radio Operator"         "BC Sirdar"                
##  [59] "BC Staff"                  "BC Staff (KBS News)"      
##  [61] "BC Support"                "BC Support/Cameraman"     
##  [63] "BC Technician"             "BC Telecommunications"    
##  [65] "BC/ABC Mgr"                "Broadcast Crew"           
##  [67] "Business Manager"          "C1 Manager"               
##  [69] "C2 Cook"                   "Cameraman"                
##  [71] "Camp Manager"              "Cartographer"             
##  [73] "Cash Director"             "Charge of publicity"      
##  [75] "Chief Coach"               "Chief Instructor"         
##  [77] "Chief Leader"              "Cilmber"                  
##  [79] "Cinematographer"           "Climb Ldr (torchbearer 2)"
##  [81] "Climb Ldr (torchbearer 3)" "climber"                  
##  [83] "Climber"                   "Climber (Ama Dablam)"     
##  [85] "Climber (Autonomous)"      "Climber (BC/ABC staff)"   
##  [87] "Climber (Bobaye)"          "Climber (C2 only)"        
##  [89] "Climber (C3 only)"         "Climber (cameraman)"      
##  [91] "Climber (CCTV)"            "Climber (Cholatse)"       
##  [93] "Climber (CSS)"             "Climber (Dinner)"         
##  [95] "Climber (Film Team)"       "Climber (Grp 1)"          
##  [97] "Climber (Grp 2)"           "Climber (Guest)"          
##  [99] "Climber (I or II)"         "Climber (II)"             
## [101] "Climber (Jannu)"           "Climber (Kanch Central)"  
## [103] "Climber (Kanch S)"         "Climber (Lhotse Shar)"    
## [105] "Climber (Lhotse)"          "Climber (MM)"             
## [107] "Climber (N Col)"           "Climber (N Face)"         
## [109] "Climber (N)"               "Climber (Nampa)"          
## [111] "Climber (NE Ridge)"        "Climber (Pvt)"            
## [113] "Climber (rope team)"       "Climber (S Face)"         
## [115] "Climber (S)"               "Climber (security)"       
## [117] "Climber (support)"         "Climber (SW Face)"        
## [119] "Climber (Tawoche)"         "Climber (torch)"          
## [121] "Climber (torchbear 4)"     "Climber (torchbearer 1)"  
## [123] "Climber (torchbearer 5)"   "Climber (torchlighter)"   
## [125] "Climber (Trainee)"         "Climber (trekking agency)"
## [127] "Climber (unofficial)"      "Climber (W Ridge)"        
## [129] "Climber (with Arnot)"      "Climber (Xixa also)"      
## [131] "Climber & transport"       "Climber Guide"            
## [133] "Climber Leader"            "Climber Sirdar"           
## [135] "Climber/Asst BC Manager"   "Climber/BC Manager"       
## [137] "Climber/BC Mgr"            "Climber/Cameraman"        
## [139] "Climber/Cleaner"           "Climber/Coach"            
## [141] "Climber/Cyclist"           "Climber/Equipment Mgr"    
## [143] "Climber/Exp Doctor"        "Climber/Exp Doctor ?"     
## [145] "Climber/Exp Secretary"     "Climber/Exp. Doctor"      
## [147] "Climber/Film Crew"         "Climber/Guide"            
## [149] "Climber/H-A Worker"        "Climber/Interpreter"      
## [151] "Climber/Journalist"        "Climber/Kayaker"          
## [153] "Climber/Manager"           "Climber/Reporter"         
## [155] "Climber/Research"          "Climber/Scientist"        
## [157] "Climber/Sirdar"            "Climber/Transport"        
## [159] "Climber/Trek Guide"        "Climbing Advisor"         
## [161] "Climbing Assistant"        "Climbing Guide"           
## [163] "Climbing Guide (Grp 1)"    "Climbing Guide (Pvt)_"    
## [165] "Climbing Ldr (N Face)"     "Climbing Ldr (N)"         
## [167] "Climbing Ldr (NE Ridge)"   "Climbing Ldr (S)"         
## [169] "Climbing Leader"           "Climbing Leader (Lhotse)" 
## [171] "Climbing Leader (Tawoche)" "Climbing Scientist"       
## [173] "Climbing sirdar"           "Climbing Sirdar"          
## [175] "Co-Deputy Leader"          "Co-Leader"                
## [177] "Co-Pilot"                  "Commandant"               
## [179] "Commander General"         "Commander-in-Chief"       
## [181] "Communications"            "Communications Mgr"       
## [183] "Communications Officer"    "Communications staff"     
## [185] "Convenor/Organizer"        "Cook"                     
## [187] "Cook at C2"                "Dep Climbing Ldr (N)"     
## [189] "Dep Climbing Ldr (S)"      "Dep Climbing Leader"      
## [191] "Dep Ldr Scientist"         "Dep Ldr/Climb Ldr"        
## [193] "Dep Ldr/Exp Doctor"        "Dep Ldr/Organizer"        
## [195] "Dep Leader (Film Team)"    "Dep Leader & BC Manager"  
## [197] "Dep Leader/Exp Doctor"     "Deputy Climbing Leader"   
## [199] "Deputy Gen Ldr"            "Deputy Ldr (N Face)"      
## [201] "Deputy Ldr (NE Ridge)"     "Deputy Ldr/BC Mgr"        
## [203] "Deputy Ldr/Climbing Ldr"   "Deputy Ldr/Exp Doctor"    
## [205] "Deputy Ldr/Exp Mgr"        "Deputy leader"            
## [207] "Deputy Leader"             "Deputy Leader (Ama Dab)"  
## [209] "Deputy Leader (French)"    "Deputy Leader (Xixa)"     
## [211] "Deputy Leader I/QM"        "Deputy Leader II"         
## [213] "Deputy Leader III"         "Deputy Leader?"           
## [215] "Deputy Leader/BC Mgr"      "Deputy Leader/Guide"      
## [217] "Ecological Survey"         "Equipment"                
## [219] "Everest Climber"           "Everest-Lhotse Climber"   
## [221] "Exp Advisor"               "Exp Artist"               
## [223] "Exp Captain"               "Exp Captain (Ama Dablam)" 
## [225] "Exp Commandant"            "Exp Commander"            
## [227] "Exp Cook"                  "Exp Dentist"              
## [229] "Exp Docter"                "Exp Doctor"               
## [231] "Exp Doctor (ABC)"          "Exp Doctor (C1 only)"     
## [233] "Exp Doctor (C2 only)"      "Exp Doctor (Ev & Lhotse)" 
## [235] "Exp Doctor (N side)"       "Exp Doctor (NE Ridge)"    
## [237] "Exp Doctor (S side)"       "Exp Doctor (S)"           
## [239] "Exp Doctor (SW Face)"      "Exp Doctor (W Ridge)"     
## [241] "Exp Doctor & Logistics"    "Exp Doctor 1"             
## [243] "Exp Doctor 2"              "Exp Doctor, BC Mgr"       
## [245] "Exp Doctor/BC Manager"     "Exp Doctor/Climber"       
## [247] "Exp Doctor/Kayaker"        "Exp Doctor/Medical Res."  
## [249] "Exp Doctor/Researcher"     "Exp Driver"               
## [251] "Exp Guide"                 "Exp Interpreter"          
## [253] "Exp Leader"                "Exp Manager"              
## [255] "Exp Nurse"                 "Exp Organizer"            
## [257] "Exp Organizer (BC)"        "Exp Pharmacist"           
## [259] "Exp Photographer"          "Exp President"            
## [261] "Exp Satellite Engineer"    "Exp Secretary"            
## [263] "Exp Trainer"               "Exp TV Director"          
## [265] "Face Leader"               "Film Crew"                
## [267] "Film Crew (TET)"           "Film Director"            
## [269] "Film Support"              "Film support crew"        
## [271] "Film team"                 "Film Team"                
## [273] "Film Team (NE Ridge)"      "Film Team Leader"         
## [275] "Film Team Liaison"         "Film Team/Exp Doctor"     
## [277] "Food"                      "Food Manager"             
## [279] "Food Officer"              "Foods"                    
## [281] "General Advisor"           "General Affairs"          
## [283] "General Commander"         "General Leader"           
## [285] "General Manager"           "Gurkha Officer"           
## [287] "H-A Assistand"             "H-A Assistant"            
## [289] "H-A Assistant (Guide"      "H-A Assistant (Guide)"    
## [291] "H-A Cameraman"             "H-A Cook"                 
## [293] "H-A Exp Doctor"            "H-A Rope-Fixer"           
## [295] "H-A Worder"                "H-A Worker"               
## [297] "H-A WOrker"                "H-A Worker (C4 Mgr)"      
## [299] "H-A Worker (Cook)"         "H-A Worker/Climber"       
## [301] "H-A Worker/Kitchen Boy"    "H-W Worker"               
## [303] "HA-Cook"                   "HA-Worker"                
## [305] "Hang-glider Pilot"         "Head of Organization"     
## [307] "Helicopter Tech"           "High-Altitude Doctor"     
## [309] "Historian"                 "Honorary Leader"          
## [311] "Icefall Doctor"            "Interpreter"              
## [313] "Intrepreter"               "Joint Leader"             
## [315] "Journalist"                "Journalist at BC"         
## [317] "Journalist/Photographer"   "Kayaker"                  
## [319] "Kitchen Boy"               "Kitchen Staff"            
## [321] "Ladder Removal"            "leader"                   
## [323] "Leader"                    "Leader (Admin)"           
## [325] "Leader (American)"         "Leader (BC only)"         
## [327] "Leader (Czech)"            "Leader (French)"          
## [329] "Leader (Group 1)"          "Leader (Group 2)"         
## [331] "Leader (Honorable)"        "Leader (II)"              
## [333] "Leader (Ktm only)"         "Leader (Lhotse S Face)"   
## [335] "Leader (Lhotse W Face)"    "Leader (N)"               
## [337] "Leader (nomimal)"          "Leader (Nominal)"         
## [339] "Leader (North side)"       "Leader (S)"               
## [341] "Leader (South side)"       "Leader (Swiss)"           
## [343] "Leader (Tawoche)"          "Leader (Tibetan Staff)"   
## [345] "Leader (until BC)"         "Leader (until C1)"        
## [347] "Leader (Xixa also)"        "Leader & Exp Doctor"      
## [349] "Leader of Hankuk Grp"      "Leader of Jap. Mbrs"      
## [351] "Leader of KPNU Grp"        "Leader Scientist"         
## [353] "Leader, BC Manager"        "Leader, Exp Doctor"       
## [355] "Leader, Manager"           "Leader, Organizer"        
## [357] "Leader, Speed Climber"     "Leader?"                  
## [359] "Leader/Adv BC Manager"     "Leader/BC Manager"        
## [361] "Leader/BC Mgr"             "Leader/Exp Doctor"        
## [363] "Leader/Exp Mgr"            "Leader/Film Team"         
## [365] "Leader/Instructor"         "Leader/Journalist"        
## [367] "Leader/Organizer"          "Leader/Transport"         
## [369] "Lhotse Climber"            "Liaison Officer"          
## [371] "Logistics Supervisor"      "Mailrunner"               
## [373] "Manager"                   "Media Coordinator"        
## [375] "Media Officer"             "Medical Assistant"        
## [377] "Medical Officer"           "Medical Research"         
## [379] "Medical Team"              "Member"                   
## [381] "Member (ABC/N Col)"        "Member/Instructor"        
## [383] "Meteorologist"             "Mgr & Climbing Ldr"       
## [385] "Mountain guide"            "Movie Team"               
## [387] "Naike"                     "Non-Climber"              
## [389] "Nuptse Group Leader"       "Nurse (S side)"           
## [391] "Organizer"                 "Organizer & Med Off"      
## [393] "Original Deputy Leader"    "Original Leader"          
## [395] "Photo Team"                "Photographer"             
## [397] "Physiologist"              "Pilot"                    
## [399] "Piotr Czeslaw"             "Police Leader"            
## [401] "Porter"                    "PR & media manager"       
## [403] "President"                 "Press Correspondent"      
## [405] "Press Director"            "Press Officer"            
## [407] "Producer, Director"        "Programme Director"       
## [409] "Radio Officer"             "Radio operator"           
## [411] "Radio Operator"            "Radio Operator (BC)"      
## [413] "Radio Technician"          "Reporter"                 
## [415] "Reporter Team"             "Research"                 
## [417] "Research Doctor"           "Researcher"               
## [419] "Rope Team"                 "Rope Team/H-A Assistant"  
## [421] "Rope-fixing"               "Sandrine Marie Rose"      
## [423] "Scientific Coordinator"    "Scientific Team"          
## [425] "Scientist"                 "Secretary"                
## [427] "Secretary/interpreter"     "Signal Officer"           
## [429] "Sikkim Police"             "Sirdar"                   
## [431] "Sirdar/Climber"            "Sirdar/Climbing Ldr"      
## [433] "Sirdar/Cook"               "Sirder"                   
## [435] "Ski Team"                  "Ski Team Leader"          
## [437] "Skiing Demonstrator"       "Staff"                    
## [439] "Staff Member"              "Staff Photographer"       
## [441] "Staff Writer"              "Standing V. Pres"         
## [443] "Storekeeper"               "Student Leader"           
## [445] "Support"                   "Support Climber"          
## [447] "Support Crew"              "Support member"           
## [449] "Support Member"            "Support Member (N side)"  
## [451] "Support Member (S side)"   "Support Member (S)"       
## [453] "Survey Party Leader"       "Surveyor"                 
## [455] "Tactician"                 "Team Historian"           
## [457] "Team Leader (N)"           "Team Leader (S)"          
## [459] "Team Mgr, Trainer"         "Technical Advisor"        
## [461] "Technical Manager"         "Trainer"                  
## [463] "Transport"                 "Transport Manager"        
## [465] "Transport Officer"         "Transport Offier"         
## [467] "Transportation"            "Treasurer"                
## [469] "Trekker"                   "Trekker (non-member)"     
## [471] "Trekking Leader"           "TV & Radio Team"          
## [473] "TV Cameraman"              "TV Crew"                  
## [475] "TV Director"               "TV Film Team"             
## [477] "TV Producer"               "TV Reporter"              
## [479] "Vice Leader"               "Video Editor"             
## [481] "Video Photographer"        "Winch Operator at C1"     
## [483] "Wireless Operater"         "Wireless Operator"        
## [485] "Woodcutter"                "xClimber"                 
## 
## $death_cause
##  [1] "AMS"                         "Avalanche"                  
##  [3] "Crevasse"                    "Disappearance (unexplained)"
##  [5] "Exhaustion"                  "Exposure / frostbite"       
##  [7] "Fall"                        "Falling rock / ice"         
##  [9] "Icefall collapse"            "Illness (non-AMS)"          
## [11] "Other"                       "Unknown"                    
## 
## $injury_type
##  [1] "AMS"                         "Avalanche"                  
##  [3] "Crevasse"                    "Disappearance (unexplained)"
##  [5] "Exhaustion"                  "Exposure / frostbite"       
##  [7] "Fall"                        "Falling rock / ice"         
##  [9] "Icefall collapse"            "Illness (non-AMS)"          
## [11] "Other"
```

## Creating Modeling Dataframe
Here we will select the features (variables) we will include in our model. We will used variables *peak_id*, *year*, *season*, *sex*, *age*, *citizenship*, *hired*, *solo*, and *oxygen_used* to try and predict *success*.  

```r
# Removing observation where season is not known and sex and citizenship is NA
himal_df <- himal_100_df %>%
  filter(season != "Unknown", 
         !is.na(sex), 
         !is.na(citizenship)) %>%
  select(peak_id, year, season, sex, age, citizenship, 
         hired, success, solo, oxygen_used) %>%
  mutate_if(is.character, factor) %>%
  mutate_if(is.logical, as.integer) %>%
  mutate(success = factor(success, levels = c(0, 1), labels = c("fail", "success")))
```


Let's pause here to see how balanced our data is.

```r
# Checking outcome balance
table(himal_df$success)
```

```
## 
##    fail success 
##   41954   26479
```
Our outcome is not perfectly balanced, but it should be fine!


## Splitting the Data

```r
# Creating training and test splits. Stratifying by outcome: success
set.seed(111)
himal_df_split <- initial_split(himal_df, strata = success)
himal_train <- training(himal_df_split)
himal_test <- testing(himal_df_split)
```

## Creating Ten-fold Cross-Validation Folds

```r
# Creating 10 fold cross-validation folds. Stratifying by outcome: success
set.seed(222)
himal_folds <- vfold_cv(himal_train, strata = success)
himal_folds
```

```
## #  10-fold cross-validation using stratification 
## # A tibble: 10 × 2
##    splits               id    
##    <list>               <chr> 
##  1 <split [46191/5133]> Fold01
##  2 <split [46191/5133]> Fold02
##  3 <split [46191/5133]> Fold03
##  4 <split [46191/5133]> Fold04
##  5 <split [46191/5133]> Fold05
##  6 <split [46192/5132]> Fold06
##  7 <split [46192/5132]> Fold07
##  8 <split [46192/5132]> Fold08
##  9 <split [46192/5132]> Fold09
## 10 <split [46193/5131]> Fold10
```

## Preprocessing Recipe
- First, we will define the model within the recipe. 
- Next, we will create a high cardinality mini-model for peak_id where the mini-model step will convert the nominal (i.e., factor) predictor (peak_id) into a single set of scores derived from a generalized linear model.
- From our **skim()** of the dataframe, we saw over two thousand rows with a missing age variable. Instead of removing those rows, we will impute the median age to be able to get the rest of the data in those rows.
- Like peak_id, the Variable *citizenship* also has high cardinality. However, *citizenship's* cardinality is over four times that of peak_id, full of many small cell counts and multi-country combinations. To address this, we will create a new category (other) within *citizenship* to capture all instances representing less than 5 percent of the data.
- Lastly, we will use at least one prediction model that does not handle factor variables. We must create dummy variables from each of our dataframe's factor variables, except for our outcome variable *success*.


```r
library(embed)
himal_recip <- recipe(success ~ ., data = himal_train) %>%
  embed::step_lencode_glm(peak_id, outcome = vars(success)) %>%
  step_impute_median(age) %>%
  step_other(citizenship) %>%
  step_dummy(all_nominal(), -success)

himal_recip
```

```
## 
```

```
## ── Recipe ──────────────────────────────────────────────────────────────────────
```

```
## 
```

```
## ── Inputs
```

```
## Number of variables by role
```

```
## outcome:   1
## predictor: 9
```

```
## 
```

```
## ── Operations
```

```
## • Linear embedding for factors via GLM for: peak_id
```

```
## • Median imputation for: age
```

```
## • Collapsing factor levels for: citizenship
```

```
## • Dummy variables from: all_nominal(), -success
```
We now have our data recipe. We must run it through the **prep()** and **bake()** functions to see it applied to the training data.

```r
prep(himal_recip) %>%
  bake(new_data = NULL)
```

```
## # A tibble: 51,324 × 16
##    peak_id  year   age hired  solo oxygen_used success season_Spring
##      <dbl> <dbl> <dbl> <int> <int>       <int> <fct>           <dbl>
##  1  -0.107  1978    40     0     0           0 fail                0
##  2  -0.107  1978    41     0     0           0 fail                0
##  3  -0.107  1978    40     0     0           0 fail                0
##  4  -0.107  1978    25     0     0           0 fail                0
##  5  -0.107  1978    41     0     0           0 fail                0
##  6  -0.107  1978    29     0     0           0 fail                0
##  7  -0.107  1979    35     0     0           0 fail                1
##  8  -0.107  1979    44     0     0           0 fail                1
##  9  -0.107  1979    28     0     0           0 fail                1
## 10  -0.107  1979    32     0     0           0 fail                1
## # ℹ 51,314 more rows
## # ℹ 8 more variables: season_Summer <dbl>, season_Winter <dbl>, sex_M <dbl>,
## #   citizenship_Japan <dbl>, citizenship_Nepal <dbl>, citizenship_UK <dbl>,
## #   citizenship_USA <dbl>, citizenship_other <dbl>
```


Let's look at the tidied result of the first step of our recipe, where we convert the predictor *peak_id* into a single set of scores derived from a generalized linear model. 

```r
prep(himal_recip) %>%
  tidy(number = 1) %>%
  arrange(value)
```

```
## # A tibble: 52 × 4
##    level  value terms   id               
##    <chr>  <dbl> <chr>   <chr>            
##  1 SARI  -0.785 peak_id lencode_glm_iEZlM
##  2 DHAM  -0.560 peak_id lencode_glm_iEZlM
##  3 AMAD  -0.107 peak_id lencode_glm_iEZlM
##  4 EVER   0.156 peak_id lencode_glm_iEZlM
##  5 CHOL   0.203 peak_id lencode_glm_iEZlM
##  6 CHOY   0.284 peak_id lencode_glm_iEZlM
##  7 HIML   0.303 peak_id lencode_glm_iEZlM
##  8 CTSE   0.389 peak_id lencode_glm_iEZlM
##  9 PUTH   0.441 peak_id lencode_glm_iEZlM
## 10 MANA   0.485 peak_id lencode_glm_iEZlM
## # ℹ 42 more rows
```
Here we can see that each peak has been converted to the quantified association with the outcome variable *success* and that the same peaks we identified as having the lowest success rate in our EDA are also negatively associated with success in our new peak_id variable.

However, this is a living database that is still being updated. How would ascents of new peaks be predicted by this model? You may notice a new level within peak_id *..new*. This is a placeholder for any encounter of a new peak that includes a value that roughly equals the median value of all the peaks. Therefore, the *embed* package and the **step_lencode_glm()** function have built-in imputation!


```r
prep(himal_recip) %>%
  tidy(number = 1) %>%
  filter(level == "..new")
```

```
## # A tibble: 1 × 4
##   level value terms   id               
##   <chr> <dbl> <chr>   <chr>            
## 1 ..new  1.18 peak_id lencode_glm_iEZlM
```


## Model Specifications
Logistic regression and random forest are not the most complex prediction algorithms. However, I like using these two models early in my prediction journey for many reasons. First, they are both quick to run, and the results are far more interpretable than many other algorithms. Second, I often don't bother to tune any hyperparameters because 1) they are no hyperparameters to tune for logistic regression,  or 2) random forests often perform well without extensive hyperparameter tuning. 

However, for this demonstration, we will tune the random forest model to optimize model fit.

```r
# Creating a logistic regression model specification
log_spec <- logistic_reg() %>%
  set_engine("glm")

# Creating a random forest model specification with tuning parameters
rf_spec <- rand_forest(mtry = tune(),
                       min_n = tune(), 
                       trees = 1000) %>%
  set_engine("ranger") %>%
  set_mode("classification")
```


## Creating Workflows
[Workflows](https://workflows.tidymodels.org) are the tidymodel's method of compartmentalizing (or bundling) the pre-processing, modeling, (and even post-processing) steps of your modeling process.   

### Logistic Regression Model
We will name our logistic regression model workflow "*log_wf*," which consists of our recipe and model specification.

```r
# Building logistic regression workflow
log_wf <- workflow() %>%
  add_recipe(himal_recip) %>%
  add_model(log_spec)
```

Here we will turn our logistic regression model loose on the cross-validation folds from the training data.

```r
# Fitting the logistic regression model to the cross-validation folds from the
# training data.
log_wf_train <- log_wf %>%
  fit_resamples(
    resamples = himal_folds,
    metrics = metric_set(roc_auc, accuracy, sensitivity, specificity), 
    control = control_resamples(save_pred = TRUE)
    )
```

### Random Forest Model 
Here, we are naming our workflow "*rf_wf*," which consists of our recipe and model specification.

```r
# Building random forest workflow
rf_wf <- workflow() %>%
  add_recipe(himal_recip) %>%
  add_model(rf_spec) 
```


Now we will tune the *mtry* and *min_n* hyperparameters in our random forest model using 13 parameter sets.

```r
set.seed(333)
doParallel::registerDoParallel()
rf_train_rs <- tune_grid(object = rf_wf,
                         resamples = himal_folds, 
                         grid = 13, 
                         metrics = metric_set(roc_auc, accuracy, sensitivity, specificity), 
                         control = control_resamples(save_pred = TRUE)
    )
```

```
## i Creating pre-processing data to finalize unknown parameter: mtry
```

```
## Warning: package 'ranger' was built under R version 4.1.2
```
![](https://media.gadventures.com/media-server/image_library/variants/itinerary_mobi_Nepal-Himalaya-Mountains-Annapurna-Pokhara-Prayer-Flags-IS-027332084-Lg-RGB.jpg)
Tashiling Tibetan Refugee Camp (Photo Credit: [G Adventures](https://www.gadventures.com/trips/nepal-himalaya-highlights/ANENG/))

## Model Evaluation
### Logistic Regression

```r
collect_metrics(log_wf_train)
```

```
## # A tibble: 4 × 6
##   .metric     .estimator  mean     n std_err .config             
##   <chr>       <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy    binary     0.772    10 0.00227 Preprocessor1_Model1
## 2 roc_auc     binary     0.809    10 0.00246 Preprocessor1_Model1
## 3 sensitivity binary     0.882    10 0.00190 Preprocessor1_Model1
## 4 specificity binary     0.598    10 0.00449 Preprocessor1_Model1
```
Our logistic regression model accurately identified about 77% percent of summit attempts correctly, with an Area Under the Curve (AUC) of over 80% percent. Not horrible, but let's see if our random forest model can outperform that.

### Random Forest
For our random forest model, we will plot the values for accuracy, roc_auc, sensitivity, and specificity. The result of the tuning object works seamlessly with the **autoplot()** function. Next, we will identify the tuning parameters with the highest accuracy and roc_auc.

```r
autoplot(rf_train_rs)
```

<img src="/blog/himalaya/himal_climb_files/figure-html/unnamed-chunk-19-1.png" width="672" />

```r
collect_metrics(rf_train_rs) %>%
  filter(.metric == "accuracy") %>%
  arrange(desc(mean))
```

```
## # A tibble: 13 × 8
##     mtry min_n .metric  .estimator  mean     n std_err .config              
##    <int> <int> <chr>    <chr>      <dbl> <int>   <dbl> <chr>                
##  1     7    16 accuracy binary     0.808    10 0.00169 Preprocessor1_Model01
##  2     8    17 accuracy binary     0.807    10 0.00176 Preprocessor1_Model03
##  3    11    39 accuracy binary     0.806    10 0.00184 Preprocessor1_Model04
##  4     9    12 accuracy binary     0.806    10 0.00176 Preprocessor1_Model07
##  5    12    36 accuracy binary     0.805    10 0.00187 Preprocessor1_Model05
##  6    13    20 accuracy binary     0.805    10 0.00177 Preprocessor1_Model13
##  7     5    24 accuracy binary     0.804    10 0.00182 Preprocessor1_Model11
##  8     5    30 accuracy binary     0.804    10 0.00171 Preprocessor1_Model09
##  9    11    10 accuracy binary     0.803    10 0.00143 Preprocessor1_Model10
## 10     4    26 accuracy binary     0.800    10 0.00189 Preprocessor1_Model12
## 11     3     6 accuracy binary     0.795    10 0.00225 Preprocessor1_Model06
## 12    14     4 accuracy binary     0.793    10 0.00161 Preprocessor1_Model08
## 13     2    32 accuracy binary     0.779    10 0.00202 Preprocessor1_Model02
```

```r
collect_metrics(rf_train_rs) %>%
  filter(.metric == "roc_auc") %>%
  arrange(desc(mean))
```

```
## # A tibble: 13 × 8
##     mtry min_n .metric .estimator  mean     n std_err .config              
##    <int> <int> <chr>   <chr>      <dbl> <int>   <dbl> <chr>                
##  1     7    16 roc_auc binary     0.884    10 0.00183 Preprocessor1_Model01
##  2     8    17 roc_auc binary     0.883    10 0.00187 Preprocessor1_Model03
##  3    11    39 roc_auc binary     0.882    10 0.00190 Preprocessor1_Model04
##  4    12    36 roc_auc binary     0.882    10 0.00192 Preprocessor1_Model05
##  5     9    12 roc_auc binary     0.882    10 0.00189 Preprocessor1_Model07
##  6    13    20 roc_auc binary     0.880    10 0.00195 Preprocessor1_Model13
##  7     5    24 roc_auc binary     0.879    10 0.00190 Preprocessor1_Model11
##  8     5    30 roc_auc binary     0.878    10 0.00187 Preprocessor1_Model09
##  9    11    10 roc_auc binary     0.878    10 0.00188 Preprocessor1_Model10
## 10     4    26 roc_auc binary     0.873    10 0.00191 Preprocessor1_Model12
## 11    14     4 roc_auc binary     0.867    10 0.00187 Preprocessor1_Model08
## 12     3     6 roc_auc binary     0.863    10 0.00204 Preprocessor1_Model06
## 13     2    32 roc_auc binary     0.840    10 0.00223 Preprocessor1_Model02
```

The output above shows that a *mtry* value of 7 and a min_n value of 16 produce the highest accuracy and roc_auc on the cross-validation training folds. We can easily select the model with the best hyperparameters with the **select_best()** function.

```r
# Defining the best performing random forest model (by mtry) based on accuracy
best_rf_model <- select_best(rf_train_rs, metric = "accuracy", mtry)

best_rf_model
```

```
## # A tibble: 1 × 3
##    mtry min_n .config              
##   <int> <int> <chr>                
## 1     7    16 Preprocessor1_Model01
```

## Finalize Model
### Finalized Workflow

```r
# Specifying final random forest workflow with best performing tunning model
final_rf_wf <- finalize_workflow(rf_wf, parameters = best_rf_model)
final_rf_wf
```

```
## ══ Workflow ════════════════════════════════════════════════════════════════════
## Preprocessor: Recipe
## Model: rand_forest()
## 
## ── Preprocessor ────────────────────────────────────────────────────────────────
## 4 Recipe Steps
## 
## • step_lencode_glm()
## • step_impute_median()
## • step_other()
## • step_dummy()
## 
## ── Model ───────────────────────────────────────────────────────────────────────
## Random Forest Model Specification (classification)
## 
## Main Arguments:
##   mtry = 7
##   trees = 1000
##   min_n = 16
## 
## Computational engine: ranger
```
### Finalized Model Fit

```r
# Fitting final random forest workflow to training data
set.seed(444)
doParallel::registerDoParallel()
final_rf_fit <- fit(final_rf_wf, himal_train)
final_rf_fit
```

```
## ══ Workflow [trained] ══════════════════════════════════════════════════════════
## Preprocessor: Recipe
## Model: rand_forest()
## 
## ── Preprocessor ────────────────────────────────────────────────────────────────
## 4 Recipe Steps
## 
## • step_lencode_glm()
## • step_impute_median()
## • step_other()
## • step_dummy()
## 
## ── Model ───────────────────────────────────────────────────────────────────────
## Ranger result
## 
## Call:
##  ranger::ranger(x = maybe_data_frame(x), y = y, mtry = min_cols(~7L,      x), num.trees = ~1000, min.node.size = min_rows(~16L, x),      num.threads = 1, verbose = FALSE, seed = sample.int(10^5,          1), probability = TRUE) 
## 
## Type:                             Probability estimation 
## Number of trees:                  1000 
## Sample size:                      51324 
## Number of independent variables:  15 
## Mtry:                             7 
## Target node size:                 16 
## Variable importance mode:         none 
## Splitrule:                        gini 
## OOB prediction error (Brier s.):  0.1329073
```

```r
# saving the model
# saveRDS(final_rf_fit, file = "your_filepath_here/final_rf_fit.rda")
```
We now have a trained workflow consisting of the preprocessing steps and a tuned model (tuned on cv-folds from the training data). We can save this object and use it to predict on new data in the future.

### Evaluating Model on Test Data
Luckily, we don't have to wait for new climbing ascents. We can evaluate our models on the test data we created in our initial data split. 

First, we will evaluate our logistic regression model.

```r
# Fitting the logistic regression model to the testing data
himal_log_final_fit <- log_wf %>%
  last_fit(split = himal_df_split)

# Collecting test accuracy and roc_auc for logistic regression model
collect_metrics(himal_log_final_fit)
```

```
## # A tibble: 2 × 4
##   .metric  .estimator .estimate .config             
##   <chr>    <chr>          <dbl> <chr>               
## 1 accuracy binary         0.774 Preprocessor1_Model1
## 2 roc_auc  binary         0.811 Preprocessor1_Model1
```

```r
# Creating a confusion matrix for model performance on test data
collect_predictions(himal_log_final_fit) %>%
  conf_mat(success, .pred_class)
```

```
##           Truth
## Prediction fail success
##    fail    9224    2600
##    success 1265    4020
```
Our logistic regression metrics on the test data are almost exactly equal to the training metrics (the test accuracy and roc_auc are actually a little better than the training metrics). Therefore, we certainly didn't overfit our training data with the logistic regression model.


Now we will evaluate our random forest model on the test data.

```r
# Predicting success on new (test) data

# doParallel::registerDoParallel()
# test_rs <- predict(final_rf_fit, new_data = himal_test) %>%
#   bind_cols(himal_test) %>%
#   select(peak_id:hired, solo, oxygen_used, success, .pred_class)
# 
# test_rs %>%
#   mutate(.pred_class = as.numeric(.pred_class)) %>%
#   roc_curve(truth = success, .pred_class) %>%
#   autoplot()

# Enabling parallel-processing
doParallel::registerDoParallel()

set.seed(555)
# Predicting outcomes on test data with random forest model
himal_rf_final_fit <- final_rf_wf %>%
  last_fit(split = himal_df_split)

# Collecting test accuracy and roc_auc for random forest model
collect_metrics(himal_rf_final_fit)
```

```
## # A tibble: 2 × 4
##   .metric  .estimator .estimate .config             
##   <chr>    <chr>          <dbl> <chr>               
## 1 accuracy binary         0.813 Preprocessor1_Model1
## 2 roc_auc  binary         0.888 Preprocessor1_Model1
```

```r
# Creating a confusion matrix for model performance on test data
collect_predictions(himal_rf_final_fit) %>%
  conf_mat(success, .pred_class)
```

```
##           Truth
## Prediction fail success
##    fail    9155    1859
##    success 1334    4761
```

```r
# Ploting ROC AUC curve of random forest model performance
collect_predictions(himal_rf_final_fit) %>%
  roc_curve(truth = success, .pred_success, event_level = "second") %>%
  autoplot()
```

<img src="/blog/himalaya/himal_climb_files/figure-html/unnamed-chunk-24-1.png" width="672" />

With our random forest model and our hyper-tuning, we increased our accuracy to over 0.81 and our AUC to nearly **0.89** on the test data. Not a bad a classifier for a fairly simple model. 

## Variable Importance
Unfortunately, we didn't set up our random forest *ranger* engine to be able to look at variable importance. Fortunately, we can lean on our logistic regression model to quantify the contribution of each predictor.


First, we can pull() the S3 object (.workflow) from the final logistic regression fit. Next, we can pluck the trained workflow and tidy the results. Here, we will exponentiate the coefficient estimates to get odds ratios.

```r
# Plucking the trained workflow from the final logistic regression model fit
himal_log_final_fit %>%
  pull(.workflow) %>%
  pluck(1) %>%
  tidy(exponentiate = TRUE) %>% 
  arrange(estimate) %>%
  knitr::kable(digits = 3)
```



|term              | estimate| std.error| statistic| p.value|
|:-----------------|--------:|---------:|---------:|-------:|
|(Intercept)       |    0.000|     1.985|   -22.303|   0.000|
|season_Summer     |    0.440|     0.169|    -4.866|   0.000|
|season_Spring     |    0.509|     0.025|   -27.574|   0.000|
|peak_id           |    0.551|     0.022|   -26.777|   0.000|
|season_Winter     |    0.567|     0.074|    -7.696|   0.000|
|citizenship_UK    |    0.689|     0.063|    -5.881|   0.000|
|citizenship_USA   |    0.788|     0.058|    -4.097|   0.000|
|citizenship_Japan |    0.924|     0.064|    -1.237|   0.216|
|citizenship_other |    0.958|     0.048|    -0.903|   0.367|
|age               |    0.979|     0.001|   -17.720|   0.000|
|year              |    1.022|     0.001|    22.227|   0.000|
|citizenship_Nepal |    1.139|     0.088|     1.473|   0.141|
|sex_M             |    1.227|     0.038|     5.351|   0.000|
|hired             |    1.717|     0.077|     7.023|   0.000|
|solo              |    5.342|     0.251|     6.673|   0.000|
|oxygen_used       |    9.629|     0.028|    80.653|   0.000|

We see that specific predictors like "*oxygen_used*" and "*solo*" seem to be strongly correlated with success, while other variables like "*season_Summer*" and "*season_Spring*" appear to be negatively correlated with a climber's probability of reaching the summit.

We can easily keep the estimated values on the log scale, which is often better for visualizing the data.

```r
himal_log_final_fit %>%
  pull(.workflow) %>%
  pluck(1) %>%
  tidy(exponentiate = FALSE) %>%
  arrange(estimate) %>%
  knitr::kable(digits = 3)
```



|term              | estimate| std.error| statistic| p.value|
|:-----------------|--------:|---------:|---------:|-------:|
|(Intercept)       |  -44.264|     1.985|   -22.303|   0.000|
|season_Summer     |   -0.820|     0.169|    -4.866|   0.000|
|season_Spring     |   -0.676|     0.025|   -27.574|   0.000|
|peak_id           |   -0.597|     0.022|   -26.777|   0.000|
|season_Winter     |   -0.568|     0.074|    -7.696|   0.000|
|citizenship_UK    |   -0.372|     0.063|    -5.881|   0.000|
|citizenship_USA   |   -0.238|     0.058|    -4.097|   0.000|
|citizenship_Japan |   -0.079|     0.064|    -1.237|   0.216|
|citizenship_other |   -0.043|     0.048|    -0.903|   0.367|
|age               |   -0.021|     0.001|   -17.720|   0.000|
|year              |    0.022|     0.001|    22.227|   0.000|
|citizenship_Nepal |    0.130|     0.088|     1.473|   0.141|
|sex_M             |    0.204|     0.038|     5.351|   0.000|
|hired             |    0.541|     0.077|     7.023|   0.000|
|solo              |    1.676|     0.251|     6.673|   0.000|
|oxygen_used       |    2.265|     0.028|    80.653|   0.000|


Let's plot this to make it easier to see the contribution of each predictor in our logistic regression model.

```r
himal_log_final_fit %>%
  pull(.workflow) %>%
  pluck(1) %>%
  tidy(exponentiate = FALSE) %>%
  filter(term != "(Intercept)") %>%
  ggplot(aes(estimate, fct_reorder(term, estimate))) +
  geom_vline(xintercept = 0, color = "gray50", lty = 2, size = 1.2) +
  geom_errorbar(aes(xmin = estimate - std.error, 
                    xmax = estimate + std.error), 
                width = 0.2, alpha = 0.7) +
  geom_point()
```

```
## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
## ℹ Please use `linewidth` instead.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
## generated.
```

<img src="/blog/himalaya/himal_climb_files/figure-html/unnamed-chunk-27-1.png" width="672" />
Interestingly, nearly all of our predictors (except for a few citizenship features) have a statistically significant relationship with the expedition's success. It is not surprising that the use of oxygen seems to significantly increase the likelihood of success. However, less intuitive is the relationship of solo climbing to our outcome. Perhaps soloists are among the most experienced and skilled climbers, and the routes and peaks they choose to solo have a lower difficulty and risk profile since they are climbing without the support of a partner or a team.

There is so much more to explore with this data set. Maybe I will revisit it in the future to answer a different question. Thanks for reading to the end!

![](https://img.traveltriangle.com/blog/wp-content/uploads/2020/01/cover-Mountaineering-in-Himalayas_17th-jan.jpg)

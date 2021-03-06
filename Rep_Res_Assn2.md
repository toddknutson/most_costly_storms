

Severe Weather Events in the US: Tornadoes, Floods, and Droughts Are Most Costly and Dangerous.
========================================================






## Synopsis: 

The National Oceanic and Atmospheric Association (NOAA) maintains a database of severe, rare, and other significant weather events in the USA that caused loss of life, significant damage, or disruption to commerce. The goal of this report is to identify (1) which types of events are most harmful with respect to population health, and (2) which types have the greatest economic consequences.

To address these questions, data from the NOAA's National Weather Service Instruction 10-1605 AUGUST 17 will be analyzed. We will identify the number of fatalities, number of injuries, property damage costs, and crop damage costs from the data set. We will summarize these values depending on the type of weather event and report storm types with the highest costs to human health and disruption of commerce.





## Data Processing

Import data set: National Weather Service Instruction 10-1605 AUGUST 17.

```r
# Data downloaded from Coursera website:
# https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2
data <- read.csv("repdata-data-StormData.csv.bz2", header = TRUE)
```

The property damage and crop damage dollar values are presented in a numeric value, multiplied by "hundreds"", "thousands"", "millions"", or "billions" labels. In addition, these damage values contain missing or unusual (erroneous) values. The small number of weather events containing these unusual values will be removed from the analysis. For all other weather events, new numeric vectors will be created containing the actual dollar values. These "tidy" dollar values will be added to the data set as new columns.

```r
# Summarize the dollar value multipliers for property damages:
kable(as.data.frame(table(data$PROPDMGEXP)), row.names = NA)
```



|Var1 |   Freq|
|:----|------:|
|     | 465934|
|-    |      1|
|?    |      8|
|+    |      5|
|0    |    216|
|1    |     25|
|2    |     13|
|3    |      4|
|4    |      4|
|5    |     28|
|6    |      4|
|7    |      5|
|8    |      1|
|B    |     40|
|h    |      1|
|H    |      6|
|K    | 424665|
|m    |      7|
|M    |  11330|

```r
# Summarize the dollar value multipliers for crop damages:
kable(as.data.frame(table(data$CROPDMGEXP)), row.names = NA)
```



|Var1 |   Freq|
|:----|------:|
|     | 618413|
|?    |      7|
|0    |     19|
|2    |      1|
|B    |      9|
|k    |     21|
|K    | 281832|
|m    |      1|
|M    |   1994|
Remove observations that contain unusual dollar amount multipliers, by keeping only observations that contain the correct multipliers.

```r
# Create a new vector of actual dollar amounts for "property damage"
# Get indicies for values
h_ind_pd <- which(data$PROPDMGEXP == "h" | data$PROPDMGEXP == "H")
k_ind_pd <- which(data$PROPDMGEXP == "k" | data$PROPDMGEXP == "K")
m_ind_pd <- which(data$PROPDMGEXP == "m" | data$PROPDMGEXP == "M")
b_ind_pd <- which(data$PROPDMGEXP == "b" | data$PROPDMGEXP == "B")
# initilize an ampty vector of NA values
prop_dmg_dollars <- rep(NA, dim(data)[1])
# Re-assign full dollar ammounts to vector
prop_dmg_dollars[h_ind_pd] <- data$PROPDMG[h_ind_pd] * 100
prop_dmg_dollars[k_ind_pd] <- data$PROPDMG[k_ind_pd] * 1000
prop_dmg_dollars[m_ind_pd] <- data$PROPDMG[m_ind_pd] * 1000000
prop_dmg_dollars[b_ind_pd] <- data$PROPDMG[b_ind_pd] * 1000000000
# Add new vector to data frame
data$prop_dmg_dollars <- prop_dmg_dollars


# Create a new vector of actual dollar amounts for "crop damage"
# Get indicies for values
h_ind_cd <- which(data$CROPDMGEXP == "h" | data$CROPDMGEXP == "H")
k_ind_cd <- which(data$CROPDMGEXP == "k" | data$CROPDMGEXP == "K")
m_ind_cd <- which(data$CROPDMGEXP == "m" | data$CROPDMGEXP == "M")
b_ind_cd <- which(data$CROPDMGEXP == "b" | data$CROPDMGEXP == "B")
# initilize an ampty vector of NA values
crop_dmg_dollars <- rep(NA, dim(data)[1])
# Re-assign full dollar ammounts to vector
crop_dmg_dollars[h_ind_cd] <- data$CROPDMG[h_ind_cd] * 100
crop_dmg_dollars[k_ind_cd] <- data$CROPDMG[k_ind_cd] * 1000
crop_dmg_dollars[m_ind_cd] <- data$CROPDMG[m_ind_cd] * 1000000
crop_dmg_dollars[b_ind_cd] <- data$CROPDMG[b_ind_cd] * 1000000000
# Add new vector to data frame
data$crop_dmg_dollars <- crop_dmg_dollars
```






Using our data set "data", we can find the total number of fatalities, injuries, property damage, and crop damage costs separated by storm type. These "summed" values can be displayed in a table:

```r
# Aggregate the data by Event Type
library(reshape)
data_melt <- melt(data, id.vars = "EVTYPE", measure.vars = c("INJURIES", "FATALITIES", "prop_dmg_dollars", "crop_dmg_dollars"), na.rm = TRUE)
data_cast <- cast(data_melt, EVTYPE ~ variable, sum)

# Display aggregated data
library(knitr)
kable(as.data.frame(head(data_cast)), row.names = NA)
```



|EVTYPE                | INJURIES| FATALITIES| prop_dmg_dollars| crop_dmg_dollars|
|:---------------------|--------:|----------:|----------------:|----------------:|
|   HIGH SURF ADVISORY |        0|          0|           200000|                0|
| COASTAL FLOOD        |        0|          0|                0|                0|
| FLASH FLOOD          |        0|          0|            50000|                0|
| LIGHTNING            |        0|          0|                0|                0|
| TSTM WIND            |        0|          0|          8100000|                0|
| TSTM WIND (G45)      |        0|          0|             8000|                0|


## Results



Using this aggregated data table, we can now sort each column to find which storms are most costly.

```r
top_injuries <- data_cast[order(data_cast$INJURIES, decreasing = TRUE), ]
top_fatalities <- data_cast[order(data_cast$FATALITIES, decreasing = TRUE), ]
top_prop_dmg <- data_cast[order(data_cast$prop_dmg_dollars, decreasing = TRUE), ]
top_crop_dmg <- data_cast[order(data_cast$crop_dmg_dollars, decreasing = TRUE), ]
```


### Table 1: Top storms that caused the most injuries.


```r
kable(head(top_injuries[1:20, ]), row.names = NA)
```



|    |EVTYPE         | INJURIES| FATALITIES| prop_dmg_dollars| crop_dmg_dollars|
|:---|:--------------|--------:|----------:|----------------:|----------------:|
|834 |TORNADO        |    91346|       5633|        5.694e+10|        4.150e+08|
|856 |TSTM WIND      |     6957|        504|        4.485e+09|        5.540e+08|
|170 |FLOOD          |     6789|        470|        1.447e+11|        5.662e+09|
|130 |EXCESSIVE HEAT |     6525|       1903|        7.754e+06|        4.924e+08|
|464 |LIGHTNING      |     5230|        816|        9.287e+08|        1.209e+07|
|275 |HEAT           |     2100|        937|        1.797e+06|        4.015e+08|

### Table 2: Top storms that caused the most deaths.

```r
kable(head(top_fatalities[1:20, ]), row.names = NA)
```



|    |EVTYPE         | INJURIES| FATALITIES| prop_dmg_dollars| crop_dmg_dollars|
|:---|:--------------|--------:|----------:|----------------:|----------------:|
|834 |TORNADO        |    91346|       5633|        5.694e+10|        4.150e+08|
|130 |EXCESSIVE HEAT |     6525|       1903|        7.754e+06|        4.924e+08|
|153 |FLASH FLOOD    |     1777|        978|        1.614e+10|        1.421e+09|
|275 |HEAT           |     2100|        937|        1.797e+06|        4.015e+08|
|464 |LIGHTNING      |     5230|        816|        9.287e+08|        1.209e+07|
|856 |TSTM WIND      |     6957|        504|        4.485e+09|        5.540e+08|


### Table 3: Top storms that caused the most property damage.

```r
kable(head(top_prop_dmg[1:20, ]), row.names = NA)
```



|    |EVTYPE            | INJURIES| FATALITIES| prop_dmg_dollars| crop_dmg_dollars|
|:---|:-----------------|--------:|----------:|----------------:|----------------:|
|170 |FLOOD             |     6789|        470|        1.447e+11|        5.662e+09|
|411 |HURRICANE/TYPHOON |     1275|         64|        6.931e+10|        2.608e+09|
|834 |TORNADO           |    91346|       5633|        5.694e+10|        4.150e+08|
|670 |STORM SURGE       |       38|         13|        4.332e+10|        5.000e+03|
|153 |FLASH FLOOD       |     1777|        978|        1.614e+10|        1.421e+09|
|244 |HAIL              |     1361|         15|        1.573e+10|        3.026e+09|


### Table 4: Top storms that caused the most crop damage.

```r
kable(head(top_crop_dmg[1:20, ]), row.names = NA)
```



|    |EVTYPE      | INJURIES| FATALITIES| prop_dmg_dollars| crop_dmg_dollars|
|:---|:-----------|--------:|----------:|----------------:|----------------:|
|95  |DROUGHT     |        4|          0|        1.046e+09|        1.397e+10|
|170 |FLOOD       |     6789|        470|        1.447e+11|        5.662e+09|
|590 |RIVER FLOOD |        2|          2|        5.119e+09|        5.029e+09|
|427 |ICE STORM   |     1975|         89|        3.945e+09|        5.022e+09|
|244 |HAIL        |     1361|         15|        1.573e+10|        3.026e+09|
|402 |HURRICANE   |       46|         61|        1.187e+10|        2.742e+09|




### Figure 1: Property and Crop Damages 
These sorted data can easily be used in plots. Below you can see the top 20 weather related events that caused the most number of injuries and deaths from 1950 to 2007.


```r
library(ggplot2)
p1 <- ggplot(top_injuries[1:20, ], aes(x= EVTYPE, y = INJURIES)) + geom_bar(fill = "orange", stat = "identity") + coord_flip() + scale_x_discrete(limits = rev(top_injuries$EVTYPE[1:20])) + geom_text(aes(label = top_injuries$INJURIES[1:20]), hjust = -0.3) + labs(x = "Weather Event", y = "Total Number of Injuries (1950 - 2007)", title = "Top 20 Weather Events That \nCaused The Most Injuries") + ylim(0, 110000)


p2 <- ggplot(top_fatalities[1:20, ], aes(x= EVTYPE, y = FATALITIES)) + geom_bar(fill = "red", stat = "identity") + coord_flip() + scale_x_discrete(limits = rev(top_fatalities$EVTYPE[1:20])) + geom_text(aes(label = FATALITIES[1:20]), hjust = -0.3) + labs(x = "Weather Event", y = "Total Number of Deaths (1950 - 2007)", title = "Top 20 Weather Events That\n Caused The Most Fatalities") + ylim(0, 6300)

library(gridExtra)
```

```
## Loading required package: grid
```

```r
grid.arrange(p1, p2, ncol = 2)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 



### Figure 2: Property and Crop Damages 
Below you can see the top 20 weather related events that caused the greatest amount of property and crop damages from 1950 to 2007.

```r
library(ggplot2)
p3 <- ggplot(top_prop_dmg[1:20, ], aes(x= EVTYPE, y = prop_dmg_dollars)) + geom_bar(fill = "blue", stat = "identity") + coord_flip() + scale_x_discrete(limits = rev(top_prop_dmg$EVTYPE[1:20])) + labs(x = "Weather Event", y = "Total Cost in Dollars (1950 - 2007)", title = "Top 20 Weather Events That Caused the\n Greatest Cost in Property Damage")


p4 <- ggplot(top_crop_dmg[1:20, ], aes(x= EVTYPE, y = crop_dmg_dollars)) + geom_bar(fill = "darkslategray4", stat = "identity") + coord_flip() + scale_x_discrete(limits = rev(top_crop_dmg$EVTYPE[1:20])) + labs(x = "Weather Event", y = "Total Cost in Dollars (1950 - 2007)", title = "Top 20 Weather Events That Caused the\n Greatest Cost in Crop Damage")

library(gridExtra)
grid.arrange(p3, p4, ncol = 2)
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 

## Conclusions


Looking at the graphs above, it is clear that Tornadoes are the most dangerous weather events for human health -- they have cost the USA over 91,000 injuries and over 5,600 deaths since 1950.    

In addition, flooding and drought have been the most expensive weather events in the USA since 1950. Flooding has cost more than 144 Billion in property damages since 1950. Drought has been the most expensive weather event for crop damage, costing more than 13.9 Billion since 1950.   
                 

.   
.   
.   
.   
.   
.   

    
       






















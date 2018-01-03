+++
date = "2017-12-28T19:32:52-06:00"
draft = false
title = "Economic and Health Impact of Severe Weather Events"

+++

## Synopsis

This post aims to explore the NOAA Storm Database and answer some basic questions about severe weather events focusing on their economic and health impact. The U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage. The events in the database start in the year 1950 and end in November 2011. This report concludes the following:

* Tornadoes, excessive heat and flash floods are primary causes of fatalities. 
* Injuries are mostly caused by tornadoes. The data shows that tornadoes alone resulted in more than 90,000 injuries while all other events caused less than 10,000 injuries.
* In addition, floods cause most economic damage, followed by hurricanes, tornadoes and storms. These four events alone caused damages in excess of 300 Billion US Dollars.

## Data Processing
### Loading Libraries


{{<highlight R>}}
rm(list = ls())
library(lubridate)
library(dplyr)
library(ggplot2)
{{</highlight>}}

Now we are going to import data using `download.file` command and then this data is read into data frame using `read.csv` command. After reading the data, date variable (BGN_DATE) is transformed into 'date' class while event type (EVTYPE) is transformed into factor. The data is finally stored as tibble in `data1`.


{{<highlight R>}}
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
download.file(url,"./datadown.csv.bz2")
data <- read.csv("datadown.csv.bz2",stringsAsFactors = FALSE)

data$BGN_DATE <- mdy_hms(data$BGN_DATE)
data$EVTYPE <- as.factor(data$EVTYPE)

data1 <- tbl_df(data)
{{</highlight>}}

## Results

### Data analysis to find events causing most fatalities
We will first process the data to find events that caused most fatalities. We use `dplyr` package functions for this to first group the data by event type followed by adding fatalities and finally filtering events that caused 90% of the total fatalities.


{{<highlight R>}}
fatal <- data1  %>% 
        group_by(EVTYPE) %>% 
        summarize(FATALITIES=sum(FATALITIES)) %>% 
        arrange(desc(FATALITIES)) %>% 
        select(EVTYPE,FATALITIES) %>% 
        mutate(FATALITIES_CUM = cumsum(FATALITIES)) %>%
        filter(FATALITIES_CUM < 0.9*sum(FATALITIES))

ggplot(data=fatal,aes(x=reorder(EVTYPE,FATALITIES),y=FATALITIES)) + 
        geom_bar(stat = "identity",fill="steelblue4") + 
        coord_flip() + 
        labs(title="Events Causing 90% of Fatalities",
             y = "Number of Fatalities",
             x = "Type of Events")
{{</highlight>}}

{{<figure src="../images/fatalities-1.png" width="90%" >}}

The plot shows that tornadoes are the largest cause of fatalities resulting in more than 5500 deaths. These are followed by excessive heat causing close to 2000 deaths. Other events causing large fatalities include flash floods, lightning and thunderstorm winds.

### Data analysis to find events causing most injuries
Processing data from injuries follows the same method described earlier to process fatalities data.


{{<highlight R>}}
injury <- data1  %>% 
        group_by(EVTYPE) %>% 
        summarize(INJURIES=sum(INJURIES)) %>% 
        arrange(desc(INJURIES)) %>% 
        select(EVTYPE,INJURIES) %>% 
        mutate(INJURIES_CUM = cumsum(INJURIES)) %>%
        filter(INJURIES_CUM < 0.9*sum(INJURIES))

ggplot(data=injury,aes(x=reorder(EVTYPE,INJURIES),y=INJURIES)) + 
        geom_bar(stat = "identity",fill="palegreen4") + 
        coord_flip() +
        labs(title="Events Causing 90% of Injuries",
                y = "Number of Injuries",
                x = "Type of Events")
{{</highlight>}}

{{<figure src="../images/injuries-1.png" width="90%" >}}

It is clear from the above plot that tornadoes are leading causes of injuries. These are followed by thunderstorm winds, floods, excessive heat and lightning as other major events causing injuries.

### Data analysis to find events causing most economic damage
Economic damage has variety of units that we need to consolidate first. For this we want to work with only economic variables so we select appropriate columns. We then create a table (using `data.frame`) for key-value pairs of `PROPDMGEXP` and their values. Similar method is applied for `CROPDMGEXP`.


{{<highlight R>}}
data1e <- data1 %>% select(EVTYPE,PROPDMG:CROPDMGEXP)

dmgindx <- unique(c(unique(data1e$PROPDMGEXP),unique(data1e$CROPDMGEXP)))
dmgindx.val <- c(1e3,1e6,0,1e9,1e6,1,10,10,10,0,10,10,10,100,10,100,0,10,10,1e3)
KeyValue <- data.frame(dmgindx,dmgindx.val)
KeyValue$dmgindx <- as.character(KeyValue$dmgindx)
{{</highlight>}}

This table is added to the main data using `merge` command.


{{<highlight R >}}
data1e <- merge(data1e,KeyValue,by.x = "PROPDMGEXP",by.y = "dmgindx")
data1e <- data1e %>% rename(PropDmgExpVal=dmgindx.val)
data1e <- merge(data1e,KeyValue,by.x = "CROPDMGEXP",by.y = "dmgindx")
{{</highlight>}}

Using the unit conversion data calculated earlier, property damage and crop damage data is computed and then added together to find total economic damage.


{{<highlight R>}}
data.econ <- data1e %>% 
        rename(CropDmgExpVal=dmgindx.val) %>% 
        mutate(PropDmgDollars=PROPDMG*PropDmgExpVal) %>% 
        mutate(CropDmgDollars=CROPDMG*CropDmgExpVal) %>% 
        mutate(EconDmg.Billions=(PropDmgDollars+CropDmgDollars)/1e9) %>%
        select(EVTYPE,PROPDMG,PROPDMGEXP,PropDmgExpVal,PropDmgDollars,
               CROPDMG,CROPDMGEXP,CropDmgExpVal,CropDmgDollars,EconDmg.Billions)
{{</highlight>}}

Then cumulative damage is computed after grouping the data  by event type and filtering data that constitutes 90% of the damage.


{{<highlight R>}}
econ <- data.econ  %>% 
        group_by(EVTYPE) %>% 
        summarize(DMG=sum(EconDmg.Billions)) %>% 
        arrange(desc(DMG)) %>% 
        select(EVTYPE,DMG) %>% 
        mutate(DMG_CUM = cumsum(DMG)) %>%
        filter(DMG_CUM < 0.9*sum(DMG))

ggplot(data=econ,aes(x=reorder(EVTYPE,DMG),y=DMG)) + 
        geom_bar(stat = "identity",fill="tomato4") + 
        coord_flip() + 
        labs(title="Events Causing 90% of Economic Damage",
                y = "Economic Damage (Billions of US Dollars)",
                x = "Type of Events")
{{</highlight>}}

{{<figure src="../images/econ-1.png" width="90%" >}}

The plot above shows that floods are most severe events as far as economic damage is concerned. Other events that resulted in large economic damage include hurricanes, tornadoes and storm surges.

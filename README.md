## Welcome to Boston Crime Data Analysis

---
author: "Abhishek Raj, Sarthak Batra and Parthiv Khaund"
output: html_notebook
---

## Final Project 


## Front Matter
```{r}
# clean up workspace environment
rm(list = ls())

# all packages used for the assignment
library(mosaic)
library(tidyverse)
library(leaflet)
library(ggplot2)
```


#### Loading Data

```{r}
crime_stats <- read.csv("crime.csv")
Van_crime_stats <- read.csv("Vancouver_crime_data.csv")
```


#### Summary Statistics
```{r}
str(crime_stats)
```
```{r}
str(Van_crime_stats)
```
```{r}
summary(crime_stats)
```
```{r}
summary(Van_crime_stats)
```
```{r}
crime_stats
```
```{r}
Van_crime_stats
```
### Data Wrangling

#### Data Cleaning

```{r}
# clean up data that acts as an outlier (error in data)

df <- crime_stats
df <- df[, -c(13, 17)] 
```

```{r}
# creating a new dataframe for only the 2 required columns from the 2nd database 

df1 <- Van_crime_stats
df1 <- df1[, c(1,3)] 
```

#### Districts with the total percentage of crime & total number of crime 

```{r}
# code to find districts based on most number fo incidents 

distric_count<-
  crime_stats %>%
  group_by(DISTRICT)%>%
  summarise(number_of_crimes = n())%>%
  arrange(desc(number_of_crimes))

# selecting top 12 districts based on number of incidents 

distric_count <- distric_count[1:12,]

# calculating the toal number of incidents 

total <- sum(distric_count$number_of_crimes)

# creating a pie chart based on the percentage of incidents for each district  

pie<-
  distric_count%>%
  mutate(percentage_of_total_crime = (number_of_crimes*100)/total)

pie
  
```

#### Pie chart based on the percentage of incidents for each district 

```{r}
ggplot(data = pie ,aes(x= "" , y = percentage_of_total_crime,fill=DISTRICT))+geom_bar(stat = "identity",width=1,color="white")+coord_polar("y", start=0)
```

```{r}
# top 10 latitudes & longitudes based on number on incidents 

lat_long_count<-
  crime_stats %>%
  group_by(Lat,Long)%>%
  summarise(counter = n())%>%
  arrange(desc(counter))

# removing an outlier longitude 

lat_long_count <- lat_long_count[-c(9),] 

# taking top 20 values based on number of incidents 

lat_long_count <- lat_long_count[2:21,]
lat_long_count
```
### Leaflet for top 20 areas of Boston with highest number of incidents 

```{r}
# code to make leafet based on our calculated table above 

boston_map <-
  leaflet(lat_long_count) %>%   # like ggplot()
  addTiles() %>%          # add the map
  addMarkers(clusterOptions = markerClusterOptions())

boston_map
```


```{r}
# code to find highest number of offence group in Boston 

Highest_Boston<-
  df%>%
  select(OFFENSE_CODE_GROUP)%>%
  group_by(OFFENSE_CODE_GROUP)%>%
  summarise(count = n())%>%
  mutate(rankings = rank(count))%>%
  arrange(desc(count))

# Using Control-Flow to rate the top 10 offense types

Highest_Boston$Popularity <- ifelse(Highest_Boston$rankings > max(Highest_Boston$rankings) - 10, "Top_10", "Below")
Highest_Boston
```


```{r}
# code to find to per month number of incidences in Boston

Boston_repeated_months<-
  df%>%
  select(OFFENSE_CODE_GROUP,MONTH)%>%
  count(OFFENSE_CODE_GROUP, MONTH, sort = TRUE)%>%
  select(MONTH,n)

Boston_total_crimes <- aggregate(Boston_repeated_months$n,by=list(MONTH=Boston_repeated_months$MONTH), FUN=sum)

Boston_total_crimes
```

```{r}
# using the top 10 in highest_boston for Regex

regex <- "Motor Vehicle Accident Response|Larceny|Medical Assistance|Investigate Person|Other|Drug Violation|Simple Assault|Vandalism|Verbal Disputes|Towed"

plot<-
  df%>%
  select(OFFENSE_CODE_GROUP,MONTH)%>%
  count(OFFENSE_CODE_GROUP, MONTH, sort = TRUE)%>%
  filter(grepl(pattern = regex, OFFENSE_CODE_GROUP))

plot
```

### 3 variable to graph to show months with highest incidences and the types of incidences 

```{r}
ggplot(data=plot,aes(x=MONTH,y=n ,fill=OFFENSE_CODE_GROUP))+geom_bar(stat='identity',position='stack', width=.9) + scale_x_continuous(name = "Months ", breaks = c(1,2,3,4,5,6,7,8,9,10,11,12)) + scale_y_continuous(name = "Number of Crimes " , breaks = c(0,5000,10000,15000,20000))

```

```{r}
# code to find to per month number of incidences in Boston

Vancouver_repeated_months<-
  df1%>%
  select(TYPE,MONTH)%>%
  count(TYPE, MONTH, sort = TRUE)%>%
  select(MONTH,n)%>%
  arrange(desc(n))

Vancouver_total_crimes <- aggregate(Vancouver_repeated_months$n, by=list(MONTH=Vancouver_repeated_months$MONTH), FUN=sum)

Vancouver_total_crimes
```

```{r}
# joining Vancouver and Boston per month incidences sets 

crimes_list<-left_join(Vancouver_total_crimes,Boston_total_crimes,by="MONTH")
```

```{r}
# code to create Multiple Geom plot to compare boston incidences per month with Vancouver 
plot<-
  crimes_list%>%
  rename(vancouver_crime = x.x,boston_crime = x.y)

plot
```

### Multiple Geom plot to compare boston incidences per month with Vancouver 

```{r}
ggplot(plot)+
  geom_col(aes(x = MONTH, y = vancouver_crime), size = 1, color = "white", fill = "black") +
  geom_line(aes(x = MONTH, y = boston_crime), size = 1.5, color="lightgreen", group = 1) + scale_x_continuous(name = "Months ", breaks = c(1,2,3,4,5,6,7,8,9,10,11,12)) +
  scale_y_continuous(name = "Number of Crimes " , breaks = c(5000,10000,15000,20000,25000,30000,35000,40000,45000))
```




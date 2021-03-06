---
title: "STA304 Problem Set1"
author: "Zefeng Weng"
date: "9/21/2020"
Abstract: "This analyst is briefly explaning about if grandtheft auto occurrence
was influenced by population and the average grandtheft auto occurrancy from 2016
to 2019."
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE)
library(opendatatoronto)
library(tidyverse)
library(ggplot2)
```

Introduction about dataset called "Neighbourhood Crime Rates":
  This is a dataset published by Toronto Police Services, and it is basically 
gathering the frequencies about assault, auto theft, break and enter, bobbery, 
theft over and Homicide in specific locations and the relationship between other 
obervations, which I assumed it exists positive correlation.
  first of all, I need to know how many observations totally in the dataset, and
set "Neighbourhood" as my catogorical data. Then I chose AutoTheft occurrences 
from 2017 to 2019 also the Averate auto thefts from 2014-2019 and percentage 
Change in auto thefts from 2018-2019.
  I am going to pull out only AutoTheft observations from 2016-2019, and plot the
population and AutoTheft average in the diagram, where we could found out these 
two variables have positive linear relation. Then I computed the mean values of
autotheft from 2016 to 2019 and analysis the trend of autotheft.
```{r setup, include=FALSE}
data1_resources= list_package_resources("https://open.toronto.ca/dataset/neighbourhood-crime-rates/")
data1=data1_resources %>% get_resource()
```
The City of Toronto Open Dataset started in 2009 to satisfied the demand of increasing opened data. Since then,it has apprently increasing momentum in civic spaces.

```{r download,warning=FALSE, message=FALSE}
summary(data1$Neighbourhood)
data1$Neighbourhood=factor(data1$Neighbourhood)
data=data1
summary(data1)
#_id,OBJECTID,Neighbourhood and Hood_ID summary are meaningless.
head(data1)
str(data1)

data=
  data %>%
  select(Hood_ID,Neighbourhood, Population, AutoTheft_2017, AutoTheft_2018, AutoTheft_2019,AutoTheft_AVG, AutoTheft_CHG, AutoTheft_Rate_2019)

summary(data)
head(data)
str(data)
```
The variables we focus on are the Population, AutoTheft from 2017 to 2019, average AutoTheft and AutoTheft rate in 2019.
There are totally 10 variables in summary, among them, Hood_ID, Neighbourhood and geometry aren't valid data to observe; thus I factored "Neighbourhood" variable.
```{r}
ggplot(data)+geom_histogram(aes(x=Population))+theme_bw()
# We found out that the histogram is extremely skewwed. 
ggplot(data)+geom_point(aes(x=Population, y=AutoTheft_AVG))+theme_bw()

m=mean(data$Population)
data=
  data %>% filter(Population>m)

data %>%
  ggplot(aes(x=Population,y=AutoTheft_2017,color=Neighbourhood))+
  geom_point()

r=subset(data1,Neighbourhood=="South Riverdale")

df=data.frame(year=c(2016,2017,2018,2019),Autotheft=c(r$AutoTheft_2016,r$AutoTheft_2017,r$AutoTheft_2018,r$AutoTheft_2019))

ggplot(data=df,aes(x=year,y=Autotheft))+geom_point()+geom_line()
```

Another perspective from observing data. I am going to see if the average of 
grandtheft auto is increasing or decreasing.
```{r}
cutoff=as.vector(quantile(data1$Population, probs=c(0.25,0.5,0.75)))

g1=subset(data1,Population<=cutoff[1])
g1$group=1

g2=subset(data1,Population<=cutoff[2]&Population>=cutoff[1])
g2$group=2

g3=subset(data1,Population<=cutoff[3]&Population>=cutoff[2])
g3$group=3

g4=subset(data1,Population>cutoff[3])
g4$group=4

group.data=rbind(g1[1,],g2[1,],g3[1,],g4[1,])

str(group.data)
group.data=data.frame(group.data)

group.data=
  group.data %>%
  dplyr::select(AutoTheft_2016, AutoTheft_2017, AutoTheft_2018,AutoTheft_2019, group)

autotheft=c(group.data$AutoTheft_2016,group.data$AutoTheft_2017,group.data$AutoTheft_2018,group.data$AutoTheft_2019)
year=c(rep(2016,4),rep(2017,4),rep(2018,4),rep(2019,4))
g=rep(group.data$group,4)

df=data.frame(autotheft,year,group=factor(g))

ggplot(data=df,aes(x=year,y=autotheft,color=group))+geom_point()+geom_line()+theme_bw()

g1mean=c(mean(g1$AutoTheft_2016),mean(g1$AutoTheft_2017),mean(g1$AutoTheft_2018),mean(g1$AutoTheft_2019))
g2mean=c(mean(g2$AutoTheft_2016),mean(g2$AutoTheft_2017),mean(g2$AutoTheft_2018),mean(g2$AutoTheft_2019))
g3mean=c(mean(g3$AutoTheft_2016),mean(g3$AutoTheft_2017),mean(g3$AutoTheft_2018),mean(g3$AutoTheft_2019))
g4mean=c(mean(g4$AutoTheft_2016),mean(g4$AutoTheft_2017),mean(g4$AutoTheft_2018),mean(g4$AutoTheft_2019))
autotheft=c(g1mean,g2mean,g3mean,g4mean)
```

```{r}
data1$diff1819 = data1$AutoTheft_2019-data1$AutoTheft_2018
ggplot(data1)+geom_histogram(aes(x=diff1819))+theme_bw()
mean1819=mean(data1$diff1819)

data1$diff1718 = data1$AutoTheft_2018-data1$AutoTheft_2017
ggplot(data1)+geom_histogram(aes(x=diff1718))+theme_bw()
mean1718=mean(data1$diff1718)

data1$diff1617 = data1$AutoTheft_2017-data1$AutoTheft_2016
ggplot(data1)+geom_histogram(aes(x=diff1617))+theme_bw()
mean1617=mean(data1$diff1617)
```

In general, I found that the grandtheft auto occurrence has such a strong correlation with population,
and they are showing a linearly regression, which means higher grandtheft auto occurrence in higher
population region. However, since 2018 to 2019, the grandtheft auto hasn't increased obviously; from
my perspective, it might be affected by increase the strength of investigating from Toronto police
department and the increasing awareness of people.




# references

- Open Data Dataset. (n.d.). Retrieved September 23, 2020, from https://open.toronto.ca/dataset/neighbourhood-crime-rates/
- R Core Team (2020). R: A language and environment for statistical computing. R, Foundation for Statistical Computing, Vienna, Austria. URL, https://www.R-project.org/.
- JJ Allaire and Yihui Xie and Jonathan McPherson and Javier Luraschi and Kevin Ushey and Aron Atkins and Hadley Wickham and Joe Cheng and Winston Chang and Richard Iannone (2020). rmarkdown: Dynamic Documents for R. R package version 2.3. URL https://rmarkdown.rstudio.com.
- Hadley Wickham, Jim Hester and Winston Chang (2020). devtools: Tools to Make Developing R Packages Easier. https://devtools.r-lib.org/,https://github.com/r-lib/devtools. 

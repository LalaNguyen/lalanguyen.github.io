---
layout: post
title:  "Classifying Demand Real-Estate Market"
date:   2014-12-10
categories: jekyll update
permalink: classifying-demand-real-estate-market
---
# Tools
  + Programing Language: Rscript, Javascript
  + Data Object: Prospect
  + [RStudio][rstudio] for statistics.

# Analytical Objectives
Through out the blog, It's important to define our goals by answering these following questions:

1. How are the demands of rental in Industry in term of country ?
2. Visualizing difference in rental areas desired among different companies.
  
# 1. Extracting Data 

Our dataset today is not comprehensive, that leaves me quite a lot of conversions needed to be done. To begin with, we will break down individual files and observe what kinds of data can be valuable to us. In specific, we might be interested in `Industrial-Active`, promising fields are `Company Name`, `Country` and `Area Required` (since they have less empty cells than the other, that also explains why a usable database should not contain `null values` and `multiple formats` within each column, unless you want data analysts to spend time on conversions and filtering null values.)

Let's prepare data for our first experiment:

{% highlight r %}
mydata=read.csv("/../datasets/2014-12-10-classifying-real-estate-prospect-demand/Industrial - Active-Table 1.csv")
{% endhighlight %}



{% highlight text %}
## Warning in file(file, "rt"): cannot open file
## '/../datasets/2014-12-10-classifying-real-estate-prospect-demand/Industrial
## - Active-Table 1.csv': No such file or directory
{% endhighlight %}



{% highlight text %}
## Error in file(file, "rt"): cannot open the connection
{% endhighlight %}



{% highlight r %}
cdata<-data.frame(Comp.name=mydata$X.1,Country=mydata$X.2,Area=mydata$X.5)
{% endhighlight %}



{% highlight text %}
## Error in data.frame(Comp.name = mydata$X.1, Country = mydata$X.2, Area = mydata$X.5): object 'mydata' not found
{% endhighlight %}



{% highlight r %}
#Filtering meaningful data rows 
cdata<-cdata[11:40,]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
cdata=cdata[-13,]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
#Remove unecessary column from data.frame
rownames(cdata)<-NULL
{% endhighlight %}



{% highlight text %}
## Error in rownames(cdata) <- NULL: object 'cdata' not found
{% endhighlight %}



{% highlight r %}
names(cdata)<-c("Company","Country","Area")
{% endhighlight %}



{% highlight text %}
## Error in names(cdata) <- c("Company", "Country", "Area"): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
#As csv file's elements are all factor type and numerical value are usually presented with ',',it's better to strip off the commar and convert to int for further processing.
cdata$Area=as.numeric(gsub(',','',as.character(cdata$Area)))
{% endhighlight %}



{% highlight text %}
## Error in gsub(",", "", as.character(cdata$Area)): object 'cdata' not found
{% endhighlight %}
# 2. Visualizing 

Largest demand is notably comming from Bericap(`Germany`), with highest 5200sqm required area is recorded. This area is nearly two time bigger than required areas by many companies which come from `Japan`. 

{% highlight r %}
require(methods)
require(rCharts)
require(knitr)
require(reshape2)
{% endhighlight %}



{% highlight text %}
## Warning: package 'reshape2' was built under R version 3.1.2
{% endhighlight %}



{% highlight r %}
d1<-dPlot(y="Company", x="Area",data=cdata, groups="Country",type="bar")
{% endhighlight %}



{% highlight text %}
## Error in getLayer.default(...): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
d1$yAxis(type="addCategoryAxis", orderRule="Area")
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd1' not found
{% endhighlight %}



{% highlight r %}
d1$xAxis(type="addMeasureAxis")
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd1' not found
{% endhighlight %}



{% highlight r %}
d1$set(width = 700)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd1' not found
{% endhighlight %}



{% highlight r %}
d1$show("iframesrc",cdn=TRUE)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd1' not found
{% endhighlight %}
---

Ah, talking about Japan, we can look at by light blue bar representing JP companies from the above chart. Despite having less demands in size of area than `Korea` and `Germany` individually, JP still is promising client as they are completely outstanding in term of total area demanded. Aggregating all companies by country would produce clearer observation in the following graph.


{% highlight r %}
d2=dPlot(y="Country", x="Area",data=cdata, groups="Company",type="bar")
{% endhighlight %}



{% highlight text %}
## Error in getLayer.default(...): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
d2$yAxis(type="addCategoryAxis", orderRule="Area")
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd2' not found
{% endhighlight %}



{% highlight r %}
d2$xAxis(type="addMeasureAxis")
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd2' not found
{% endhighlight %}



{% highlight r %}
d2$show("iframesrc",cdn=TRUE)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd2' not found
{% endhighlight %}

---

Same results can be obtained from the bubble chart. Bubles with larger area will float higher than the rest. 

{% highlight r %}
n1 <- rPlot(Area ~ Country, data = cdata, color="Company" ,type = "point")
{% endhighlight %}



{% highlight text %}
## Error in eval(i, data, env): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
n1$addControls("x", value = "Country", values = names(cdata))
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}



{% highlight r %}
n1$addControls("color", value = "Country", values = names(cdata))
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}



{% highlight r %}
n1$show("iframesrc",cdn=TRUE)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}

---
Let's move to Inactive sheet.

{% highlight text %}
## Error in file(file, "rt"): cannot open the connection
{% endhighlight %}



{% highlight text %}
## Error in data.frame(Enquiry_Date = mydata$X, Company = mydata$X.1, Country = mydata$X.2, : object 'mydata' not found
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'cdata' not found
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'cdata' not found
{% endhighlight %}



{% highlight text %}
## Error in rownames(cdata) <- NULL: object 'cdata' not found
{% endhighlight %}



{% highlight text %}
## Error in gsub(",", "", as.character(cdata$Area)): object 'cdata' not found
{% endhighlight %}



{% highlight text %}
## Error in cdata[116, 4] = 1000: object 'cdata' not found
{% endhighlight %}


{% highlight r %}
d2=dPlot(y="Country", x="Area",data=cdata, groups="Company",type="bar")
{% endhighlight %}



{% highlight text %}
## Error in getLayer.default(...): object 'cdata' not found
{% endhighlight %}



{% highlight r %}
d2$yAxis(type="addCategoryAxis", orderRule="Area")
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd2' not found
{% endhighlight %}



{% highlight r %}
d2$xAxis(type="addMeasureAxis")
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd2' not found
{% endhighlight %}



{% highlight r %}
d2$show("iframesrc",cdn=TRUE)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd2' not found
{% endhighlight %}



{% highlight text %}
## Warning in file(file, "rt"): cannot open file
## '/../datasets/2014-12-10-classifying-real-estate-prospect-demand/industrial_data.csv':
## No such file or directory
{% endhighlight %}



{% highlight text %}
## Error in file(file, "rt"): cannot open the connection
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'x' not found
{% endhighlight %}



{% highlight text %}
## Error in melt(x, value.name = "Area", variable.name = "Activity", id.vars = c("Country")): object 'x' not found
{% endhighlight %}



{% highlight text %}
## Error in getLayer.default(...): object 'x.melt' not found
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}
To compare Inactive vs Active by Country, I find it useful to convey idea using pyramid chart from [Walkerke's Blog][walker]. Then, we plot the chart. It's clearly can be seen that not many demands are actually passed the negotiation state at the moment this report was produced. However, the outstanding inactive deals might signal the potential of profitable revenues for company in the future.


{% highlight r %}
n1$chart(color = c('silver','red'))
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}



{% highlight r %}
n1$chart(stacked = TRUE)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}



{% highlight r %}
n1$show("iframesrc",cdn=TRUE)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'n1' not found
{% endhighlight %}

[walker]: http://walkerke.github.io/2014/06/rcharts-pyramids/

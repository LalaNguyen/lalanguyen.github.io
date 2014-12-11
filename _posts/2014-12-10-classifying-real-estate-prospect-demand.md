---
layout: post
title:  "Classifying Demand Real-Estate Market"
date:   2014-12-10
categories: jekyll update
permalink: classifying-demand-real-estate-market
---
#Tools
***
  + Programing Language: Rscript, Javascript
  + Data Object: Prospect
  + [RStudio][rstudio] for statistics.
  
# Analytical Objectives
---
Through out the blog, It's important to define our goals by answering these following questions:

1. How are the demands of rental in Industry in term of country ?
2. Visualizing difference in rental areas desired among different companies.
  
# Extracting Data 
---
Our dataset today is not comprehensive, that leaves me quite a lot of conversions needed to be done. To begin with, we will break down individual files and observe what kinds of data can be valuable to us. In specific, we might be interested in `Industrial-Active`, promising fields are `Company Name`, `Country` and `Area Required` (since they have less empty cells than the other, that also explains why a usable database should not contain `null values` and `multiple formats` within each column, unless you want data analysts to spend time on conversions and filtering null values.)

Let's prepare data for our first experiment:

{% highlight r %}
mydata=read.csv("/Users/mac/Development/env/lalanguyen.github.io/datasets/2014-12-10-classifying-real-estate-prospect-demand/Industrial - Active-Table 1.csv")
cdata<-data.frame(Comp.name=mydata$X.1,Country=mydata$X.2,Area=mydata$X.5)
#Filtering meaningful data rows 
cdata<-cdata[11:40,]
cdata=cdata[-13,]
#Remove unecessary column from data.frame
rownames(cdata)<-NULL
names(cdata)<-c("Company","Country","Area")
#As csv file's elements are all factor type and numerical value are usually presented with ',',it's better to strip off the commar and convert to int for further processing.
cdata$Area=as.numeric(gsub(',','',as.character(cdata$Area)))
{% endhighlight %}
# Visualizing 
---
Largest demand is notably comming from Bericap(`Germany`), with highest 5200sqm required area is recorded. This area is nearly two time bigger than required areas by many companies which come from `Japan`. 

{% highlight r %}
require(methods)
require(rCharts)
require(knitr)
require(reshape2)
d1<-dPlot(y="Company", x="Area",data=cdata, groups="Country",type="bar")
d1$yAxis(type="addCategoryAxis", orderRule="Area")
d1$xAxis(type="addMeasureAxis")
d1$set(width = 700)
d1$print('chart8', cdn = TRUE)
{% endhighlight %}


<script src="/../js/d3.v3.4.8.min.js" charset="utf-8"></script>
<script type='text/javascript' src="/../js/dimple.v2.1.1.min.js"></script> 
 <style>
  .rChart {
    display: block;
    margin-left: auto; 
    margin-right: auto;
    width: 700px;
    height: 400px;
  }  
  </style>
<div id = 'chart8' class = 'rChart dimple'></div>
<script type="text/javascript">
  var opts = {
 "dom": "chart8",
"width":    700,
"height":    400,
"xAxis": {
 "type": "addMeasureAxis",
"showPercent": false 
},
"yAxis": {
 "type": "addCategoryAxis",
"showPercent": false,
"orderRule": "Area" 
},
"zAxis": [],
"colorAxis": [],
"defaultColors": [],
"layers": [],
"legend": [],
"x": "Area",
"y": "Company",
"groups": "Country",
"type": "bar",
"id": "chart8" 
},
    data = [{"Company":"Kyoei-Seisakusho Manufacturing Co.,Ltd.","Country":"JP","Area":2000},{"Company":"Noegel","Country":"DE","Area":1500},{"Company":"Na Jin","Country":"KR","Area":4000},{"Company":"Tamasu Co.,Ltd","Country":"JP","Area":2000},{"Company":"Digital Age Dental Laboratories ","Country":"US","Area":2000},{"Company":"Pretty Woman ","Country":"US","Area":2000},{"Company":"Tohin Industry Co.,Ltd.","Country":"JP","Area":1000},{"Company":"Cystone","Country":"DE","Area":2000},{"Company":"Japan Plus Vietnam Co. Ltd.","Country":"JP","Area":1040},{"Company":"Checkpoint Systems","Country":"US","Area":1043},{"Company":"Sanshin Chemical Industry Vietnam","Country":"JP","Area":3123},{"Company":"IWK VIET NAM","Country":"JP","Area":1040},{"Company":"Pilosio","Country":"EU","Area":3123},{"Company":"Dian International Trading Ltd  (Ban San Se Garment's partner)","Country":"CN","Area":1043},{"Company":"Asia Industry Co., Ltd","Country":"JP","Area":1500},{"Company":"Intermarine Supply","Country":"SG","Area":2003},{"Company":"Kawamura Sangyo Co.,Ltd.","Country":"JP","Area":1040},{"Company":"Sunleaf","Country":"JP","Area":1040},{"Company":"J-TEC Co.,Ltd.","Country":"JP","Area":1000},{"Company":"Bericap","Country":"DE","Area":5206},{"Company":"Tellbe","Country":"SE","Area":1000},{"Company":"Marusan Kigata Seisakusho","Country":"JP","Area":1000},{"Company":"Tada Plastics Moldco","Country":"JP","Area":2000},{"Company":"Kaneko Sangyo Co.,Ltd.","Country":"JP","Area":2000},{"Company":"Suzuki Denso Co., Ltd","Country":"JP","Area":1000},{"Company":"Daito Special Wire","Country":"JP","Area":2000},{"Company":"Relats","Country":"ES","Area":1000},{"Company":"Gondola Kogyo","Country":"JP","Area":1000},{"Company":"DFG","Country":"IT","Area":1040}];
  var svg = dimple.newSvg("#" + opts.id, opts.width, opts.height);

  var myChart = new dimple.chart(svg, data);
  if (opts.bounds) {
    myChart.setBounds(opts.bounds.x, opts.bounds.y, opts.bounds.width, opts.bounds.height);//myChart.setBounds(80, 30, 480, 330);
  }
  //dimple allows use of custom CSS with noFormats
  if(opts.noFormats) { myChart.noFormats = opts.noFormats; };
  //for markimekko and addAxis also have third parameter measure
  //so need to evaluate if measure provided
  
  //function to build axes
  function buildAxis(position,layer){
    var axis;
    var axisopts;
    if (!layer[position+"Axis"]){
      axisopts = opts[position+"Axis"];
    } else axisopts = layer[position+"Axis"];
    
    if(axisopts.measure) {
      axis = myChart[axisopts.type](position,layer[position],axisopts.measure);
    } else {
      axis = myChart[axisopts.type](position, layer[position]);
    };
    if(!(axisopts.type === "addPctAxis")) axis.showPercent = axisopts.showPercent;
    if (axisopts.orderRule) axis.addOrderRule(axisopts.orderRule);
    if (axisopts.grouporderRule) axis.addGroupOrderRule(axisopts.grouporderRule);  
    if (axisopts.overrideMin) axis.overrideMin = axisopts.overrideMin;
    if (axisopts.overrideMax) axis.overrideMax = axisopts.overrideMax;
    if (axisopts.inputFormat) axis.dateParseFormat = axisopts.inputFormat;
    if (axisopts.outputFormat) axis.tickFormat = axisopts.outputFormat;    
    return axis;
  };
  
  var c = null;
  if(d3.keys(opts.colorAxis).length > 0) {
    c = myChart[opts.colorAxis.type](opts.colorAxis.colorSeries,opts.colorAxis.palette) ;
  }
  
  //allow manipulation of default colors to use with dimple
  if(opts.defaultColors.length) {
    opts.defaultColors = opts.defaultColors[0];
    if (typeof(opts.defaultColors) == "function") {
      //assume this is a d3 scale
      //for now loop through first 20 but need a better way to handle
      defaultColorsArray = [];
      for (var n=0;n<20;n++) {
        defaultColorsArray.push(opts.defaultColors(n));
      };
      opts.defaultColors = defaultColorsArray;
    }
    opts.defaultColors.forEach(function(d,i) {
      opts.defaultColors[i] = new dimple.color(d);
    })
    myChart.defaultColors = opts.defaultColors;
  }  
  
  //do series
  //set up a function since same for each
  //as of now we have x,y,groups,data,type in opts for primary layer
  //and other layers reside in opts.layers
  function buildSeries(layer, hidden){
    //inherit from primary layer if not intentionally changed or xAxis, yAxis, zAxis null
    if (!layer.xAxis) layer.xAxis = opts.xAxis;    
    if (!layer.yAxis) layer.yAxis = opts.yAxis;
    if (!layer.zAxis) layer.zAxis = opts.zAxis;
    
    var x = buildAxis("x", layer);
    x.hidden = hidden;
    
    var y = buildAxis("y", layer);
    y.hidden = hidden;
    
    //z for bubbles
    var z = null;
    if (!(typeof(layer.zAxis) === 'undefined') && layer.zAxis.type){
      z = buildAxis("z", layer);
    };
    
    //here think I need to evaluate group and if missing do null
    //as the group argument
    //if provided need to use groups from layer
    var s = new dimple.series(myChart, null, x, y, z, c, null, dimple.plot[layer.type], dimple.aggregateMethod.avg, dimple.plot[layer.type].stacked);
    
    //as of v1.1.4 dimple can use different dataset for each series
    if(layer.data){
      //convert to an array of objects
      //avoid lodash for now
      datakeys = d3.keys(layer.data)
      layer.dataarray = layer.data[datakeys[1]].map(function(d,i){
        var tempobj = {}
        datakeys.forEach(function(key){
          tempobj[key] = layer.data[key][i]
        })
        return tempobj
      })
      s.data = layer.dataarray;
    }
    
    //for measure axis dimple sorts at the series level not at axis level
    ['x','y'].map(function(ax){
      if( layer[ax + 'Axis'].type=="addMeasureAxis" && layer[ax + 'Axis'].orderRule ){
        if( typeof layer[ax + 'Axis'].orderRule == "string" ){
          s.addOrderRule( layer[ax + 'Axis'].orderRule );
        } else if ( typeof layer[ax + 'Axis'].orderRule == "object" ) {
          s._orderRules = layer[ax + 'Axis'].orderRule;
        }
      }
    })
    
    if(layer.hasOwnProperty("groups")) {
      s.categoryFields = (typeof layer.groups === "object") ? layer.groups : [layer.groups];
      //series offers an aggregate method that we will also need to check if available
      //options available are avg, count, max, min, sum
    }
    if (!(typeof(layer.aggregate) === 'undefined')) {
      s.aggregate = eval(layer.aggregate);
    }
    if (!(typeof(layer.lineWeight) === 'undefined')) {
      s.lineWeight = layer.lineWeight;
    }
    if (!(typeof(layer.lineMarkers) === 'undefined')) {
      s.lineMarkers = layer.lineMarkers;
    }
    if (!(typeof(layer.barGap) === 'undefined')) {
      s.barGap = layer.barGap;
    }    
    if (!(typeof(layer.interpolation) === 'undefined')) {
      s.interpolation = layer.interpolation;
    } 
    
    myChart.series.push(s);
    
    /*placeholder fix domain of primary scale for new series data
    //not working right now but something like this
    //for now just use overrideMin and overrideMax from rCharts
    for( var i = 0; i<2; i++) {
      if (!myChart.axes[i].overrideMin) {
        myChart.series[0]._axisBounds(i==0?"x":"y").min = myChart.series[0]._axisBounds(i==0?"x":"y").min < s._axisBounds(i==0?"x":"y").min ? myChart.series[0]._axisBounds(i==0?"x":"y").min : s._axisBounds(i==0?"x":"y").min;
      }
      if (!myChart.axes[i].overrideMax) {  
        myChart.series[0]._axisBounds(i==0?"x":"y")._max = myChart.series[0]._axisBounds(i==0?"x":"y").max > s._axisBounds(i==0?"x":"y").max ? myChart.series[0]._axisBounds(i==0?"x":"y").max : s._axisBounds(i==0?"x":"y").max;
      }
      myChart.axes[i]._update();
    }
    */
      
    
    return s;
  };
  
  buildSeries(opts, false);
  if (opts.layers.length > 0) {
    opts.layers.forEach(function(layer){
      buildSeries(layer, true);
    })
  }
  //unsure if this is best but if legend is provided (not empty) then evaluate
  if(d3.keys(opts.legend).length > 0) {
    var l =myChart.addLegend();
    d3.keys(opts.legend).forEach(function(d){
      l[d] = opts.legend[d];
    });
  }
  //quick way to get this going but need to make this cleaner
  if(opts.storyboard) {
    myChart.setStoryboard(opts.storyboard);
  };
  myChart.draw();

</script>
Ah, talking about Japan, we can look at by light blue bar representing JP companies from the above chart. Despite having less demands in size of area than `Korea` and `Germany` individually, JP still is promising client as they are completely outstanding in term of total area demanded. Aggregating all companies by country would produce clearer observation in the following graph.


{% highlight r %}
d2=dPlot(y="Country", x="Area",data=cdata, groups="Company",type="bar")
d2$yAxis(type="addCategoryAxis", orderRule="Area")
d2$xAxis(type="addMeasureAxis")
{% endhighlight %}
<div id = 'chart9' class = 'rChart dimple'></div>
<script type="text/javascript">
  var opts = {
 "dom": "chart9",
"width":    800,
"height":    400,
"xAxis": {
 "type": "addMeasureAxis",
"showPercent": false 
},
"yAxis": {
 "type": "addCategoryAxis",
"showPercent": false,
"orderRule": "Area" 
},
"zAxis": [],
"colorAxis": [],
"defaultColors": [],
"layers": [],
"legend": [],
"x": "Area",
"y": "Country",
"groups": "Company",
"type": "bar",
"id": "chart9" 
},
    data = [{"Company":"Kyoei-Seisakusho Manufacturing Co.,Ltd.","Country":"JP","Area":2000},{"Company":"Noegel","Country":"DE","Area":1500},{"Company":"Na Jin","Country":"KR","Area":4000},{"Company":"Tamasu Co.,Ltd","Country":"JP","Area":2000},{"Company":"Digital Age Dental Laboratories ","Country":"US","Area":2000},{"Company":"Pretty Woman ","Country":"US","Area":2000},{"Company":"Tohin Industry Co.,Ltd.","Country":"JP","Area":1000},{"Company":"Cystone","Country":"DE","Area":2000},{"Company":"Japan Plus Vietnam Co. Ltd.","Country":"JP","Area":1040},{"Company":"Checkpoint Systems","Country":"US","Area":1043},{"Company":"Sanshin Chemical Industry Vietnam","Country":"JP","Area":3123},{"Company":"IWK VIET NAM","Country":"JP","Area":1040},{"Company":"Pilosio","Country":"EU","Area":3123},{"Company":"Dian International Trading Ltd  (Ban San Se Garment's partner)","Country":"CN","Area":1043},{"Company":"Asia Industry Co., Ltd","Country":"JP","Area":1500},{"Company":"Intermarine Supply","Country":"SG","Area":2003},{"Company":"Kawamura Sangyo Co.,Ltd.","Country":"JP","Area":1040},{"Company":"Sunleaf","Country":"JP","Area":1040},{"Company":"J-TEC Co.,Ltd.","Country":"JP","Area":1000},{"Company":"Bericap","Country":"DE","Area":5206},{"Company":"Tellbe","Country":"SE","Area":1000},{"Company":"Marusan Kigata Seisakusho","Country":"JP","Area":1000},{"Company":"Tada Plastics Moldco","Country":"JP","Area":2000},{"Company":"Kaneko Sangyo Co.,Ltd.","Country":"JP","Area":2000},{"Company":"Suzuki Denso Co., Ltd","Country":"JP","Area":1000},{"Company":"Daito Special Wire","Country":"JP","Area":2000},{"Company":"Relats","Country":"ES","Area":1000},{"Company":"Gondola Kogyo","Country":"JP","Area":1000},{"Company":"DFG","Country":"IT","Area":1040}];
  var svg = dimple.newSvg("#" + opts.id, opts.width, opts.height);

  var myChart = new dimple.chart(svg, data);
  if (opts.bounds) {
    myChart.setBounds(opts.bounds.x, opts.bounds.y, opts.bounds.width, opts.bounds.height);//myChart.setBounds(80, 30, 480, 330);
  }
  //dimple allows use of custom CSS with noFormats
  if(opts.noFormats) { myChart.noFormats = opts.noFormats; };
  //for markimekko and addAxis also have third parameter measure
  //so need to evaluate if measure provided
  
  //function to build axes
  function buildAxis(position,layer){
    var axis;
    var axisopts;
    if (!layer[position+"Axis"]){
      axisopts = opts[position+"Axis"];
    } else axisopts = layer[position+"Axis"];
    
    if(axisopts.measure) {
      axis = myChart[axisopts.type](position,layer[position],axisopts.measure);
    } else {
      axis = myChart[axisopts.type](position, layer[position]);
    };
    if(!(axisopts.type === "addPctAxis")) axis.showPercent = axisopts.showPercent;
    if (axisopts.orderRule) axis.addOrderRule(axisopts.orderRule);
    if (axisopts.grouporderRule) axis.addGroupOrderRule(axisopts.grouporderRule);  
    if (axisopts.overrideMin) axis.overrideMin = axisopts.overrideMin;
    if (axisopts.overrideMax) axis.overrideMax = axisopts.overrideMax;
    if (axisopts.inputFormat) axis.dateParseFormat = axisopts.inputFormat;
    if (axisopts.outputFormat) axis.tickFormat = axisopts.outputFormat;    
    return axis;
  };
  
  var c = null;
  if(d3.keys(opts.colorAxis).length > 0) {
    c = myChart[opts.colorAxis.type](opts.colorAxis.colorSeries,opts.colorAxis.palette) ;
  }
  
  //allow manipulation of default colors to use with dimple
  if(opts.defaultColors.length) {
    opts.defaultColors = opts.defaultColors[0];
    if (typeof(opts.defaultColors) == "function") {
      //assume this is a d3 scale
      //for now loop through first 20 but need a better way to handle
      defaultColorsArray = [];
      for (var n=0;n<20;n++) {
        defaultColorsArray.push(opts.defaultColors(n));
      };
      opts.defaultColors = defaultColorsArray;
    }
    opts.defaultColors.forEach(function(d,i) {
      opts.defaultColors[i] = new dimple.color(d);
    })
    myChart.defaultColors = opts.defaultColors;
  }  
  
  //do series
  //set up a function since same for each
  //as of now we have x,y,groups,data,type in opts for primary layer
  //and other layers reside in opts.layers
  function buildSeries(layer, hidden){
    //inherit from primary layer if not intentionally changed or xAxis, yAxis, zAxis null
    if (!layer.xAxis) layer.xAxis = opts.xAxis;    
    if (!layer.yAxis) layer.yAxis = opts.yAxis;
    if (!layer.zAxis) layer.zAxis = opts.zAxis;
    
    var x = buildAxis("x", layer);
    x.hidden = hidden;
    
    var y = buildAxis("y", layer);
    y.hidden = hidden;
    
    //z for bubbles
    var z = null;
    if (!(typeof(layer.zAxis) === 'undefined') && layer.zAxis.type){
      z = buildAxis("z", layer);
    };
    
    //here think I need to evaluate group and if missing do null
    //as the group argument
    //if provided need to use groups from layer
    var s = new dimple.series(myChart, null, x, y, z, c, null, dimple.plot[layer.type], dimple.aggregateMethod.avg, dimple.plot[layer.type].stacked);
    
    //as of v1.1.4 dimple can use different dataset for each series
    if(layer.data){
      //convert to an array of objects
      //avoid lodash for now
      datakeys = d3.keys(layer.data)
      layer.dataarray = layer.data[datakeys[1]].map(function(d,i){
        var tempobj = {}
        datakeys.forEach(function(key){
          tempobj[key] = layer.data[key][i]
        })
        return tempobj
      })
      s.data = layer.dataarray;
    }
    
    //for measure axis dimple sorts at the series level not at axis level
    ['x','y'].map(function(ax){
      if( layer[ax + 'Axis'].type=="addMeasureAxis" && layer[ax + 'Axis'].orderRule ){
        if( typeof layer[ax + 'Axis'].orderRule == "string" ){
          s.addOrderRule( layer[ax + 'Axis'].orderRule );
        } else if ( typeof layer[ax + 'Axis'].orderRule == "object" ) {
          s._orderRules = layer[ax + 'Axis'].orderRule;
        }
      }
    })
    
    if(layer.hasOwnProperty("groups")) {
      s.categoryFields = (typeof layer.groups === "object") ? layer.groups : [layer.groups];
      //series offers an aggregate method that we will also need to check if available
      //options available are avg, count, max, min, sum
    }
    if (!(typeof(layer.aggregate) === 'undefined')) {
      s.aggregate = eval(layer.aggregate);
    }
    if (!(typeof(layer.lineWeight) === 'undefined')) {
      s.lineWeight = layer.lineWeight;
    }
    if (!(typeof(layer.lineMarkers) === 'undefined')) {
      s.lineMarkers = layer.lineMarkers;
    }
    if (!(typeof(layer.barGap) === 'undefined')) {
      s.barGap = layer.barGap;
    }    
    if (!(typeof(layer.interpolation) === 'undefined')) {
      s.interpolation = layer.interpolation;
    } 
    
    myChart.series.push(s);
    
    /*placeholder fix domain of primary scale for new series data
    //not working right now but something like this
    //for now just use overrideMin and overrideMax from rCharts
    for( var i = 0; i<2; i++) {
      if (!myChart.axes[i].overrideMin) {
        myChart.series[0]._axisBounds(i==0?"x":"y").min = myChart.series[0]._axisBounds(i==0?"x":"y").min < s._axisBounds(i==0?"x":"y").min ? myChart.series[0]._axisBounds(i==0?"x":"y").min : s._axisBounds(i==0?"x":"y").min;
      }
      if (!myChart.axes[i].overrideMax) {  
        myChart.series[0]._axisBounds(i==0?"x":"y")._max = myChart.series[0]._axisBounds(i==0?"x":"y").max > s._axisBounds(i==0?"x":"y").max ? myChart.series[0]._axisBounds(i==0?"x":"y").max : s._axisBounds(i==0?"x":"y").max;
      }
      myChart.axes[i]._update();
    }
    */
      
    
    return s;
  };
  
  buildSeries(opts, false);
  if (opts.layers.length > 0) {
    opts.layers.forEach(function(layer){
      buildSeries(layer, true);
    })
  }
  //unsure if this is best but if legend is provided (not empty) then evaluate
  if(d3.keys(opts.legend).length > 0) {
    var l =myChart.addLegend();
    d3.keys(opts.legend).forEach(function(d){
      l[d] = opts.legend[d];
    });
  }
  //quick way to get this going but need to make this cleaner
  if(opts.storyboard) {
    myChart.setStoryboard(opts.storyboard);
  };
  myChart.draw();

</script>

Let's move to Inactive sheet.



{% highlight r %}
d2=dPlot(y="Country", x="Area",data=cdata, groups="Company",type="bar")
d2$yAxis(type="addCategoryAxis", orderRule="Area")
d2$xAxis(type="addMeasureAxis")
d2$print("barchart",cdn=TRUE)
{% endhighlight %}


<div id = 'barchart' class = 'rChart dimple'></div>
<script type="text/javascript">
  var opts = {
 "dom": "barchart",
"width":    800,
"height":    400,
"xAxis": {
 "type": "addMeasureAxis",
"showPercent": false 
},
"yAxis": {
 "type": "addCategoryAxis",
"showPercent": false,
"orderRule": "Area" 
},
"zAxis": [],
"colorAxis": [],
"defaultColors": [],
"layers": [],
"legend": [],
"x": "Area",
"y": "Country",
"groups": "Company",
"type": "bar",
"id": "barchart" 
},
    data = [{"Enquiry_Date":"17-Nov-14","Company":"Kim Hoa Ltd","Country":"KR","Area":1000},{"Enquiry_Date":"5-May-14","Company":"Sangyo Supply","Country":"JP","Area":500},{"Enquiry_Date":"22-Oct-14","Company":"Mr. Mochizuki","Country":"JP","Area":500},{"Enquiry_Date":"10-May-12","Company":"Inoac Polymer Vietnam Co. , Ltd.","Country":"JP","Area":2003},{"Enquiry_Date":"3-Sep-14","Company":"Erai Vietnam","Country":"FR","Area":1500},{"Enquiry_Date":"20-Aug-14","Company":"Bridge Corporation","Country":"KR","Area":2000},{"Enquiry_Date":"31-Jul-14","Company":"Toda Group","Country":"JP","Area":2000},{"Enquiry_Date":"28-Jul-14","Company":"NE WHE CO., LTD","Country":"KR","Area":2500},{"Enquiry_Date":"17-Jul-14","Company":"Fico Trading ","Country":"VN","Area":2000},{"Enquiry_Date":"15-Jul-14","Company":"CSP Enterprise Ltd","Country":"KR","Area":5000},{"Enquiry_Date":"7-Jul-14","Company":"World Tech Vina","Country":"KR","Area":2000},{"Enquiry_Date":"30-Jun-14","Company":"Dynaplast","Country":"ID","Area":4000},{"Enquiry_Date":"26-Jun-14","Company":"DRS","Country":"KR","Area":2000},{"Enquiry_Date":"23-Jun-14","Company":"SNL","Country":"HK","Area":1000},{"Enquiry_Date":"11-Jun-14","Company":"Yergat Foods","Country":"US","Area":2000},{"Enquiry_Date":"11-Jun-14","Company":"Swisstec Sourcing","Country":"CH","Area":2000},{"Enquiry_Date":"11-Jun-14","Company":"Zephyr","Country":"SG","Area":2000},{"Enquiry_Date":"3-Jun-14","Company":"OKENSEIKO VIETNAM CO., LTD","Country":"JP","Area":2000},{"Enquiry_Date":"23-May-14","Company":"Sin Hwa Dee Foodstuff Industries Pte Ltd","Country":"SG","Area":2000},{"Enquiry_Date":"14-May-14","Company":"Royal Spirit","Country":"TW","Area":1040},{"Enquiry_Date":"14-May-14","Company":"Asean","Country":"VN","Area":2000},{"Enquiry_Date":"14-May-14","Company":"Saydo","Country":"VN","Area":10000},{"Enquiry_Date":"14-May-14","Company":"Thinh Phat ","Country":"MY","Area":2000},{"Enquiry_Date":"May-14","Company":"Premco","Country":"IN","Area":7000},{"Enquiry_Date":"2-May-14","Company":"Base Group","Country":"UK","Area":2000},{"Enquiry_Date":"28-Apr-14","Company":"Likelion","Country":"KR","Area":3000},{"Enquiry_Date":"18-Apr-14","Company":"Sketch-Interior Design ","Country":"DK","Area":2000},{"Enquiry_Date":"14-Apr-14","Company":"Texchem-Pack","Country":"MY","Area":1000},{"Enquiry_Date":"11-Apr-14","Company":"Raw Technologies","Country":"CA","Area":2000},{"Enquiry_Date":"2-Apr-14","Company":"Moda Design","Country":"SG","Area":1000},{"Enquiry_Date":"25-Mar-14","Company":"Wanek Furniture","Country":"US","Area":4000},{"Enquiry_Date":"21-Mar-14","Company":"Yumark ","Country":"TW","Area":3000},{"Enquiry_Date":"7-Mar-14","Company":"Dong A","Country":"VN","Area":1000},{"Enquiry_Date":"6-Mar-14","Company":"Dat Vinh M&E Company Limited","Country":"VN","Area":1000},{"Enquiry_Date":"6-Mar-14","Company":"Morstar Electric","Country":"HK","Area":2000},{"Enquiry_Date":"4-Mar-14","Company":"Seikow Chemical Engineering & Machinery, Ltd.","Country":"JP","Area":1000},{"Enquiry_Date":"4-Mar-14","Company":"Fukuoka Tape","Country":"JP","Area":1000},{"Enquiry_Date":"4-Mar-14","Company":"Phuc Hung Co.","Country":"VN","Area":1000},{"Enquiry_Date":"4-Mar-14","Company":"Infratrol","Country":"US","Area":1000},{"Enquiry_Date":"1-Mar-14","Company":"Toyotsu Vehitecs Vietnam","Country":"JP","Area":2000},{"Enquiry_Date":"24-Feb-14","Company":"DAEHACABLE","Country":"KR","Area":4000},{"Enquiry_Date":"21-Feb-14","Company":"Shimabun Engineering Co.,Ltd","Country":"JP","Area":1000},{"Enquiry_Date":"21-Feb-14","Company":"Koryo ","Country":"KR","Area":4000},{"Enquiry_Date":"17-Feb-14","Company":"Yoshino","Country":"JP","Area":1000},{"Enquiry_Date":"13-Feb-14","Company":"Itokoki","Country":"JP","Area":2000},{"Enquiry_Date":"12-Feb-14","Company":"Sanix","Country":"JP","Area":2000},{"Enquiry_Date":"28-Dec-13","Company":"Meditec Japan","Country":"JP","Area":2000},{"Enquiry_Date":"27-Dec-13","Company":"Tokyo Paint","Country":"JP","Area":2000},{"Enquiry_Date":"19-Dec-13","Company":"Unicity","Country":"US","Area":1000},{"Enquiry_Date":"6-Dec-13","Company":"Fischer Asia","Country":"DE","Area":2000},{"Enquiry_Date":"06-Dec-13","Company":"Hiteso Co., Ltd","Country":"JP","Area":2000},{"Enquiry_Date":"3-Dec-13","Company":"New World Fashion","Country":"CN","Area":5000},{"Enquiry_Date":"30-Nov-13","Company":"Sin\u00a0Huat Le\u00a0Sdn. Bhd","Country":"MY","Area":1000},{"Enquiry_Date":"29-Nov-13","Company":"One prospect from India","Country":"IN","Area":1000},{"Enquiry_Date":"29-Nov-13","Company":"H.B.C International","Country":"US","Area":1000},{"Enquiry_Date":"26-Nov-13","Company":"Hung Minh Co","Country":"CN","Area":1000},{"Enquiry_Date":"20-Nov-13","Company":"Tokyo Light Co","Country":"JP","Area":10000},{"Enquiry_Date":"19-Nov-13","Company":"Yamazaki Corporation","Country":"JP","Area":4000},{"Enquiry_Date":"11-Nov-13","Company":"Thinh Phat Co., Ltd. ","Country":"VN","Area":2000},{"Enquiry_Date":"1-Nov-13","Company":"Furniweb (their Italian customer)","Country":"IT","Area":3000},{"Enquiry_Date":"31-Oct-13","Company":"WB Coatings","Country":"GE","Area":2000},{"Enquiry_Date":"24-Oct-13","Company":"SMK Co., Ltd","Country":"JP","Area":3000},{"Enquiry_Date":"11-Oct-13","Company":"Solimac Group","Country":"TH","Area":1000},{"Enquiry_Date":"11-Oct-13","Company":"Nihonhanda","Country":"JP","Area":1000},{"Enquiry_Date":"11-Oct-13","Company":"Emuge Franken","Country":"DE","Area":1000},{"Enquiry_Date":"11-Oct-13","Company":"Thermotech Engineering (PUNE) - India","Country":"IN","Area":1000},{"Enquiry_Date":"8-Oct-13","Company":"I-tak","Country":"JP","Area":2000},{"Enquiry_Date":"23-Sep-13","Company":"CNA (Korea)","Country":"KR","Area":1000},{"Enquiry_Date":"4-Sep-13","Company":"REDRANGER","Country":"AU","Area":4000},{"Enquiry_Date":"26-Aug-13","Company":"Changse","Country":"CN","Area":2000},{"Enquiry_Date":"23-Aug-13","Company":"Daiwa Kouki","Country":"JP","Area":3000},{"Enquiry_Date":"19-Aug-13","Company":"NSG","Country":"UK","Area":5000},{"Enquiry_Date":"13-Aug-13","Company":"Aironware Fasteners (China)","Country":"CN","Area":3000},{"Enquiry_Date":"9-Aug-13","Company":"One prospect from Chao ","Country":"CN","Area":1000},{"Enquiry_Date":"7-Aug-13","Company":"One JP prospect ","Country":"JP","Area":1000},{"Enquiry_Date":"4-Aug-13","Company":"KIM ANN ENGINEERING PTE LTD","Country":"TH","Area":2000},{"Enquiry_Date":"31-Jul-13","Company":"Kitani Denki","Country":"JP","Area":2000},{"Enquiry_Date":"22-Jul-13","Company":"CJ Corp","Country":"KR","Area":1500},{"Enquiry_Date":"22-Jul-13","Company":"Hanil U.S.G Vina","Country":"KR","Area":1000},{"Enquiry_Date":"10-Jul-13","Company":"Shining Labels \u2013 Hong Kong","Country":"HK","Area":2500},{"Enquiry_Date":"05-Jul-13","Company":"Microtest AG","Country":"CH","Area":1000},{"Enquiry_Date":"02-Jul-13","Company":"Xiamen Unique Bridal","Country":"CN","Area":20000},{"Enquiry_Date":"01-Jul-13","Company":"OTS","Country":"US","Area":1000},{"Enquiry_Date":"26-Jun-13","Company":"KUWAHARA VN","Country":"JP","Area":1800},{"Enquiry_Date":"Jun-13","Company":"Gelan Textile","Country":"CN","Area":2000},{"Enquiry_Date":"14-Jun-13","Company":"Salcomp","Country":"FI","Area":5000},{"Enquiry_Date":"Jun-13","Company":"Astee Horie","Country":"JP","Area":500},{"Enquiry_Date":"May-13","Company":"STC Hongkong","Country":"HK","Area":800},{"Enquiry_Date":"23-Apr-13","Company":"NIKKO","Country":"JP","Area":2000},{"Enquiry_Date":"Apr-13","Company":"Federal Mogul","Country":"US","Area":2500},{"Enquiry_Date":"Apr-13","Company":"Naneu Bags","Country":"US","Area":2500},{"Enquiry_Date":"Apr-13","Company":"D&A Industries","Country":"VN","Area":2000},{"Enquiry_Date":"1-Apr-13","Company":"Kleen-pak","Country":"SG","Area":4000},{"Enquiry_Date":"Mar-13","Company":"Chugai-technos","Country":"JP","Area":500},{"Enquiry_Date":"Mar-13","Company":"Regus","Country":"LU","Area":1000},{"Enquiry_Date":"Feb-13","Company":"PPG Coatings","Country":"US","Area":1000},{"Enquiry_Date":"Feb-13","Company":"Bierens","Country":"NL","Area":1000},{"Enquiry_Date":"18-Jan-13","Company":"United Packaging","Country":"PA","Area":6000},{"Enquiry_Date":"14-Dec-12","Company":"SST Castings","Country":"US","Area":5000},{"Enquiry_Date":"Nov-12","Company":"One premium wine company","Country":"US","Area":500},{"Enquiry_Date":"27-Oct-12","Company":"SHIRAKAWA SEISAKUSHO","Country":"JP","Area":1000},{"Enquiry_Date":"25-Sep-12","Company":"Phu Thai CAT","Country":"VN","Area":4000},{"Enquiry_Date":"Sep-12","Company":"Okatsune","Country":"JP","Area":1000},{"Enquiry_Date":"28-Aug-12","Company":"SSLG Industrial","Country":"HK","Area":7000},{"Enquiry_Date":"20-Aug-12","Company":"Uchihashi Vietnam Co., Ltd.","Country":"JP","Area":2000},{"Enquiry_Date":"9-Aug-12","Company":"Isuzu Dengyo","Country":"JP","Area":1000},{"Enquiry_Date":"06-Aug-12","Company":"Anh Thi Co., Ltd","Country":"VN","Area":3000},{"Enquiry_Date":"31-Jul-12","Company":"KODEN","Country":"JP","Area":6000},{"Enquiry_Date":"10-Jul-12","Company":"Jyohoku Spring ","Country":"JP","Area":1000},{"Enquiry_Date":"6-Jul-12","Company":"Plasticolor","Country":"CA","Area":2000},{"Enquiry_Date":"14-Jun-12","Company":"Nagae Co., Ltd","Country":"JP","Area":1000},{"Enquiry_Date":"12-Jun-12","Company":"Fuji Chemical ","Country":"JP","Area":1000},{"Enquiry_Date":"12-Jun-12","Company":"Fuji Bakelite","Country":"JP","Area":2000},{"Enquiry_Date":"07-Jun-12","Company":"Zalora / Lazada","Country":"VN","Area":1000},{"Enquiry_Date":"06-Jun-12","Company":"(Tecasin Mr. Andy Lee)","Country":"VN","Area":1200},{"Enquiry_Date":"Jun-12","Company":"Toho Koki Co., LTD","Country":"JP","Area":1000},{"Enquiry_Date":"Jun-12","Company":"Amore Pacific","Country":"KR","Area":1000},{"Enquiry_Date":"Jun-12","Company":"Aggreko (Singapore) Pte Ltd","Country":"SG","Area":10000},{"Enquiry_Date":"30-May-12","Company":"DP Consulting","Country":"VN","Area":2000},{"Enquiry_Date":"10-May-12","Company":"Inoac","Country":"JP","Area":1500},{"Enquiry_Date":"May-12","Company":"Naniwa Gosei CO.,LTD","Country":"JP","Area":1000},{"Enquiry_Date":"May-12","Company":"Perennial Design & Build Pte Ltd (Singapore)","Country":"SG","Area":1000},{"Enquiry_Date":"26-Mar-12","Company":"Shanghai Lirong Nickel Screen","Country":"CN","Area":4000},{"Enquiry_Date":"31-Jan-12","Company":"Aica Kogyo","Country":"JP","Area":2000},{"Enquiry_Date":"20-Jan-12","Company":"Pharmascience ","Country":"CA","Area":1000},{"Enquiry_Date":"18-Jan-12","Company":"Vietnam Mobile Telecom Services Co. (VMS)","Country":"VN","Area":1500},{"Enquiry_Date":"30-Nov-11","Company":"Kahoku Lighting Solutions","Country":"JP","Area":1000},{"Enquiry_Date":"25-Oct-11","Company":"Avery Dennison","Country":"US","Area":5000},{"Enquiry_Date":"25-Oct-11","Company":"Avery Dennison","Country":"US","Area":4000},{"Enquiry_Date":"24-Oct-11","Company":"Harada","Country":"JP","Area":1000},{"Enquiry_Date":"14-Oct-11","Company":"Kyouwa Corporation ","Country":"JP","Area":2000},{"Enquiry_Date":"11-Oct-11","Company":"Tontec International ","Country":"HK","Area":1000},{"Enquiry_Date":"30-Sep-11","Company":"Primus International","Country":"US","Area":10000},{"Enquiry_Date":"22-Sep-11","Company":"Chemtech","Country":"IN","Area":5000},{"Enquiry_Date":"7-Jul-11","Company":"Advance Tech Automation ","Country":"SG","Area":1000},{"Enquiry_Date":"28-Jun-11","Company":"Rheem Australia","Country":"AU","Area":7000},{"Enquiry_Date":"14-Jun-11","Company":"RF Micro Devices","Country":"US","Area":32402},{"Enquiry_Date":"8-Jun-11","Company":"Saveri","Country":"DE","Area":4000},{"Enquiry_Date":"7-Apr-11","Company":"Siegwerk","Country":"DE","Area":1000},{"Enquiry_Date":"14-Mar-11","Company":"BTS Wonderful Saigon Electric ","Country":"JP","Area":20000}];
  var svg = dimple.newSvg("#" + opts.id, opts.width, opts.height);

  var myChart = new dimple.chart(svg, data);
  if (opts.bounds) {
    myChart.setBounds(opts.bounds.x, opts.bounds.y, opts.bounds.width, opts.bounds.height);//myChart.setBounds(80, 30, 480, 330);
  }
  //dimple allows use of custom CSS with noFormats
  if(opts.noFormats) { myChart.noFormats = opts.noFormats; };
  //for markimekko and addAxis also have third parameter measure
  //so need to evaluate if measure provided
  
  //function to build axes
  function buildAxis(position,layer){
    var axis;
    var axisopts;
    if (!layer[position+"Axis"]){
      axisopts = opts[position+"Axis"];
    } else axisopts = layer[position+"Axis"];
    
    if(axisopts.measure) {
      axis = myChart[axisopts.type](position,layer[position],axisopts.measure);
    } else {
      axis = myChart[axisopts.type](position, layer[position]);
    };
    if(!(axisopts.type === "addPctAxis")) axis.showPercent = axisopts.showPercent;
    if (axisopts.orderRule) axis.addOrderRule(axisopts.orderRule);
    if (axisopts.grouporderRule) axis.addGroupOrderRule(axisopts.grouporderRule);  
    if (axisopts.overrideMin) axis.overrideMin = axisopts.overrideMin;
    if (axisopts.overrideMax) axis.overrideMax = axisopts.overrideMax;
    if (axisopts.inputFormat) axis.dateParseFormat = axisopts.inputFormat;
    if (axisopts.outputFormat) axis.tickFormat = axisopts.outputFormat;    
    return axis;
  };
  
  var c = null;
  if(d3.keys(opts.colorAxis).length > 0) {
    c = myChart[opts.colorAxis.type](opts.colorAxis.colorSeries,opts.colorAxis.palette) ;
  }
  
  //allow manipulation of default colors to use with dimple
  if(opts.defaultColors.length) {
    opts.defaultColors = opts.defaultColors[0];
    if (typeof(opts.defaultColors) == "function") {
      //assume this is a d3 scale
      //for now loop through first 20 but need a better way to handle
      defaultColorsArray = [];
      for (var n=0;n<20;n++) {
        defaultColorsArray.push(opts.defaultColors(n));
      };
      opts.defaultColors = defaultColorsArray;
    }
    opts.defaultColors.forEach(function(d,i) {
      opts.defaultColors[i] = new dimple.color(d);
    })
    myChart.defaultColors = opts.defaultColors;
  }  
  
  //do series
  //set up a function since same for each
  //as of now we have x,y,groups,data,type in opts for primary layer
  //and other layers reside in opts.layers
  function buildSeries(layer, hidden){
    //inherit from primary layer if not intentionally changed or xAxis, yAxis, zAxis null
    if (!layer.xAxis) layer.xAxis = opts.xAxis;    
    if (!layer.yAxis) layer.yAxis = opts.yAxis;
    if (!layer.zAxis) layer.zAxis = opts.zAxis;
    
    var x = buildAxis("x", layer);
    x.hidden = hidden;
    
    var y = buildAxis("y", layer);
    y.hidden = hidden;
    
    //z for bubbles
    var z = null;
    if (!(typeof(layer.zAxis) === 'undefined') && layer.zAxis.type){
      z = buildAxis("z", layer);
    };
    
    //here think I need to evaluate group and if missing do null
    //as the group argument
    //if provided need to use groups from layer
    var s = new dimple.series(myChart, null, x, y, z, c, null, dimple.plot[layer.type], dimple.aggregateMethod.avg, dimple.plot[layer.type].stacked);
    
    //as of v1.1.4 dimple can use different dataset for each series
    if(layer.data){
      //convert to an array of objects
      //avoid lodash for now
      datakeys = d3.keys(layer.data)
      layer.dataarray = layer.data[datakeys[1]].map(function(d,i){
        var tempobj = {}
        datakeys.forEach(function(key){
          tempobj[key] = layer.data[key][i]
        })
        return tempobj
      })
      s.data = layer.dataarray;
    }
    
    //for measure axis dimple sorts at the series level not at axis level
    ['x','y'].map(function(ax){
      if( layer[ax + 'Axis'].type=="addMeasureAxis" && layer[ax + 'Axis'].orderRule ){
        if( typeof layer[ax + 'Axis'].orderRule == "string" ){
          s.addOrderRule( layer[ax + 'Axis'].orderRule );
        } else if ( typeof layer[ax + 'Axis'].orderRule == "object" ) {
          s._orderRules = layer[ax + 'Axis'].orderRule;
        }
      }
    })
    
    if(layer.hasOwnProperty("groups")) {
      s.categoryFields = (typeof layer.groups === "object") ? layer.groups : [layer.groups];
      //series offers an aggregate method that we will also need to check if available
      //options available are avg, count, max, min, sum
    }
    if (!(typeof(layer.aggregate) === 'undefined')) {
      s.aggregate = eval(layer.aggregate);
    }
    if (!(typeof(layer.lineWeight) === 'undefined')) {
      s.lineWeight = layer.lineWeight;
    }
    if (!(typeof(layer.lineMarkers) === 'undefined')) {
      s.lineMarkers = layer.lineMarkers;
    }
    if (!(typeof(layer.barGap) === 'undefined')) {
      s.barGap = layer.barGap;
    }    
    if (!(typeof(layer.interpolation) === 'undefined')) {
      s.interpolation = layer.interpolation;
    } 
    
    myChart.series.push(s);
    
    /*placeholder fix domain of primary scale for new series data
    //not working right now but something like this
    //for now just use overrideMin and overrideMax from rCharts
    for( var i = 0; i<2; i++) {
      if (!myChart.axes[i].overrideMin) {
        myChart.series[0]._axisBounds(i==0?"x":"y").min = myChart.series[0]._axisBounds(i==0?"x":"y").min < s._axisBounds(i==0?"x":"y").min ? myChart.series[0]._axisBounds(i==0?"x":"y").min : s._axisBounds(i==0?"x":"y").min;
      }
      if (!myChart.axes[i].overrideMax) {  
        myChart.series[0]._axisBounds(i==0?"x":"y")._max = myChart.series[0]._axisBounds(i==0?"x":"y").max > s._axisBounds(i==0?"x":"y").max ? myChart.series[0]._axisBounds(i==0?"x":"y").max : s._axisBounds(i==0?"x":"y").max;
      }
      myChart.axes[i]._update();
    }
    */
      
    
    return s;
  };
  
  buildSeries(opts, false);
  if (opts.layers.length > 0) {
    opts.layers.forEach(function(layer){
      buildSeries(layer, true);
    })
  }
  //unsure if this is best but if legend is provided (not empty) then evaluate
  if(d3.keys(opts.legend).length > 0) {
    var l =myChart.addLegend();
    d3.keys(opts.legend).forEach(function(d){
      l[d] = opts.legend[d];
    });
  }
  //quick way to get this going but need to make this cleaner
  if(opts.storyboard) {
    myChart.setStoryboard(opts.storyboard);
  };
  myChart.draw();

</script>




To compare Inactive vs Active by Country, I find it useful to convey idea using pyramid chart from [Walkerke's Blog][walker]. Then, we plot the chart. It's clearly can be seen that not many demands are actually passed the negotiation state at the moment this report was produced. However, the outstanding inactive deals might signal the potential of profitable revenues for company in the future.


{% highlight r %}
n1$chart(color = c('silver','red'))
n1$chart(stacked = TRUE)
n1$print("final",cdn=TRUE)
n1$show("pyramic",cdn=TRUE)
{% endhighlight %}
<script src="/../js/jquery-2.1.1.min.js"></script>

<script src="/../js/nv.d3.min.js"></script>
<link rel="stylesheet" href="/../css/nv.d3.min.css"/>
<div id = 'final' class = 'rChart nvd3'></div>
<script type='text/javascript'>
 $(document).ready(function(){
      drawfinal()
    });
    function drawfinal(){  
      var opts = {
 "dom": "final",
"width":    800,
"height":    400,
"x": "Country",
"y": "Area",
"group": "Activity",
"type": "multiBarHorizontalChart",
"id": "final" 
},
        data = [
 {
 "Country": "AU",
"Activity": "Inactive",
"Area":         -11000 
},
{
 "Country": "CA",
"Activity": "Inactive",
"Area":          -5000 
},
{
 "Country": "CH",
"Activity": "Inactive",
"Area":          -3000 
},
{
 "Country": "CN",
"Activity": "Inactive",
"Area":         -38000 
},
{
 "Country": "DE",
"Activity": "Inactive",
"Area":          -8000 
},
{
 "Country": "DK",
"Activity": "Inactive",
"Area":          -2000 
},
{
 "Country": "FI",
"Activity": "Inactive",
"Area":          -5000 
},
{
 "Country": "FR",
"Activity": "Inactive",
"Area":          -1500 
},
{
 "Country": "GE",
"Activity": "Inactive",
"Area":          -2000 
},
{
 "Country": "HK",
"Activity": "Inactive",
"Area":         -14300 
},
{
 "Country": "ID",
"Activity": "Inactive",
"Area":          -4000 
},
{
 "Country": "IN",
"Activity": "Inactive",
"Area":         -14000 
},
{
 "Country": "IT",
"Activity": "Inactive",
"Area":          -3000 
},
{
 "Country": "JP",
"Activity": "Inactive",
"Area":         -99303 
},
{
 "Country": "KR",
"Activity": "Inactive",
"Area":         -30000 
},
{
 "Country": "LU",
"Activity": "Inactive",
"Area":          -1000 
},
{
 "Country": "MY",
"Activity": "Inactive",
"Area":          -4000 
},
{
 "Country": "NL",
"Activity": "Inactive",
"Area":          -1000 
},
{
 "Country": "PA",
"Activity": "Inactive",
"Area":          -6000 
},
{
 "Country": "SG",
"Activity": "Inactive",
"Area":         -21000 
},
{
 "Country": "TH",
"Activity": "Inactive",
"Area":          -3000 
},
{
 "Country": "TW",
"Activity": "Inactive",
"Area":          -4040 
},
{
 "Country": "UK",
"Activity": "Inactive",
"Area":          -7000 
},
{
 "Country": "US",
"Activity": "Inactive",
"Area":         -72902 
},
{
 "Country": "VN",
"Activity": "Inactive",
"Area":         -33700 
},
{
 "Country": "SE",
"Activity": "Inactive",
"Area":             -0 
},
{
 "Country": "EU",
"Activity": "Inactive",
"Area":             -0 
},
{
 "Country": "ES",
"Activity": "Inactive",
"Area":             -0 
},
{
 "Country": "AU",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "CA",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "CH",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "CN",
"Activity": "Active",
"Area":           1043 
},
{
 "Country": "DE",
"Activity": "Active",
"Area":           8706 
},
{
 "Country": "DK",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "FI",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "FR",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "GE",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "HK",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "ID",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "IN",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "IT",
"Activity": "Active",
"Area":           1040 
},
{
 "Country": "JP",
"Activity": "Active",
"Area":          23783 
},
{
 "Country": "KR",
"Activity": "Active",
"Area":           4000 
},
{
 "Country": "LU",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "MY",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "NL",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "PA",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "SG",
"Activity": "Active",
"Area":           2003 
},
{
 "Country": "TH",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "TW",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "UK",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "US",
"Activity": "Active",
"Area":           5043 
},
{
 "Country": "VN",
"Activity": "Active",
"Area":              0 
},
{
 "Country": "SE",
"Activity": "Active",
"Area":           1000 
},
{
 "Country": "EU",
"Activity": "Active",
"Area":           3123 
},
{
 "Country": "ES",
"Activity": "Active",
"Area":           1000 
} 
]
  
      if(!(opts.type==="pieChart" || opts.type==="sparklinePlus" || opts.type==="bulletChart")) {
        var data = d3.nest()
          .key(function(d){
            //return opts.group === undefined ? 'main' : d[opts.group]
            //instead of main would think a better default is opts.x
            return opts.group === undefined ? opts.y : d[opts.group];
          })
          .entries(data);
      }
      
      if (opts.disabled != undefined){
        data.map(function(d, i){
          d.disabled = opts.disabled[i]
        })
      }
      
      nv.addGraph(function() {
        var chart = nv.models[opts.type]()
          .width(opts.width)
          .height(opts.height)
          
        if (opts.type != "bulletChart"){
          chart
            .x(function(d) { return d[opts.x] })
            .y(function(d) { return d[opts.y] })
        }
          
         
        chart
  .tooltipContent( function(key, x, y, e){
var format = d3.format('0,000');
return '<h3>' + key + ', country ' + x + '</h3>' +
'<p>' + 'Area: ' + y + '</p>'
} )
  .color([ "silver", "red" ])
  .stacked(true)
          
        

        
        
        chart.yAxis
  .axisLabel("Area")
  .tickFormat( function(d) {
return d3.format(',.0f')(Math.abs(d) / 1000) + 'K'
} )
      
       d3.select("#" + opts.id)
        .append('svg')
        .datum(data)
        .transition().duration(500)
        .call(chart);

       nv.utils.windowResize(chart.update);
       return chart;
      });
    };
</script>

Cheers, Have fun digging up the data ~


{% highlight r %}
sessionInfo()
{% endhighlight %}



{% highlight text %}
## R version 3.1.1 (2014-07-10)
## Platform: x86_64-apple-darwin13.1.0 (64-bit)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] methods   stats     graphics  grDevices utils     datasets  base     
## 
## other attached packages:
## [1] reshape2_1.4.1 rCharts_0.4.5  knitr_1.8     
## 
## loaded via a namespace (and not attached):
##  [1] evaluate_0.5.5  formatR_1.0     grid_3.1.1      lattice_0.20-29
##  [5] plyr_1.8.1      Rcpp_0.11.3     rjson_0.2.15    RJSONIO_1.3-0  
##  [9] stringr_0.6.2   tools_3.1.1     whisker_0.3-2   yaml_2.1.13
{% endhighlight %}
[walker]: http://walkerke.github.io/2014/06/rcharts-pyramids/

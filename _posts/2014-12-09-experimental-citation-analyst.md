---
layout: post
title:  "Digging Patent US Data"
date:   2014-12-09
categories: jekyll update
permalink: digging-patent-us-data
comments: true
---
# Tools
---
  + Programing Language: Python, Rscript, Javascript
  + Data Object: Patent
  + [Neo4j][neo4j] as backend database.
  + [parse_patent][parse_patent] to extract xml info.
  + [generate_csv][generate_csv] to produce csv
  + [Gephi][gephi] to visualize citation network
  + [RStudio][rstudio] for statistics.
  + [Py2neo][py2neo] as handy wrap-around library to initiate connection with Neo4j

# Analytical Objectives
---
  1. How is the citation network of patents ?
  2. Which country has the most inventors participating in patern industry of US ?
  
# Extracting Data 
---
This experiment based on [USPTO][uspto] patent data. Firstly, we might need to parse the xml file to obtain desired results. Since ..., xml files retrieved from the site are  no longer a list of xml files but a combination of multiple xml instances, I wrote `parse_patent` to extract information, an option `-s` to specify the number of xml instances should be retrieved. 

~~~~~~~
(env)MACs-MacBook-Pro:citation_network mac$ ./parse_patent.py 
[Usage:] parse_patent.py -i <inputfile.xml>
(env)MACs-MacBook-Pro:citation_network mac$ ./parse_patent.py -i ipg141007.xml 
Starting [.........] Done! Total  100  patents
~~~~~~~

# Populating Data
---
Upon completion, `parse_patent` would insert directly to Neo4j server through restful api via bulk update. Now, we have the database with populated data. It's time to call `generate_csv` to produce some texts, I chose `csv` as it's common and easy to manipulate and visualize.

~~~~~~~
(env)MACs-MacBook-Pro:citation_network mac$ ./generate_csv.py 
Writing inventors to csv succeeded !
Writing patents to csv succeeded !
Writing inventor_to_patent links to csv succeed !
Writing patent_to_patent links to csv succeed !
Successfully populated inventors and patents to nodes_network.csv
Successfully populated links between nodes_network.csv to links_network.csv
~~~~~~~

# Visualizing
---
Finally, we have `nodes_network.csv` and `links_network.csv`, place it inside Gephi to produce citation networks.
![title](/../figs/2014-12-09-experimental-citation-analyst/citation-500.png)

Group data by country using `RStudio`, we can visualize countries with inventors:

 <!-- GeoChart generated in R 3.1.1 by googleVis 0.5.6 package -->
<!-- Tue Dec  9 12:21:16 2014 -->


<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataGeoChartID1b004a31bf5e () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
 "AR",
1 
],
[
 "AT",
2 
],
[
 "AU",
9 
],
[
 "BR",
1 
],
[
 "CA",
18 
],
[
 "CH",
4 
],
[
 "CN",
28 
],
[
 "CZ",
2 
],
[
 "DE",
27 
],
[
 "DK",
1 
],
[
 "FI",
5 
],
[
 "FR",
11 
],
[
 "GB",
3 
],
[
 "HK",
3 
],
[
 "IL",
3 
],
[
 "IS",
2 
],
[
 "IT",
6 
],
[
 "JP",
57 
],
[
 "KP",
1 
],
[
 "KR",
105 
],
[
 "NL",
7 
],
[
 "NO",
10 
],
[
 "NZ",
2 
],
[
 "SE",
10 
],
[
 "SG",
2 
],
[
 "TW",
33 
],
[
 "US",
460 
],
[
 "VE",
1 
],
[
 "VN",
1 
],
[
 "ZA",
2 
] 
];
data.addColumn('string','Country');
data.addColumn('number','No.Inventors');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartGeoChartID1b004a31bf5e() {
var data = gvisDataGeoChartID1b004a31bf5e();
var options = {};
options["width"] =    700;
options["height"] =    400;
options["projection"] = "kavrayskiy-vii";
options["colorAxis"] = {colors:['#91BFDB', '#FC8D59']};


    var chart = new google.visualization.GeoChart(
    document.getElementById('GeoChartID1b004a31bf5e')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "geochart";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartGeoChartID1b004a31bf5e);
})();
function displayChartGeoChartID1b004a31bf5e() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
 
<!-- jsChart -->  
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartGeoChartID1b004a31bf5e"></script>
 
<!-- divChart -->
  
<div id="GeoChartID1b004a31bf5e" 
  style="width: 556; height: 347;">
</div>

# Future Works
---
+ `generate_csv.py` is currently communicated with `Neo4j` through restful API and creates `csv`. It would be more convenient to implement an independent program that works with multiple databases and produce wide ranges of output types such as `json`, `geojson`.

+ As data increases, handling enormous datasets with Graph Database(E.g `Neo4j`) proved insufficient and significant downgrade in performance . Apparently, `parse_patent.py` can parse up to 500 xml instances, however alternative choices will be taken into consideration such as `MongoDB`


~ Have fun digging data ! ~


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
[uspto]: http://storage.googleapis.com/patents/grant_full_text/2014/ipg141007.zip
[neo4j]: http://neo4j.com/
[gephi]:http://gephi.github.io/
[rstudio]:http://www.rstudio.com/
[py2neo]: http://py2neo.org/2.0/



---


{% highlight text %}
## R version 3.1.1 (2014-07-10)
## Platform: x86_64-apple-darwin13.1.0 (64-bit)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  base     
## 
## other attached packages:
## [1] googleVis_0.5.6 knitr_1.8      
## 
## loaded via a namespace (and not attached):
## [1] evaluate_0.5.5 formatR_1.0    methods_3.1.1  RJSONIO_1.3-0 
## [5] stringr_0.6.2  tools_3.1.1
{% endhighlight %}

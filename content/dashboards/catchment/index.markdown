---
title: Catchment Monitor
author: 'jasper'
date: '2021-01-08'
slug: catchment
categories: []
tags:
  - dashboards
  - climate
  - hydrology
  - data
  - Jonkershoek
subtitle: ''
summary: "Rainfall and streamflow from the catchments that feed Cape Town's major dams"
authors: [jasper, abri]
lastmod: '2021-01-08T21:36:22+02:00'
featured: yes
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


<br>

Here we present the record of stream flow rates for the Langrivier catchment and rainfall from the Dwarsberg weather station in the Jonkershoek Valley for the period January 1961 to the end of December 2020. The Dwarsberg weather station is at 1214 metres above sea level on the boundary of the catchments of the Eerste, Berg and Sonderend rivers and is a good indicator of rainfall feeding the Berg and Theewaterskloof dams that feed Cape Town and surrounds.

<!--more-->

We plan to update this page monthly. For live weather data and the record over the past month please access the [Dwarsberg weather station directly](http://lognet.saeon.ac.za:8088/Dwarsberg/index.html). You may also be interested in our [Constantiaberg weather station](http://lognet.saeon.ac.za:8088/Constantiaberg/index.html) on the Cape Peninsula or our [Engelsmanskloof weather station](http://lognet.saeon.ac.za:8088/EngCed/index.html) in the Cederberg.


*PLEASE NOTE: We cannot guarantee that these data or analyses are error free, particularly with regards to the older historical data and the comparability of the older and newer instrumentation. Gaps in the data were filled using linear interpolation from the most recent data for up to a 30 day period. This filled all gaps other than the period August 2008 to September 2011 - these data exist in hard copy, but need to be digitized. There are much better gap-filling techniques, but the appropriate method depends on the question being asked.*




***

## Streamflow record

***


Here are the recent monthly streamflow values for Langrivier relative to averages across the entire time period (1961 to current).

<img src="index_files/figure-html/unnamed-chunk-2-1.png" width="672" style="display: block; margin: auto;" />

Black error bars around the mean (grey column) indicate 95% confidence intervals. Monthly values lower than the lower bar are among the lowest 5% of recorded flows for that month. Note that missing months (e.g. October-November 2015) are those that overlap a data gap of over 30 days.


We can also view this cumulatively:


<img src="index_files/figure-html/unnamed-chunk-3-1.png" width="672" style="display: block; margin: auto;" />


Or compare across all years on record:


<img src="index_files/figure-html/unnamed-chunk-4-1.png" width="768" style="display: block; margin: auto;" />


Note that the latest season is not yet complete. We just indicate it for reference. "Summer" denotes the Austral summer (here defined as October - March), overlapping 2 years. In this figure we have lumped summers by the preceding year, e.g. summer 2016 represents the period October 2016 to March 2017, which incidentally was the driest summer on record in terms of streamflow.

*Data for October - December from the summer of 2015/2016 are missing due to vandalism of the weir, but January - March 2016 was among the lowest 10% of flows for that period, suggesting that 2015 was likely the driest year on record. Of those with complete data, 2003 was the driest year, followed by 1974.








<br>

***

## Rainfall Record

***

Here are the recent monthly rainfall values relative to averages across the entire time period (1945 to current):

<img src="index_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />

Black error bars around the mean (grey column) indicate 95% confidence intervals. Monthly values lower than the lower bar are among the lowest 5% of recorded flows for that month.

*Indicates that these months were missing data in 2015 due to power supply problems.

<br>

And if we view this cumulatively:


<img src="index_files/figure-html/unnamed-chunk-9-1.png" width="672" style="display: block; margin: auto;" />


Or compare across all years on record:


<img src="index_files/figure-html/unnamed-chunk-10-1.png" width="768" style="display: block; margin: auto;" />


So 2015 and 2016 have been the lowest rainfall years on record, while 2017 and 2014 were not far behind... Note that data for the period 1992-2013 were collected by volunteers and were not necessarily collected on the first of the month or every month, creating issues in the split between years and between summer vs winter data. This appears to be more of an issue in the 2000s. Nevertheless, the low rainfall reported for 2000-2004 does coincide with a period of low streamflow.




<br>

Note the frequent "zero-rainfall" months in the period 2000 to 2013, likely because the volunteers who collected the data were not able to visit the rain gauge in those months. The gauge can collect >1000mm before overflowing, so this is unlikely to have been a problem for rainfall totals.

<br>

***

## Streamflow ~ Rainfall

***

There have been various calls for some sort of indication as to how streamflow responds to rainfall. We all want to know how much it needs to rain to fill the dams. While we don't have a fully coupled atmosphere-hydrological model on hand, and we're still reading up on the most appropriate time-series modelling approach (especially considering the data gaps...), here're some quick and dirty indicators based on the periods of overlapping streamflow and rainfall data (1961 - 1991, 2013 - 2018).

Firstly, cumulative monthly streamflow as a function of monthly rainfall:

<img src="index_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />

<br>

While it looks messy, one clear pattern is that a rainfall event in a winter month typically results in greater streamflow than a similar sized event in a summer month. This is especially evident if you compare large rainfall events in late summer/autumn (March, April, May) with those in winter (June, July, August). This is not surprising and likely a result of accumulated soil moisture (or lack thereof) throughout the season, while lower evapotranspiration in winter will also be coming into play.

What does this look like if we aggregate months by season (following the Summer = October to March, Winter = April to September approach from above).


<img src="index_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />

The trend shows an inflection reflecting in increase in streamflow for a given unit of rainfall as the total amount of rainfall increases. This could relate to seasonal soil moisture accumulation, but may also relate to groundwater deficit over longer periods.

<br>

***

## Soil Moisture!!!

***

Here we present soil moisture data from soil moisture probes at three different depths (10, 20 and 30cm) at Dwarsberg from 2013 to current. Data are presented as Volumetric Water Content (VWC) converted to percentage.

<img src="index_files/figure-html/unnamed-chunk-14-1.png" width="672" style="display: block; margin: auto;" />

Notice the late onset of wetter soils in 2015 and 2017, and early drying in 2015 and 2016. 

We hope to add an analysis of monthly streamflow received for a given rainfall and soil moisture for this period soon, but that will have to wait for now. Another thing to think about is including evapotranspiration...




Feel free to post comments/suggestions below!

***

If you spot any issues, have any queries, or would like to use or cite these data please contact us.

***

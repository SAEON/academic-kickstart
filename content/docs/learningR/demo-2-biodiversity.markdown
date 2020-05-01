---
title: 'R Intro 2: Biodiversity'
author: jasper
date: '2020-05-01'
slug: r-intro-2-biodiversity
categories: []
tags:
  - biodiversity
  - Cape Peninsula
  - data
  - R-tutorial
linktitle: R Intro 2
summary: ~
lastmod: '2020-05-01T22:38:34+02:00'
toc: yes
type: docs
menu:
  docs:
    learningR:
      name: 
    weight: 30
---

Ok. So the previous tutorial covered a bunch of the basics with some pretty straightforward data, and was more focused on how R works. Now we’ll be looking at slightly more complicated data (species data), which will require learning some fancier data handling.

First, let’s get the R packages we need and set our working directory





```r
library(tidyverse)
```

```
## ── Attaching packages ──────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✓ ggplot2 3.2.1     ✓ purrr   0.3.3
## ✓ tibble  2.1.3     ✓ dplyr   0.8.3
## ✓ tidyr   1.0.0     ✓ stringr 1.4.0
## ✓ readr   1.3.1     ✓ forcats 0.4.0
```

```
## ── Conflicts ─────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(readxl)
library(vegan) #comment out with "#" if you don't have this installed
```

```
## Loading required package: permute
```

```
## Loading required package: lattice
```

```
## This is vegan 2.5-5
```

```r
setwd("/home/jasper/GIT/academic-kickstart/static/datasets/")
```

Now we need the data. Let’s look at the sheets in our Excel workbook again


```r
excel_sheets("pnas.1619014114.sd01.xlsx")
```

```
##  [1] "METADATA"           "weather"            "fires"             
##  [4] "postfireweather"    "enviroment"         "excluded_spp"      
##  [7] "veg1966"            "veg1996"            "veg2010"           
## [10] "traits"             "speciesclimatedata"
```

So we probably need "Species" and "% cover", but we can have a glimpse at them to doublecheck they’re the data we want by including a range in read_excel() like so


```r
read_excel("pnas.1619014114.sd01.xlsx", sheet = "veg1966", range = "A1:K5")
```

```
## New names:
## * `` -> ...1
```

```
## # A tibble: 4 x 11
##   ...1  `Adenandra unif… `Adenandra vill… `Adenocline pau… `Agathosma cili…
##   <chr>            <dbl>            <dbl>            <dbl>            <dbl>
## 1 CP_1                 0                7                0                0
## 2 CP_10                0                0                0                0
## 3 CP_1…                0                7                0                0
## 4 CP_12                0                0                0                0
## # … with 6 more variables: `Agathosma hookeri` <dbl>, `Agathosma
## #   imbricata` <dbl>, `Agathosma lanceolata` <dbl>, `Agathosma
## #   serpyllacea` <dbl>, `Aizoon paniculatum` <dbl>, `Amphithalea
## #   ericifolia` <dbl>
```

```r
read_excel("pnas.1619014114.sd01.xlsx", sheet = "veg2010", range = "A1:K5")
```

```
## New names:
## * `` -> ...1
```

```
## # A tibble: 4 x 11
##   ...1  `Acacia saligna` `Adenandra unif… `Adenandra vill… `Agapanthus wal…
##   <chr>            <dbl>            <dbl>            <dbl>            <dbl>
## 1 CP_1                 0                0                3                0
## 2 CP_10                0                0                0                0
## 3 CP_12                0                0                0                0
## 4 CP_13                0                0                0                0
## # … with 6 more variables: `Agathosma ciliaris` <dbl>, `Agathosma
## #   hookeri` <dbl>, `Agathosma imbricata` <dbl>, `Agathosma
## #   serpyllacea` <dbl>, `Albuca flaccida` <dbl>, `Albuca juncifolia` <dbl>
```

Ok, so it looks like `veg1966`, `veg1996` and `veg2010` are what we call community data matrices. These are site by species matrices with measures of abundance, % cover, or similar values for each species in each plot. In this case the `veg1966` and `veg1996` data were Braun-Blanquet cover-abundance classes, which I have converted to abundance using the midpoint of each class. THIS IS NOT NECESSARILY DEFENSIBLE!!! but I just did it for the purposes of this tutorial. The `veg2010` dataset did not use the BB classes and have actual counts.

Let’s read them in and have a closer look


```r
species66 <- read_excel("pnas.1619014114.sd01.xlsx", sheet = "veg1966")
```

```
## New names:
## * `` -> ...1
```

```r
head(species66)
```

```
## # A tibble: 6 x 428
##   ...1  `Adenandra unif… `Adenandra vill… `Adenocline pau… `Agathosma cili…
##   <chr>            <dbl>            <dbl>            <dbl>            <dbl>
## 1 CP_1                 0                7                0                0
## 2 CP_10                0                0                0                0
## 3 CP_1…                0                7                0                0
## 4 CP_12                0                0                0                0
## 5 CP_13                0                0                0                0
## 6 CP_14                0                0                0                0
## # … with 423 more variables: `Agathosma hookeri` <dbl>, `Agathosma
## #   imbricata` <dbl>, `Agathosma lanceolata` <dbl>, `Agathosma
## #   serpyllacea` <dbl>, `Aizoon paniculatum` <dbl>, `Amphithalea
## #   ericifolia` <dbl>, `Anaxeton laeve` <dbl>, `Anemone knowltonia` <dbl>,
## #   `Anthochortus laxiflorus` <dbl>, `Anthospermum aethiopicum` <dbl>,
## #   `Anthospermum bergianum` <dbl>, `Anthospermum galioides` <dbl>, `Apium
## #   decumbens` <dbl>, `Arctotis aspera` <dbl>, `Argyrolobium
## #   filiforme` <dbl>, `Aristea africana` <dbl>, `Aristea capitata` <dbl>,
## #   `Aristea glauca` <dbl>, `Aspalathus abietina` <dbl>, `Aspalathus
## #   argyrella` <dbl>, `Aspalathus callosa` <dbl>, `Aspalathus
## #   capensis` <dbl>, `Aspalathus carnosa` <dbl>, `Aspalathus
## #   chenopoda` <dbl>, `Aspalathus divaricata` <dbl>, `Aspalathus
## #   ericifolia` <dbl>, `Aspalathus hispida` <dbl>, `Aspalathus
## #   laricifolia` <dbl>, `Aspalathus linguiloba` <dbl>, `Aspalathus
## #   microphylla` <dbl>, `Aspalathus retroflexa` <dbl>, `Aspalathus
## #   sericea` <dbl>, `Aspalathus serpens` <dbl>, `Asparagus
## #   aethiopicus` <dbl>, `Asparagus capensis` <dbl>, `Asparagus
## #   lignosus` <dbl>, `Asparagus rubicundus` <dbl>, `Berkheya
## #   barbata` <dbl>, `Berzelia abrotanoides` <dbl>, `Berzelia
## #   lanuginosa` <dbl>, `Bobartia filiformis` <dbl>, `Bobartia
## #   gladiata` <dbl>, `Bobartia indica` <dbl>, `Bolusafra
## #   bituminosa` <dbl>, `Bulbine abyssinica` <dbl>, `Caesia
## #   contorta` <dbl>, `Capelio tabularis` <dbl>, `Capeobolus
## #   brevicaulis` <dbl>, `Capeochloa cincta` <dbl>, `Carpacoce
## #   spermacocea` <dbl>, `Carpacoce vaginellata` <dbl>, `Carpobrotus
## #   acinaciformis` <dbl>, `Cassine peragua` <dbl>, `Cassytha
## #   ciliolata` <dbl>, `Centella affinis` <dbl>, `Centella glabrata` <dbl>,
## #   `Centella macrocarpa` <dbl>, `Centella tridentata` <dbl>, `Chironia
## #   baccifera` <dbl>, `Chironia decumbens` <dbl>, `Chironia
## #   linoides` <dbl>, `Chrysocoma coma-aurea` <dbl>, `Cineraria
## #   geifolia` <dbl>, `Cliffortia atrata` <dbl>, `Cliffortia
## #   falcata` <dbl>, `Cliffortia ferruginea` <dbl>, `Cliffortia
## #   filifolia` <dbl>, `Cliffortia glauca` <dbl>, `Cliffortia
## #   obcordata` <dbl>, `Cliffortia polygonifolia` <dbl>, `Cliffortia
## #   ruscifolia` <dbl>, `Cliffortia stricta` <dbl>, `Cliffortia
## #   subsetacea` <dbl>, `Clutia alaternoides` <dbl>, `Clutia
## #   ericoides` <dbl>, `Clutia polygonoides` <dbl>, `Coleonema
## #   album` <dbl>, `Conyza pinnatifida` <dbl>, `Corymbium africanum` <dbl>,
## #   `Corymbium glabrum` <dbl>, `Crassula capensis` <dbl>, `Crassula
## #   coccinea` <dbl>, `Crassula cymosa` <dbl>, `Crassula
## #   fascicularis` <dbl>, `Crassula flava` <dbl>, `Crassula
## #   nudicaulis` <dbl>, `Crassula subulata` <dbl>, `Cullumia
## #   squarrosa` <dbl>, `Cussonia thyrsiflora` <dbl>, `Cymbopogon
## #   marginatus` <dbl>, `Cynanchum africanum` <dbl>, `Cynanchum
## #   obtusifolium` <dbl>, `Cynodon dactylon` <dbl>, `Cyperus
## #   thunbergii` <dbl>, `Diastella divaricata` <dbl>, `Dilatris
## #   corymbosa` <dbl>, `Dilatris pillansii` <dbl>, `Dimorphotheca
## #   nudicaulis` <dbl>, `Diosma hirsuta` <dbl>, `Diosma
## #   oppositifolia` <dbl>, …
```

```r
species10 <- read_excel("pnas.1619014114.sd01.xlsx", sheet = "veg2010")
```

```
## New names:
## * `` -> ...1
```

```r
head(species10)
```

```
## # A tibble: 6 x 339
##   ...1  `Acacia saligna` `Adenandra unif… `Adenandra vill… `Agapanthus wal…
##   <chr>            <dbl>            <dbl>            <dbl>            <dbl>
## 1 CP_1                 0                0                3                0
## 2 CP_10                0                0                0                0
## 3 CP_12                0                0                0                0
## 4 CP_13                0                0                0                0
## 5 CP_14                0                0                0                0
## 6 CP_15                0                0                0                0
## # … with 334 more variables: `Agathosma ciliaris` <dbl>, `Agathosma
## #   hookeri` <dbl>, `Agathosma imbricata` <dbl>, `Agathosma
## #   serpyllacea` <dbl>, `Albuca flaccida` <dbl>, `Albuca
## #   juncifolia` <dbl>, `Amphithalea ericifolia` <dbl>, `Anaxeton
## #   laeve` <dbl>, `Anemone knowltonia` <dbl>, `Anthochortus
## #   capensis` <dbl>, `Anthospermum aethiopicum` <dbl>, `Anthospermum
## #   bergianum` <dbl>, `Anthospermum galioides` <dbl>, `Apium
## #   decumbens` <dbl>, `Arctotheca calendula` <dbl>, `Aristea
## #   africana` <dbl>, `Aspalathus callosa` <dbl>, `Aspalathus
## #   capensis` <dbl>, `Aspalathus carnosa` <dbl>, `Aspalathus
## #   chenopoda` <dbl>, `Aspalathus ciliaris` <dbl>, `Aspalathus
## #   hispida` <dbl>, `Aspalathus linguiloba` <dbl>, `Aspalathus
## #   microphylla` <dbl>, `Aspalathus retroflexa` <dbl>, `Aspalathus
## #   serpens` <dbl>, `Asparagus capensis` <dbl>, `Asparagus
## #   lignosus` <dbl>, `Asparagus rubicundus` <dbl>, `Babiana
## #   ambigua` <dbl>, `Babiana villosula` <dbl>, `Berkheya barbata` <dbl>,
## #   `Berzelia abrotanoides` <dbl>, `Berzelia lanuginosa` <dbl>, `Bobartia
## #   gladiata` <dbl>, `Bobartia indica` <dbl>, `Bolusafra
## #   bituminosa` <dbl>, `Bulbine abyssinica` <dbl>, `Bulbine
## #   alooides` <dbl>, `Bulbine lagopus` <dbl>, `Caesia contorta` <dbl>,
## #   `Capelio tabularis` <dbl>, `Capeochloa cincta` <dbl>, `Carpobrotus
## #   acinaciformis` <dbl>, `Carpobrotus edulis` <dbl>, `Cassine
## #   peragua` <dbl>, `Cassytha ciliolata` <dbl>, `Centella
## #   macrocarpa` <dbl>, `Centella tridentata` <dbl>, `Chenolea
## #   diffusa` <dbl>, `Chironia baccifera` <dbl>, `Chironia
## #   decumbens` <dbl>, `Chironia linoides` <dbl>, `Cliffortia
## #   atrata` <dbl>, `Cliffortia falcata` <dbl>, `Cliffortia
## #   ferruginea` <dbl>, `Cliffortia obcordata` <dbl>, `Cliffortia
## #   stricta` <dbl>, `Cliffortia subsetacea` <dbl>, `Clutia
## #   alaternoides` <dbl>, `Clutia ericoides` <dbl>, `Coleonema
## #   album` <dbl>, `Colpoon compressum` <dbl>, `Corymbium africanum` <dbl>,
## #   `Corymbium glabrum` <dbl>, `Cotula coronopifolia` <dbl>, `Cotula
## #   turbinata` <dbl>, `Crassula fascicularis` <dbl>, `Crassula
## #   nudicaulis` <dbl>, `Cymbopogon marginatus` <dbl>, `Cynodon
## #   dactylon` <dbl>, `Dasispermum hispidum` <dbl>, `Diastella
## #   divaricata` <dbl>, `Dilatris pillansii` <dbl>, `Diosma hirsuta` <dbl>,
## #   `Diosma oppositifolia` <dbl>, `Diospyros glabra` <dbl>, `Disa
## #   bracteata` <dbl>, `Disa obliqua` <dbl>, `Disa purpurascens` <dbl>,
## #   `Disparago tortilis` <dbl>, `Disperis capensis` <dbl>, `Drimia
## #   capensis` <dbl>, `Drosanthemum candens` <dbl>, `Drosera
## #   aliciae` <dbl>, `Drosera trinervia` <dbl>, `Edmondia
## #   sesamoides` <dbl>, `Ehrharta calycina` <dbl>, `Ehrharta ramosa` <dbl>,
## #   `Elegia cuspidata` <dbl>, `Elegia filacea` <dbl>, `Elegia
## #   juncea` <dbl>, `Elegia microcarpa` <dbl>, `Elegia nuda` <dbl>, `Elegia
## #   persistens` <dbl>, `Elegia stipularis` <dbl>, `Elegia
## #   vaginulata` <dbl>, `Elytropappus scaber` <dbl>, `Eragrostis
## #   capensis` <dbl>, `Erepsia anceps` <dbl>, …
```

Yup, that looks about right.

Let’s start by playing with the 1966 data.

Ok, so what do we want to know?


## Summarizing community data

Total abundance by plot? We can do this by summing the abundance of all species in each site (i.e. row in the matrix).


```r
rowSums(species66)
```

```
## Error in rowSums(species66): 'x' must be numeric
```

A classic R error message. It seems unhelpful, but as you learn R the error messages make more sense. In this case it is telling us that not all the values in `species66` are numbers (i.e. of class "numeric"). Some may be of class "character", "factor", etc.

If we look at the data more closely...


```r
head(species66)
```

```
## # A tibble: 6 x 428
##   ...1  `Adenandra unif… `Adenandra vill… `Adenocline pau… `Agathosma cili…
##   <chr>            <dbl>            <dbl>            <dbl>            <dbl>
## 1 CP_1                 0                7                0                0
## 2 CP_10                0                0                0                0
## 3 CP_1…                0                7                0                0
## 4 CP_12                0                0                0                0
## 5 CP_13                0                0                0                0
## 6 CP_14                0                0                0                0
## # … with 423 more variables: `Agathosma hookeri` <dbl>, `Agathosma
## #   imbricata` <dbl>, `Agathosma lanceolata` <dbl>, `Agathosma
## #   serpyllacea` <dbl>, `Aizoon paniculatum` <dbl>, `Amphithalea
## #   ericifolia` <dbl>, `Anaxeton laeve` <dbl>, `Anemone knowltonia` <dbl>,
## #   `Anthochortus laxiflorus` <dbl>, `Anthospermum aethiopicum` <dbl>,
## #   `Anthospermum bergianum` <dbl>, `Anthospermum galioides` <dbl>, `Apium
## #   decumbens` <dbl>, `Arctotis aspera` <dbl>, `Argyrolobium
## #   filiforme` <dbl>, `Aristea africana` <dbl>, `Aristea capitata` <dbl>,
## #   `Aristea glauca` <dbl>, `Aspalathus abietina` <dbl>, `Aspalathus
## #   argyrella` <dbl>, `Aspalathus callosa` <dbl>, `Aspalathus
## #   capensis` <dbl>, `Aspalathus carnosa` <dbl>, `Aspalathus
## #   chenopoda` <dbl>, `Aspalathus divaricata` <dbl>, `Aspalathus
## #   ericifolia` <dbl>, `Aspalathus hispida` <dbl>, `Aspalathus
## #   laricifolia` <dbl>, `Aspalathus linguiloba` <dbl>, `Aspalathus
## #   microphylla` <dbl>, `Aspalathus retroflexa` <dbl>, `Aspalathus
## #   sericea` <dbl>, `Aspalathus serpens` <dbl>, `Asparagus
## #   aethiopicus` <dbl>, `Asparagus capensis` <dbl>, `Asparagus
## #   lignosus` <dbl>, `Asparagus rubicundus` <dbl>, `Berkheya
## #   barbata` <dbl>, `Berzelia abrotanoides` <dbl>, `Berzelia
## #   lanuginosa` <dbl>, `Bobartia filiformis` <dbl>, `Bobartia
## #   gladiata` <dbl>, `Bobartia indica` <dbl>, `Bolusafra
## #   bituminosa` <dbl>, `Bulbine abyssinica` <dbl>, `Caesia
## #   contorta` <dbl>, `Capelio tabularis` <dbl>, `Capeobolus
## #   brevicaulis` <dbl>, `Capeochloa cincta` <dbl>, `Carpacoce
## #   spermacocea` <dbl>, `Carpacoce vaginellata` <dbl>, `Carpobrotus
## #   acinaciformis` <dbl>, `Cassine peragua` <dbl>, `Cassytha
## #   ciliolata` <dbl>, `Centella affinis` <dbl>, `Centella glabrata` <dbl>,
## #   `Centella macrocarpa` <dbl>, `Centella tridentata` <dbl>, `Chironia
## #   baccifera` <dbl>, `Chironia decumbens` <dbl>, `Chironia
## #   linoides` <dbl>, `Chrysocoma coma-aurea` <dbl>, `Cineraria
## #   geifolia` <dbl>, `Cliffortia atrata` <dbl>, `Cliffortia
## #   falcata` <dbl>, `Cliffortia ferruginea` <dbl>, `Cliffortia
## #   filifolia` <dbl>, `Cliffortia glauca` <dbl>, `Cliffortia
## #   obcordata` <dbl>, `Cliffortia polygonifolia` <dbl>, `Cliffortia
## #   ruscifolia` <dbl>, `Cliffortia stricta` <dbl>, `Cliffortia
## #   subsetacea` <dbl>, `Clutia alaternoides` <dbl>, `Clutia
## #   ericoides` <dbl>, `Clutia polygonoides` <dbl>, `Coleonema
## #   album` <dbl>, `Conyza pinnatifida` <dbl>, `Corymbium africanum` <dbl>,
## #   `Corymbium glabrum` <dbl>, `Crassula capensis` <dbl>, `Crassula
## #   coccinea` <dbl>, `Crassula cymosa` <dbl>, `Crassula
## #   fascicularis` <dbl>, `Crassula flava` <dbl>, `Crassula
## #   nudicaulis` <dbl>, `Crassula subulata` <dbl>, `Cullumia
## #   squarrosa` <dbl>, `Cussonia thyrsiflora` <dbl>, `Cymbopogon
## #   marginatus` <dbl>, `Cynanchum africanum` <dbl>, `Cynanchum
## #   obtusifolium` <dbl>, `Cynodon dactylon` <dbl>, `Cyperus
## #   thunbergii` <dbl>, `Diastella divaricata` <dbl>, `Dilatris
## #   corymbosa` <dbl>, `Dilatris pillansii` <dbl>, `Dimorphotheca
## #   nudicaulis` <dbl>, `Diosma hirsuta` <dbl>, `Diosma
## #   oppositifolia` <dbl>, …
```

...we see that the first row is actually a vector of plot names (e.g. CP_1), which is of class "character". These will be useful later, but for this analysis we want to drop them. This is easy using indexing with square brackets "[,]".


```r
rowSums(species66[,2:ncol(species66)])
```

```
##  [1]  522.0  876.0  331.0  678.0 1029.5  523.0  880.5 1054.5 1306.0 1196.5
## [11]  396.0  531.0  473.0  625.5  197.0  837.0  178.0  377.0  275.0  144.0
## [21]  723.0  624.5  391.0  577.0  441.0  442.0  347.5  419.0  181.0  307.0
## [31] 1341.0  788.5  616.5  452.5   59.0  389.5  244.0  370.0  460.0  837.0
## [41]  214.0  251.0  184.0  448.0  129.0  859.0  299.0  353.0  424.0  467.5
## [51]  461.0  625.0  536.0 1377.5  554.5  311.0  509.5  332.0 1574.0 2345.0
## [61]  562.5  574.0 1584.0  257.0  860.5 1901.0  688.0  622.0  985.0 2045.5
## [71]  485.5  282.0 1335.0  184.0  355.5  270.0  481.0  341.0   50.0  409.0
## [81]  731.0
```

In this case we selected all columns from the 2nd column to the total number of columns as returned by `ncol()` (i.e. all but the first column). You could also do it like this...


```r
rowSums(species66[,-1])
```

```
##  [1]  522.0  876.0  331.0  678.0 1029.5  523.0  880.5 1054.5 1306.0 1196.5
## [11]  396.0  531.0  473.0  625.5  197.0  837.0  178.0  377.0  275.0  144.0
## [21]  723.0  624.5  391.0  577.0  441.0  442.0  347.5  419.0  181.0  307.0
## [31] 1341.0  788.5  616.5  452.5   59.0  389.5  244.0  370.0  460.0  837.0
## [41]  214.0  251.0  184.0  448.0  129.0  859.0  299.0  353.0  424.0  467.5
## [51]  461.0  625.0  536.0 1377.5  554.5  311.0  509.5  332.0 1574.0 2345.0
## [61]  562.5  574.0 1584.0  257.0  860.5 1901.0  688.0  622.0  985.0 2045.5
## [71]  485.5  282.0 1335.0  184.0  355.5  270.0  481.0  341.0   50.0  409.0
## [81]  731.0
```

This just highlights that R almost always provides a number of ways to do the same thing. For example, if one wanted to calculate the row sums using the *tidyverse* approach, you could do this


```r
species66[,-1] %>% rowwise() %>% pmap_dbl(.,sum)
```

```
##  [1]  522.0  876.0  331.0  678.0 1029.5  523.0  880.5 1054.5 1306.0 1196.5
## [11]  396.0  531.0  473.0  625.5  197.0  837.0  178.0  377.0  275.0  144.0
## [21]  723.0  624.5  391.0  577.0  441.0  442.0  347.5  419.0  181.0  307.0
## [31] 1341.0  788.5  616.5  452.5   59.0  389.5  244.0  370.0  460.0  837.0
## [41]  214.0  251.0  184.0  448.0  129.0  859.0  299.0  353.0  424.0  467.5
## [51]  461.0  625.0  536.0 1377.5  554.5  311.0  509.5  332.0 1574.0 2345.0
## [61]  562.5  574.0 1584.0  257.0  860.5 1901.0  688.0  622.0  985.0 2045.5
## [71]  485.5  282.0 1335.0  184.0  355.5  270.0  481.0  341.0   50.0  409.0
## [81]  731.0
```

Either way, a vector of numbers is not very informative... Let’s plot it in a histogram


```r
ggplot(data.frame(abundance = rowSums(species66[,-1])), 
       aes(abundance)) +
 geom_histogram()
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-10-1.png" width="672" />

## Species diversity

What about species number by plot? This is easy to achieve by just counting the number of positive entries in each row, which we get using the logical expression `>0`, like so


```r
rowSums(species66[,-1]>0)
```

```
##  [1] 51 50 52 64 44 44 66 53 31 19 27 43 37 34 26 46 14 51 37 15 67 52 51
## [24] 45 40 34 20 51 28 35 56 61 51 28  6 20 21 22 36 33 17 32 18 42 15 47
## [47] 25 36 36 12 39 49 25 49 35 36 39 39 74 71 24 58 79 25 44 74 14 33 51
## [70] 73 24 24 72 20 11 39 27 27  4 46 38
```

```r
ggplot(data.frame(species_number = rowSums(species66[,-1]>0)), 
       aes(species_number)) +
 geom_histogram()
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-11-1.png" width="672" />

What about the relationship between the two?


```r
ggplot(data.frame(species_number = rowSums(species66[,-1]>0),
                  abundance = rowSums(species66[,-1])),
       aes(species_number, abundance)) +
  geom_point() + 
  geom_smooth(method = "lm")
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-12-1.png" width="672" />

A strong positive relationship. One would usually expect a close relationship between abundance and species number, because having more plants in your plot means greater potential to sample more species. E.g. if you only have 10 individuals, the maximum number of species you can sample is 10... In Fynbos, the number of individuals you sample in a set area depends on the vegetation age, with greater numbers of smaller individuals in younger plots.

SIDE NOTE ON SPECIES DIVERSITY MEASURES: This is why I have been referring to **species number** and not **species richness**. Species richness, when one looks across all taxonomic groups that use the term, is typically defined as the number of species encountered for a given number of individuals sampled. This accounts for any differences in the amount of effort invested in each sampling event (e.g. trawl netting, mist netting, aerial counts, etc). It is only when working with sedentary organisms like plants that we typically sample per unit area, in which case we are actually sampling **species density** and NOT species richness. While useful, species density has different properties to species richness that we should always bear in mind - e.g. species density typically declines with the size of the organism sampled (which depends on the kind of organism (tree vs grass) or the age of the organism (seedling vs adult)). One can estimate species richness from vegetation plot data, but that requires counting the number of individuals and estimating the average number of species on samples for a given number of individuals. Read Gotelli and Colwell 2001. Ecology Letters if you think this is important to your work…


## Comparison between surveys

So let's have a look at the change in diversity within sites between surveys. We can use the same code as above, just plotting the species counts from the one year against the other.


```r
ggplot(data.frame(species_number66 = rowSums(species66[,-1]>0), 
                  species_number10 = rowSums(species10[,-1])), 
       aes(species_number66, species_number10)) +
  geom_point() + 
  geom_smooth(method = "lm")
```

```
## Error in data.frame(species_number66 = rowSums(species66[, -1] > 0), species_number10 = rowSums(species10[, : arguments imply differing number of rows: 81, 63
```

Oh dear! The number of plots surveyed in each year was different for some reason. This is where the plot numbers come in. We need to make sure we're comparing the same plots. First, we need to see what plot numbers are common between years.


```r
(plots <- intersect(species66$...1, species10$...1))
```

```
##  [1] "CP_1"  "CP_10" "CP_12" "CP_13" "CP_14" "CP_15" "CP_16" "CP_17"
##  [9] "CP_18" "CP_19" "CP_2"  "CP_21" "CP_22" "CP_27" "CP_28" "CP_29"
## [17] "CP_3"  "CP_31" "CP_34" "CP_36" "CP_37" "CP_38" "CP_39" "CP_4" 
## [25] "CP_40" "CP_42" "CP_44" "CP_45" "CP_46" "CP_47" "CP_48" "CP_49"
## [33] "CP_50" "CP_55" "CP_56" "CP_57" "CP_58" "CP_59" "CP_61" "CP_62"
## [41] "CP_63" "CP_64" "CP_65" "CP_66" "CP_67" "CP_70" "CP_71" "CP_72"
## [49] "CP_73" "CP_75" "CP_76" "CP_78" "CP_79" "CP_8"  "CP_80" "CP_82"
## [57] "CP_83" "CP_88" "CP_89" "CP_9"  "CP_92" "CP_95" "CP_99"
```

Note how putting `()` around the statement printed the results instead of just assigning the values to `plot`.

Also note, that if you get an error, it may be because you don't have a column called '...1'. This was an automatic name R gave the column. On Mac it may be 'X__1'. If you get an error, run the code `names(species66)[1]` to see what it called it on your computer and change the column name in the lines of code above and below.

Now we need to extract these "intersect()ing" plots from each survey's community data matrix, like so


```r
species66c <- species66[which(species66$...1 %in% plots), -1]
species10c <- species10[which(species10$...1 %in% plots), -1]
```

Now let's try to look at the change in diversity within sites between surveys again.


```r
ggplot(data.frame(species_number66 = rowSums(species66c>0), 
                  species_number10 = rowSums(species10c>0)), 
       aes(species_number66, species_number10)) +
  geom_point() + 
  geom_smooth(method = "lm")
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-16-1.png" width="672" />


Ok, a lot of difference between the two surveys. One thing we haven't accounted for is the fact that the surveys record all species, but we exclude the "seasonally apparent" species and only analyze the data for the "permanently apparent" species. This is because some species are only visible for a few weeks or less (annuals, some geophytes, etc), but we only survey the plots once - and not every week for a full year. We also exclude alien species.

You'll note that one of the spreadsheets in our dataset is called "excluded_spp"


```r
excel_sheets("pnas.1619014114.sd01.xlsx")
```

```
##  [1] "METADATA"           "weather"            "fires"             
##  [4] "postfireweather"    "enviroment"         "excluded_spp"      
##  [7] "veg1966"            "veg1996"            "veg2010"           
## [10] "traits"             "speciesclimatedata"
```

We need to read these in...


```r
excludes <- read_excel("pnas.1619014114.sd01.xlsx", sheet = "excluded_spp")
```

```
## New names:
## * `` -> ...1
```

```r
head(excludes)
```

```
## # A tibble: 6 x 2
##    ...1 x                    
##   <dbl> <chr>                
## 1     1 Acacia saligna       
## 2     2 Adenocline pauciflora
## 3     3 Agapanthus walshii   
## 4     4 Albuca flaccida      
## 5     5 Albuca juncifolia    
## 6     6 Annesorhiza altiscapa
```

...and cut them out of our community data matrices, and plot again


```r
species66c <- species66c[, -which(colnames(species66c) %in% excludes$x)]
species10c <- species10c[, -which(colnames(species10c) %in% excludes$x)]

ggplot(data.frame(species_number66 = rowSums(species66c>0), 
                  species_number10 = rowSums(species10c>0)), 
       aes(species_number66, species_number10)) +
  geom_point() + 
  geom_smooth(method = "lm")
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-19-1.png" width="672" />

...very similar. So why the difference?

Well it could be the differences in post-fire vegetation age and the impacts on the numbers of individuals, and thus species sampled. Let's look at the relationship between the change in numbers of species and change in vegetation age.


## Environmental data

Vegetation age for each plot at the time of each survey is in the `enviroment` spreadsheet. NOTE that I spelled it incorrectly in the data I submitted...


```r
env <- read_excel("pnas.1619014114.sd01.xlsx", sheet = "enviroment", na = "NA")
```

```
## New names:
## * `` -> ...1
```

```r
head(env)
```

```
## # A tibble: 6 x 10
##    ...1  Plot Moisture_class Age1996 Age2010 Age1966 Aliens_max
##   <dbl> <dbl>          <dbl>   <dbl>   <dbl>   <dbl>      <dbl>
## 1     1     1              5      10      10       8          3
## 2     2     2              5      10      10       5          1
## 3     3     3              5      10      10       7          1
## 4     4     4              4      10      24      18          3
## 5     5     8              5      10       3       3          3
## 6     6     9              5      10      24       3          2
## # … with 3 more variables: firecount66_10 <dbl>, firecount66_96 <dbl>,
## #   firecount96_10 <dbl>
```

Ok, first we need to match the plots in the `env` object to those in the community data matrices. This is a little tricky, because the `Plot` column in the `env` object only has numbers, whereas in the community data matrices they were prefixed with "CP_". We'll have to `paste()` them in, lie so


```r
(env$...1 <- paste("CP_", env$Plot, sep = ""))
```

```
##  [1] "CP_1"   "CP_2"   "CP_3"   "CP_4"   "CP_8"   "CP_9"   "CP_10" 
##  [8] "CP_12"  "CP_13"  "CP_14"  "CP_15"  "CP_16"  "CP_17"  "CP_18" 
## [15] "CP_19"  "CP_21"  "CP_22"  "CP_23"  "CP_24"  "CP_25"  "CP_27" 
## [22] "CP_28"  "CP_29"  "CP_30"  "CP_31"  "CP_34"  "CP_36"  "CP_37" 
## [29] "CP_38"  "CP_39"  "CP_40"  "CP_42"  "CP_44"  "CP_45"  "CP_46" 
## [36] "CP_47"  "CP_48"  "CP_49"  "CP_50"  "CP_54"  "CP_55"  "CP_56" 
## [43] "CP_57"  "CP_58"  "CP_59"  "CP_60"  "CP_61"  "CP_62"  "CP_63" 
## [50] "CP_64"  "CP_65"  "CP_66"  "CP_67"  "CP_68"  "CP_70"  "CP_71" 
## [57] "CP_72"  "CP_73"  "CP_74"  "CP_75"  "CP_76"  "CP_78"  "CP_79" 
## [64] "CP_80"  "CP_81"  "CP_82"  "CP_83"  "CP_84"  "CP_86"  "CP_87" 
## [71] "CP_88"  "CP_89"  "CP_90"  "CP_91"  "CP_92"  "CP_94"  "CP_95" 
## [78] "CP_97"  "CP_98"  "CP_99"  "CP_100"
```

Note that you may need to change the '...1' above to whatever your first column of the `species10` dataframe is named. Otherwise the next line of code returns an empty object and the plotting fails.


```r
env <- merge(species10[,1], env)
```

Note that I named the plot label column "X__1" to match the name in `species10`. This was essential because the `merge()` function operates only on the matched column names, of which "X__1" was the only one. The advantage of using `merge()` is that it retains only the rows that were matched, and it sorts them into the same order as the first object supplied to `merge()`.


```r
ggplot(data.frame(change_in_species_number = rowSums(species66c>0) - rowSums(species10c>0), 
                  change_in_age = env$Age1966 - env$Age2010), 
       aes(change_in_species_number, change_in_age)) +
  geom_point() + 
  geom_smooth(method = "lm")
```

```
## Warning: Removed 6 rows containing non-finite values (stat_smooth).
```

```
## Warning: Removed 6 rows containing missing values (geom_point).
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-23-1.png" width="672" />

So vegetation age clearly is an issue, with numbers of species declining as the vegetation ages, or increasing after a plot burns.

Another approach is to "rarefy" the samples. This involves estimating each plot's expected species richness by randomly sampling a set number of individuals from each community. In this case we'll sample 50 individuals, because that is the minimum abundance recorded in our plots.


```r
ggplot(data.frame(species_richness66 = rarefy(x = ceiling(species66c), sample = 50), 
                  species_richness10 = rarefy(x = ceiling(species10c), sample = 50)), 
       aes(species_richness66, species_richness10)) +
  geom_point() + 
  geom_smooth(method = "lm")
```

<img src="/docs/learningR/demo-2-biodiversity_files/figure-html/unnamed-chunk-24-1.png" width="672" />

Much more similar... but still some differences. It looks like there may be fewer species in 2010, but this should be revealed by the slope of a regression line. We can get the slope by looking at the `summary()` of the linear model (`lm()`),


```r
sr66 <- rarefy(x = ceiling(species66c), sample = 50)
sr10 <- rarefy(x = ceiling(species10c), sample = 50)

summary(lm(sr10 ~ sr66 - 1))
```

```
## 
## Call:
## lm(formula = sr10 ~ sr66 - 1)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -7.0426 -1.9667 -0.1091  2.9906  8.0104 
## 
## Coefficients:
##      Estimate Std. Error t value Pr(>|t|)    
## sr66  0.79423    0.02309    34.4   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.51 on 62 degrees of freedom
## Multiple R-squared:  0.9502,	Adjusted R-squared:  0.9494 
## F-statistic:  1183 on 1 and 62 DF,  p-value: < 2.2e-16
```

Note that I forced the intercept to be 0 by adding "-1" to the formula. The interesting thing is that the slope is 0.7942285, which suggests lower species numbers in 2010...

We can also test this using a paired T test


```r
t.test(sr66, sr10, paired = T)
```

```
## 
## 	Paired t-test
## 
## data:  sr66 and sr10
## t = 6.9289, df = 62, p-value = 2.854e-09
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  2.471771 4.476248
## sample estimates:
## mean of the differences 
##                3.474009
```

Yup, there was a significant drop in species numbers between 1966 and 2010, even once we accounted for differences in vegetation age/numbers of individuals.


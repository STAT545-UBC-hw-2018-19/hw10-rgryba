---
title: "Homework 10 - web scraping using httr"
author: "Rowenna Gryba"
date: "04 December, 2018"
output:
  html_document:
    keep_md: yes
---

# Scraping with `httr`
## Read in required libraries

```r
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(httr))
suppressPackageStartupMessages(library(knitr))
suppressPackageStartupMessages(library(kableExtra))
```

## eBirds website - take 2!
Getting info off the eBirds site seemed overly hard with `rvest` - trying it again and we shall see if it's easier.

Let's get the most recent notable sightings for Alaska.
Yup! API makes it much easier. 

Let's try and get the recent sightings from Alaska.

First load the data.

```r
eB <- GET("https://ebird.org/ws2.0/data/obs/US-AK/recent?detail=full&key=4djb9hct7gta")
names(eB)
```

```
##  [1] "url"         "status_code" "headers"     "all_headers" "cookies"    
##  [6] "content"     "date"        "times"       "request"     "handle"
```

```r
status_code(eB) #status 200 - so good to go
```

```
## [1] 200
```

Next let's build a table of the species sighted and the date they were seen.

```r
ebAK <- content(eB) 
ebAK <- ebAK %>% {
	tibble(
		sp = map_chr(., "sciName"),
		date = map(., "obsDt")
	)
}

kable(head(ebAK)) %>%
	kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> sp </th>
   <th style="text-align:left;"> date </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Poecile atricapillus </td>
   <td style="text-align:left;"> 2018-12-04 15:15 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bonasa umbellus </td>
   <td style="text-align:left;"> 2018-12-04 15:15 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sitta canadensis </td>
   <td style="text-align:left;"> 2018-12-04 15:15 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Dryobates pubescens </td>
   <td style="text-align:left;"> 2018-12-04 15:15 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Junco hyemalis </td>
   <td style="text-align:left;"> 2018-12-04 15:15 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Poecile hudsonicus </td>
   <td style="text-align:left;"> 2018-12-04 15:15 </td>
  </tr>
</tbody>
</table>

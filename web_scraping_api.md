---
title: "Homework 10 - web scraping using httr"
author: "Rowenna Gryba"
date: "05 December, 2018"
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
Yup! API makes it much easier. 

Next let's build a table of the species sighted and the date they were seen.

```r
ebAK <- content(eB) 
ebAK <- ebAK %>% {
	tibble(
		species = map_chr(., "sciName"),
		date = map(., "obsDt")
	)
}

kable(head(ebAK)) %>%
	kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> species </th>
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

Now let's get the taxonomy they use for the bird species listed - the scientific name, common name and species code.

```r
eBhs <- GET("https://ebird.org/ws2.0/ref/taxonomy/ebird?locale=en&fmt=json&key=4djb9hct7gta")
status_code(eBhs) #status 200 - so good to go
```

```
## [1] 200
```

```r
ebAKhs <- content(eBhs)

ebAKhs <- ebAKhs %>% {
	tibble(
		sci = map_chr(., "sciName"),
		common = map_chr(., "comName"),
		code = map_chr(., "speciesCode")
	)
}

write.csv(ebAKhs, "eBirdsTax.csv")

kable(head(ebAKhs)) %>%
	kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> sci </th>
   <th style="text-align:left;"> common </th>
   <th style="text-align:left;"> code </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Struthio camelus </td>
   <td style="text-align:left;"> Common Ostrich </td>
   <td style="text-align:left;"> ostric2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Struthio molybdophanes </td>
   <td style="text-align:left;"> Somali Ostrich </td>
   <td style="text-align:left;"> ostric3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Struthio camelus/molybdophanes </td>
   <td style="text-align:left;"> Common/Somali Ostrich </td>
   <td style="text-align:left;"> y00934 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Rhea americana </td>
   <td style="text-align:left;"> Greater Rhea </td>
   <td style="text-align:left;"> grerhe1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Rhea pennata </td>
   <td style="text-align:left;"> Lesser Rhea </td>
   <td style="text-align:left;"> lesrhe2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Rhea pennata tarapacensis/garleppi </td>
   <td style="text-align:left;"> Lesser Rhea (Puna) </td>
   <td style="text-align:left;"> lesrhe4 </td>
  </tr>
</tbody>
</table>

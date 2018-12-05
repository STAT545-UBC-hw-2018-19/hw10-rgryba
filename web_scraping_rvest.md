---
title: "Homework 10 - web scraping with rvest"
author: "Rowenna Gryba"
date: "04 December, 2018"
output:
  html_document:
    keep_md: yes
---

# Scraping with `rvest`
## Read in required libraries

```r
suppressPackageStartupMessages(library(tidyverse))
suppressPackageStartupMessages(library(rvest))
suppressPackageStartupMessages(library(knitr))
suppressPackageStartupMessages(library(kableExtra))
suppressPackageStartupMessages(library(purrr))
```

## eBird website
In this first task I am taking the list of species that have undergone a taxonimic split and providing a table of which species they have been split into.


```r
#web scraping
split <- read_html("https://ebird.org/science/the-ebird-taxonomy") %>% 
	html_nodes("ul:nth-child(45) em:nth-child(1)") %>%
	html_text()

split <- tibble(split)

splitInto <- read_html("https://ebird.org/science/the-ebird-taxonomy") %>% 
	html_nodes("li:nth-child(1) em:nth-child(3) , ul:nth-child(45) em:nth-child(2)") %>%
	html_text()

splitInto <- tibble(splitInto)

#small data clean up for one of the genus names and divide the names into 2 columns
splitInto <- splitInto %>%
	separate(splitInto, c("genus", "sp2", "spp") , sep = " ") %>%
	mutate(genus = recode(genus, `O.` = "Oceanodroma"))
```

```
## Warning: Expected 3 pieces. Missing pieces filled with `NA` in 13 rows [1,
## 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13].
```

```r
split <- split %>%
	separate(split, c("genus", "sp1"), sep = " ")

#join the tables
splitCombo <- full_join(split, splitInto, by = "genus") %>%
	mutate(SpSplit = paste(genus, sp1, sep = " "),
				 SpInto = paste(genus, sp2, spp, sep = " "))
	
splitCombo <- splitCombo %>%
	select(SpSplit, SpInto)

#export the table
write.csv(splitCombo, "splitCombo.csv")

#what the table looks like for good measure
kable(head(splitCombo), format = "html") %>%
	kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> SpSplit </th>
   <th style="text-align:left;"> SpInto </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Sarkidiornis melanotos </td>
   <td style="text-align:left;"> Sarkidiornis melanotos NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sarkidiornis melanotos </td>
   <td style="text-align:left;"> Sarkidiornis sylvicola NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Anas platyrhynchos </td>
   <td style="text-align:left;"> Anas diazi NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Melanitta fusca </td>
   <td style="text-align:left;"> Melanitta deglandi NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Megascops guatemalae </td>
   <td style="text-align:left;"> Megascops guatemalae)  NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Zimmerius vilissimus </td>
   <td style="text-align:left;"> Zimmerius vilissimus NA </td>
  </tr>
</tbody>
</table>

Next task with `rvest` will be to find the bird sightings recorded in Alaska, who did it and when.

```r
ebAKrvest <- read_html("https://ebird.org/region/US-AK?yr=all") %>% 
	html_table(fill = T)  %>%
	.[[1]] 

ebAKrvest <- ebAKrvest %>% 
		select(Observer, Date)

write.csv(ebAKrvest, "eBirdsAKobsDate.csv")

kable(head(ebAKrvest), format = "html") %>%
	kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Observer </th>
   <th style="text-align:left;"> Date </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Robin Collman </td>
   <td style="text-align:left;"> 4 Dec 2018 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Robin Corcoran </td>
   <td style="text-align:left;"> 4 Dec 2018 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Justin Saunders </td>
   <td style="text-align:left;"> 4 Dec 2018 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lynn Barber </td>
   <td style="text-align:left;"> 4 Dec 2018 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Justin Saunders </td>
   <td style="text-align:left;"> 4 Dec 2018 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Medium Yellowlegs </td>
   <td style="text-align:left;"> 4 Dec 2018 </td>
  </tr>
</tbody>
</table>

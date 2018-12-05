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

## James Bond actors
Next task with `rvest` will be to determine the number of times an actor has been in a James Bond movie and which role.

```r
jbActors <- read_html("https://www.the-numbers.com/movies/franchise/James-Bond#tab=summary") %>% 
	html_table() %>%
	.[[3]] %>%
	select(Person, `Nr. ofMovies`, Role)

kable(head(jbActors), format = "html") %>%
	kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Person </th>
   <th style="text-align:right;"> Nr. ofMovies </th>
   <th style="text-align:left;"> Role </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Desmond Llewelyn </td>
   <td style="text-align:right;"> 17 </td>
   <td style="text-align:left;"> Q </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lois Maxwell </td>
   <td style="text-align:right;"> 14 </td>
   <td style="text-align:left;"> Miss Moneypenny </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bernard Lee </td>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:left;"> M </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Roger Moore </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:left;"> James Bond </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Judi Dench </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:left;"> M </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sean Connery </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:left;"> James Bond </td>
  </tr>
</tbody>
</table>

P.S. The reason I switched to James Bond is that the eBirds site is complicated!!! Going to try it with `httr` and see if it's any better.

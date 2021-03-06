---
layout: page
title: Metadata
subtitle: Has anyone seen my keys?
comments: true
---

Metadata is *data about data*

hand-made artisanal metadata
----------------------------

You can also do this simply by simple text (`.txt` or `.rtf`) with your own hands:

![brom](img/bromeliad_metadata.png)

Alternatively, you could write it in a CSV format:

    field,description
    country,"countries of the world""
    continent,"continents of the world"
    year,"the year AD"
    lifeExp,"lifeExp in years"
    pop,"population in humans"
    gdpPercap,"gdp per capita, measured in PPP"

This is convenient, but has no standard whatsoever. This makes it difficult for machines to read. An increasing number of scientists are experimenting with new ways of creating metadata that satisfies our "write data for machines" principle.

Many of these methods are novel, and rather experimental. We're going to be using some Very Modern Packages in today's exercises. We'll need to install versions of packages that are newer than what is available on CRAN (via `install.packages()`). We'll use a small package called `ghit` to do the job:

``` r
# install.packages("ghit")
```

As usual, our gapminder dataset will supply some good examples:

``` r
library(gapminder)
```

intro to JSON and XML (and YAML)
--------------------------------

These are the two common languages of the internet: **J**ava**S**cript **O**bject **N**otation and e**X**tensible **M**arkup **L**anguage.

Here's an example of JSON: from [this wonderful site](https://zapier.com/learn/apis/chapter-3-data-formats/)

``` javascript
{
  "crust": "original",
  "toppings": ["cheese", "pepperoni", "garlic"],
  "status": "cooking",
  "customer": {
    "name": "Brian",
    "phone": "573-111-1111"
  }
}
```

And here is XML:

``` xml
<order>
    <crust>original</crust>
    <toppings>
        <topping>cheese</topping>
        <topping>pepperoni</topping>
        <topping>garlic</topping>
    </toppings>
    <status>cooking</status>
</order>
```

and finally, *Y*et *A*nother *M*arkup *L*anguage:

    order:
      - crust: original
         topping: 
            - cheese
            - pepperoni
            - garlic

the classic approach: the Ecological Metadata Language (`EML`)
--------------------------------------------------------------

EML is a classic metadata approach. There is a [large amount of documentation](http://im.lternet.edu/sites/im.lternet.edu/files/emlHandbook.pdf).

It is used in the Knowledge Network for Biocomplexity

    library(ghit)
    install_github("ropensci/EML", dependencies=c("Depends", "Imports"))
    library(EML)

``` r
dat <- data.frame(river = factor(c("SAC",  
                                   "SAC",   
                                   "AM")),
                  spp   = c("Oncorhynchus tshawytscha",  
                            "Oncorhynchus tshawytscha", 
                            "Oncorhynchus kisutch"),
                  stg   = ordered(c("smolt", 
                                    "parr", 
                                    "smolt"), 
                                  levels=c("parr", 
                                           "smolt")), # => parr < smolt
                  ct    = c(293L,    
                            410L,    
                            210L),
                  day   = as.Date(c("2013-09-01", 
                                    "2013-09-1", 
                                    "2013-09-02")),
                  stringsAsFactors = FALSE)
```

now we define the column units:

``` r
col.defs <- c("River site used for collection",
              "Species scientific name",
              "Life Stage", 
              "count of live fish in traps",
              "The day on which traps were sampled")


unit.defs <- list(
  c(SAC = "The Sacramento River",     # Factor 
    AM = "The American River"),
 "Scientific name",                   # Character string 
  c(parr = "third life stage",        # Ordered factor 
    smolt = "fourth life stage"),
  c(unit = "number", 
    precision = 1, 
    bounds = c(0, Inf)),              # Integer
  c(format = "YYYY-MM-DD",            # Date
    precision = 1))
```

And finally, we write it out:

``` r
eml_write(dat, 
          col.defs = col.defs, 
          unit.defs = unit.defs, 
          creator = "Carl Boettiger <cboettig@ropensci.org>", 
          file = "EML_example.xml")
```

get the computer to write it: `CSVY`
------------------------------------

CSVY is a new idea, that embeds the metadata file into the very `.csv` file. The whole idea is described on [csvy.org](http://csvy.org/). It uses a metadata standard called `YAML`:

    ---
    name: my-dataset
    fields:
      - name: var1
        title: variable 1
        type: string
        description: explaining var1
        constraints:
          - required: true
      - name: var2
        title: variable 2
        type: integer
      - name: var3
        title: variable 3
        type: number
    ---
    var1,var2,var3
    A,1,2.5
    B,3,4.3

We're going to focus on this process, (but not on the constraints)

Let's start by using `rio` to write the data out:

``` r
library(rio)

attributes(gapminder)[1:2]
```

    ## $names
    ## [1] "country"   "continent" "year"      "lifeExp"   "pop"       "gdpPercap"
    ## 
    ## $class
    ## [1] "tbl_df"     "tbl"        "data.frame"

``` r
export(gapminder, "gapminder.csvy", format = "csvy", comment_header = FALSE)
```

    ## Warning in write.table(x, file = file, sep = sep, row.names = row.names, :
    ## appending column names to file

``` r
head(import("gapminder.csvy"))
```

    ##       country continent year lifeExp      pop gdpPercap
    ## 1 Afghanistan      Asia 1952  28.801  8425333  779.4453
    ## 2 Afghanistan      Asia 1957  30.332  9240934  820.8530
    ## 3 Afghanistan      Asia 1962  31.997 10267083  853.1007
    ## 4 Afghanistan      Asia 1967  34.020 11537966  836.1971
    ## 5 Afghanistan      Asia 1972  36.088 13079460  739.9811
    ## 6 Afghanistan      Asia 1977  38.438 14880372  786.1134

now let's edit it by hand.

### Note: the JSON table schema

     field     field
       |         |
       |         |
       V         V
     
      A     |    B    |    C    |    D      <--- Row
      ------------------------------------
      valA  |   valB  |  valC   |   valD    <--- Row
      ...

We're going to work with **field descriptors**

``` json
  "name": "name of field (e.g. column name)",
  "title": "A nicer human readable label or title for the field",
  "type": "A string specifying the type",
  "format": "A string specifying a format",
  "description": "A description for the field"
```

Note that there are no quotes when we use YAML.

datapackage
-----------

Using the R package [`dmpr`](https://github.com/christophergandrud/dpmr). It creates data packages after the standard described in [datapackage](http://data.okfn.org/doc/tabular-data-package)

``` r
#install.package("dpmr")

library(dpmr)
meta_list <- list(name = 'gapminder_data',
                    title = 'the data of Gapminder',
                    last_updated = Sys.Date(),
                    version = '0.1',
                    license = data.frame(type = 'PDDL-1.0',
                            url = 'http://opendatacommons.org/licenses/pddl/')
                            )

datapackage_init(df = gapminder, meta = meta_list) 
```

Resources
---------

-   [Metadata Guide](http://www.ands.org.au/guides/metadata-working.html) from Australian National Data Service (ANDS)
-   [Metadata Directory](http://rd-alliance.github.io/metadata-directory/) from Research Data Alliance
-   [Best Practices for Data Management](https://www.dataone.org/sites/all/documents/DataONE_BP_Primer_020212.pdf) from DataONE
-   [Ten Simple Rules for the Care and Feeding of Scientific Data](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003542) by Goodman et al
-   \[Challenges to Open Data and How to Respond\] (04c\_OpenDataChallengesQuickInfoSheet.md)

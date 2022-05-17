---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: R
  language: r
  name: ir
---

# Describing communities using summary statistics

Here we will take field data from invertebrates collected during the 2022 Kosciuszko National Park field trip and compute descriptive statistics to summarise the populations

```{code-cell}
:tags: ["remove-output"]
library(tidyverse)
```

```{code-cell}
inverts <- read_csv("https://raw.githubusercontent.com/mikheyev/ecology-r-code/main/data/inverts.csv")
```

## Define functions for diversity and evenness

[Menhinick richness index](https://search.r-project.org/CRAN/refmans/abdiv/html/menhinick.html) measures diversity without considering relative species abundance.It's relatively crude, but simple to compute.

```{code-cell}
D <- function(n) {
    if (sum(n) > 0)
        sum(n > 0)/sqrt(sum(n))
    else
        0
}
```

[Shannon's index](https://en.wikipedia.org/wiki/Diversity_index#Shannon_index) combines species abundance to provide incorporate how abundance affects diversity.

```{code-cell}
H <- function(n) {
    partH <- 0
    for (i in n)
        if (i > 0)
            partH = partH - (i / sum(n)) * log(i / sum(n))
    return(partH)
}
```

[Evenness](https://en.wikipedia.org/wiki/Species_evenness) is a measure of how similar the abundances of different species are in the community, and it is derived from dividing Shannon's index by the natural log of the species count, which actually corresponds to the maximum possible H, for a given number of species.

```{code-cell}
E <- function(n) {
    H(n)/log(sum(n > 0))
}
```

## reshape data to make it easier to analyze

The following commands reshape the data from the easy-to-enter-by-humans 'wide' format to the `long` format that R likes. The `starts_with` command says that the variables we need to reshape start with "Pack", which is the leaf pack that is our unit of replication. 

```{code-cell}
head(inverts)

inverts_long <- pivot_longer(inverts, starts_with("Pack"), names_to = "pack", values_to = "count")

head(inverts_long)
```

The set of calculations below is a bit hairy, but it illustrates the power of data manipulation in R, we do some cleaning and compute statistics to plot on the fly.

```{code-cell}
# take `inverts_long` and assign the final result to `inverts_summary`
inverts_summary <- inverts_long %>%  
  # remove any counts with missing data
  filter(!is.na(count)) %>% 
  # conduct measurements at the level of group and pack (our units of replication)
  group_by(Group, pack) %>% 
  # variables to compute -- not we're using the newly defined functions
  summarize(Elevation = first(Elevation), flow = first(`Mean flow rate (m/s)`), D = D(count), H = H(count), E = E(count) ) 
```

Take time to walk through the commands and make sure you understand what is going on in each line.
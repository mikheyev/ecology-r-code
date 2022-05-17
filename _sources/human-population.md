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

# Estimating human population growth parameters

In this exercise, we will estimate the intrinsic growth rate $r$ and density-dependent growth rate $s$ for human population data using linear regression. This is basically just a way to draw a line with a slope and an intercept to data in a way that fits the data the best. There are fancy statistics associated with this technique, but we will focus only on the value estimates and not worry about them.

This technique is very general and we will use it throughout the course for many other data sets.

First we read human population data.

```{code-cell}
:tags: ["remove-output"]
library(tidyverse)
```

```{code-cell}
pop <- read_csv("https://raw.githubusercontent.com/mikheyev/ecology-r-code/main/data/humans.csv")
pop
```

We can then plot the data over time

```{code-cell}
pop %>% ggplot(aes(x = year, y = population)) + geom_line()
```

As we can see the population continues to increase, with no obvious pattern. Is it exponential? Let's find out!
To do this, we will compute per capita population change

```{code-cell}
pop <- pop %>% mutate(perCapChange = (lead(population) - population)/population/(lead(year) - year))
pop
```

Note: We are taking $\Delta N$ and $\Delta t$ using the `lead` command. All it code does is take all the values of a column pairwise, and subtracts the *leading* value from the one before it. This is why there is a `NA` in the last row -- there is no corresponding value for `lead` to take.

*Question:* Examine the new pop variable, at when did the human population have the highest per capita growth rate?


```{code-cell}
pop %>% ggplot(aes(x = population, y = perCapChange)) + geom_point()
```

Now we can see that something interesting is happening -- there is a change in how the population grows at about 3.4 billion people. We can a line with a slope $s$ and an intercept $r$ to the these data using linear regression.

We start at looking at the earlier time in the human population. The `lm` and `summary` commands fit the data and display the results, in that order
```{code-cell}
summary(lm(perCapChange ~ population, data = filter(pop, population < 3.4)))
```

Note that `(Intercept)` is the x-intercept, or $r$. The variable `population` is the independent variable in the data and is the slope, or $s$.

Similarly, we can plot the values for the latter stage of the human population

```{code-cell}
summary(lm(perCapChange ~ population, data = filter(pop, population > 3.4)))
```

We can plot the lines with the parameter estimates, which gives us Figure 6.3 from the textbook.

```{code-cell}
pop %>% ggplot(aes(x = population, y = perCapChange)) + geom_point() +
  geom_abline(intercept = 0.0296044, slope =-0.0026489, color = "blue") +
  geom_abline(intercept = -0.0014347, slope =0.0067378, color = "red") +
  annotate("text", x = 3.8, y = .022, label = "1965")
```
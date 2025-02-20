# (PART\*) References {.unnumbered}

# References {-}

<div id="refs"></div>

# (APPENDIX) Appendix {-} 

# Cover design

The cover of the book is inspired by the fast growing generative art community in R.\index{Generative art}
Generative art refers to art that in whole or in part has been created with the use of an autonomous system. 
Instead of creating random dynamics we rely on what is core to the book: The evolution of financial markets. 
Each circle in the cover figure corresponds to daily market return within one year of our sample. Deviations from the circle line indicate positive or negative returns. 
The colors are determined by the standard deviation of market returns during the particular year. 
The few lines of code below replicate the entire figure. 
We use the Wes Andersen color palette (also throughout the entire book), provided by the package ´wesanderson´ [@wesanderson]


```r
library(tidyverse)
library(lubridate)
library(RSQLite)
library(wesanderson)

tidy_finance <- dbConnect(
  SQLite(),
  "data/tidy_finance.sqlite",
  extended_types = TRUE
)

factors_ff_daily <- tbl(
  tidy_finance,
  "factors_ff_daily"
) |>
  collect()

data_plot <- factors_ff_daily |>
  select(date, mkt_excess) |>
  group_by(year = floor_date(date, "year")) |>
  mutate(group_id = cur_group_id())

data_plot <- data_plot |>
  group_by(group_id) |>
  mutate(
    day = 2 * pi * (1:n()) / 252,
    ymin = pmin(1 + mkt_excess, 1),
    ymax = pmax(1 + mkt_excess, 1),
    vola = sd(mkt_excess)
  ) |>
  filter(year >= "1962-01-01")

levels <- data_plot |>
  distinct(group_id, vola) |>
  arrange(vola) |>
  pull(vola)

cp <- coord_polar(
  direction = -1,
  clip = "on"
)

cp$is_free <- function() TRUE

colors <- wes_palette("Zissou1",
  n_groups(data_plot),
  type = "continuous"
)

plot <- data_plot |>
  mutate(vola = factor(vola, levels = levels)) |>
  ggplot(aes(
    x = day,
    y = mkt_excess,
    group = group_id,
    fill = vola
  )) +
  cp +
  geom_ribbon(aes(
    ymin = ymin,
    ymax = ymax,
    fill = vola
  ), alpha = 0.90) +
  theme_void() +
  facet_wrap(~group_id,
    ncol = 10,
    scales = "free"
  ) +
  theme(
    strip.text.x = element_blank(),
    legend.position = "None",
    panel.spacing = unit(-5, "lines")
  ) +
  scale_fill_manual(values = colors)

ggsave(
 plot = plot,
 width = 10,
 height = 6,
 filename = "cover.jpg",
 bg = "white"
)
```

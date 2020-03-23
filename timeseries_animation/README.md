
# Animating timeseries

``` r
library(tidyverse)
library(lubridate)
library(janitor)
library(ggtext)
library(gganimate)
```

``` r
cases <- read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Confirmed.csv")
```

The following visualisation is inspired by John Burn-Murdoch work.
\#855C75,\#D9AF6B,\#AF6458,\#736F4C,\#526A83,\#625377,\#68855C,\#9C9C5E,\#A06177,\#8C785D,\#467378,\#7C7C7C

``` r
cases <- cases %>%
  pivot_longer(cols= 5:ncol(.), names_to = "date", values_to = "cases")%>% 
  mutate(date = mdy(date)) %>% 
  clean_names()%>% 
  group_by(country_region, date) %>% 
  summarise(cases = sum(cases)) %>% 
  filter(country_region != "Others" & country_region != "China") %>%
  group_by(country_region) %>%
  mutate(days_since_100 = as.numeric(date-min(date[cases >= 100]))) %>%
  ungroup()%>%
  filter(is.finite(days_since_100)) %>% 
  group_by(country_region) %>%
  mutate(new_cases = cases-cases[days_since_100 == 0])%>%
  filter(sum(cases >= 100) >= 5) %>%
  filter(cases >= 100) %>%
  ungroup() %>% 
  mutate(color_label = case_when(
    country_region == "Switzerland" ~ "#D63D32",
    country_region == "Italy" ~ "#888888",
    country_region == "Iran" ~ "#6699CC",
    country_region == "Germany" ~ "#661100",
    country_region == "France" ~ "#882255",
    country_region == "United Kingdom" ~ "#999933",
    country_region == "US"~ "#44AA99",
    country_region == "Spain" ~ "#332288",
    country_region == "Korea, South" ~ "#117733",
    country_region == "Netherlands" ~ "#DDCC77",
    country_region == "Cruise Ship" ~ "#666666",
    country_region == "Hong Kong" ~ "#CC503E",
    country_region == "Singapore" ~ "#1D6996",
    country_region == "Japan" ~ "#855C75",
    TRUE ~ "grey90"
    ))
```

``` r
ggplot(data = cases, aes(days_since_100, cases, 
                         color = color_label,
                         group = country_region)) +
  geom_point(size = 0.8, alpha = 0.9)+
  geom_line(size = 0.7, alpha =0.9) +
  scale_y_log10(expand = expansion(add = c(0,0.1)), breaks=c(100, 500, 2000,  10000, 60000)) +
  geom_text(aes(x = 40, label = country_region), hjust = 0)+
  scale_color_identity()+
  theme_minimal() +
  theme(
    panel.grid.minor = element_blank(),
    legend.position = "none",
    plot.margin = margin(3,15,3,3,"mm")
  )+
  coord_cartesian(clip = "off") +
  labs(x = "Number of days since 100th case", 
       y = "Total Number of Cases",
       title = "Total number of COVID-19 cases",
       subtitle =  "Outside of China",
       caption = "Data Source: John Hopkins University") +
  transition_reveal(days_since_100)
```

![](README_files/figure-gfm/unnamed-chunk-4-1.gif)<!-- -->
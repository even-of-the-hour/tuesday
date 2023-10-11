When does Tuesday take place?
================

<<<<<<< HEAD
=======
    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

>>>>>>> ff258f4c06189b199404ff793fe93529b76adee3
## When does *Tuesday* take place?

In *Tuesday*, the moon is full and near the horizon. It’s fully dark.
It’s 8pm. It’s Tuesday. The frogs are flying. When and where is this
happening?

The position of the sun, along with the position and phase of the moon,
depend on the time and location of the observation. To explore the
question, let’s isolate one of the variables and pick a location. Say,
Rayne, Louisiana–the Frog Capital of the World, whose latitude and
longitude are 30.2349° N, 92.2685° W.

To determine all possible Tuesdays where the sky was dark in Rayne, we
need a full \[Metonic
cycle\])(<https://en.wikipedia.org/wiki/Metonic_cycle>) of Tuesdays.
Metonic cycles refer to the recurrence of lunar phases throughout the
solar calendar year–they last 19 years. So we can restrict our search to
a 19 year period, and if we don’t find a Very Special Flying Frog
Tuesday in Rayna, LA, there won’t be one ever. We can pick an arbitrary
19 years, so let’s pick the 19 years before *Tuesday* [was awarded the
Caldecott
medal](https://www.chicagotribune.com/news/ct-xpm-1992-01-28-9201090055-story.html).

We create the object `metonic_cycle` then restrict it to only Tuesdays
using `wday()`.

``` r
caldecott_award <- mdy("01-27-1992")

metonic_cycle <- seq.Date(from = caldecott_award - years(19), 
                          to = caldecott_award, 
                          by = "day")

tuesdays <- metonic_cycle[wday(metonic_cycle) == 3]
```

We will use `getSunlighttimes()` which can take a data.frame of dates,
latitudes, and longitudes. We have our dates in `tuesdays`, so we just
need to use `mutate()` to make a simple tibble with the coordinates for
Rayne, LA.

``` r
tuesdays_lat_long <- tibble(date = tuesdays) %>%
  mutate(lat = 30.2349,
         lon = -92.2685)

# getSunlightTimes() lets us grab any sunset/sunrise sort of thing; we just want to know when it's full night.

tuesday_nights <- getSunlightTimes(data = tuesdays_lat_long,
                                  keep = c("night"),
                                  tz = "US/Central") 
```

`getSunlightTimes()` returns a nice new dataframe which we’ll call
`tuesday_nights` with all the times when it’s full night in Rayne, LA on
all the dates of Tuesdays in the 19 year Metonic Cycle we identified.
For it to match *Tuesday*, though, it has to be dark by 8pm. (“TUESDAY
EVENING, AROUND EIGHT”, as the story goes.) We can use
`lubridate::hour()` to return just th hour of our `night` variable and
`filter()` to days when it’s fully dark by 8pm. We’ll call that new data
frame `tuesday_evening_around_eight`.

``` r
tuesday_evening_around_eight <- 
  tuesday_nights %>%
  filter(hour(night) >= 20) 
```

At this point there are 525 candidate Tuesdays–but we need to check
their moon phase to see when the moon is full. `getMoonIllumination()`,
also from the `suncalc` package, works similarly to
`getSunlightTimes()`, but just needs a `date` argument as [everyone sees
the same phases of the
moon](https://moon.nasa.gov/inside-and-out/top-moon-questions/).

But, but!! The moon does have to be above the horizon at 8pm, so the
frogs can see it. So we’ll do two steps here. First, define
`tuesday_full_moons` as Tuesdays (already including only dark-at-8pm
Tuesdays) when the moon is full. Second, run those candidate dates
through `getMoonPosition` to make sure the moon is visible (i.e., above
the horizon).

``` r
tuesday_full_moons <- 
  tuesday_evening_around_eight %>%
  mutate(getMoonIllumination(date = date, keep = "phase")) %>%
  # per getMoonIllumination() documentation, including phase values close to 1 as full.
  filter(phase > .95) 
  
tuesday_full_moons_frogs_can_see <- 
  tuesday_full_moons %>%
  getMoonPosition(data = .,
                  keep = "altitude") %>%
  mutate(VAL_altitude_deg = altitude * 180 * pi) %>%
  arrange(desc(altitude))
```

If *Tuesday* takes place in Rayne, LA, then it takes place on September
25, 1984.

``` r
tuesday_full_moons_frogs_can_see <- 
  tuesday_full_moons_frogs_can_see %>%
  filter(
      # Above the horizon
      VAL_altitude_deg > 0,
      
      # But not too high
      VAL_altitude_deg <= 25) %>% 
    as_tibble()
```

## Coming soon - Investigating Tuesdays in other cities

``` r
when_is_tuesday <- function(my_lat, my_lon) {
  
  metonic_cycle <- seq.Date(from = Sys.Date() - years(19), 
                          to = Sys.Date(), 
                          by = "day")
  
  tuesdays <- metonic_cycle[wday(metonic_cycle) == 3]
  
  
  tuesdays_lat_long <- tibble(date = tuesdays) %>%
    mutate(lat = my_lat,
           lon = my_lon)
  
  # getSunlightTimes() lets us grab any sunset/sunrise sort of thing; we just want to know when it's full night.
  
  tuesday_nights <- getSunlightTimes(data = tuesdays_lat_long,
                                     keep = c("night"),
                                     tz = "US/Eastern") 
  
  
    
  tuesday_evening_around_eight <- 
    tuesday_nights %>%
    filter(hour(night) >= 20) 
  
    
  tuesday_full_moons <- 
    tuesday_evening_around_eight %>%
    mutate(getMoonIllumination(date = date, keep = "phase")) %>%
    # per getMoonIllumination() documentation, including phase values close to 1 as full.
    filter(phase > .95) 
    
  tuesday_full_moons_frogs_can_see <- 
    tuesday_full_moons %>%
    getMoonPosition(data = .,
                    keep = "altitude") %>%
    mutate(VAL_altitude_deg = altitude * 180 * pi) %>%
    arrange(desc(altitude))
  
  tuesday_full_moons_frogs_can_see %>%
    filter(
      # Above the horizon
      VAL_altitude_deg > 0,
      
      # But not too high
      VAL_altitude_deg <= 25) %>% 
    as_tibble()
  
    
  
}
```

# How Much Cheaper Is a Damaged Car, Really?

Dataquest guided project, *Exploring eBay Car Sales Data*. Dataset: 50,000 used car listings scraped from eBay Kleinanzeigen, crawled March to April 2016.

## The question

**How much cheaper are cars with unrepaired damage than their undamaged counterparts?**

Comparing the two groups directly gives a **76% discount**. That is implausibly large, because damaged cars are not otherwise identical to undamaged ones: they are older and have more miles, and the naive comparison charges damage for all of it.

## Findings

### 1. The real discount is about 44%, not 76%

| Estimate | Discount |
|---|---|
| Naive comparison of group medians | 76.2% |
| Matched on brand, model, age band and mileage | **44.4%** |
| Interquartile range across 176 matched cells | 33.3% to 52.9% |

About **32 percentage points**, two fifths of the apparent effect, was age and mileage. Damaged cars average 15.0 years old against 12.3 for undamaged, and 77% sit on the 150,000 km cap against 60%.

44% is still large, and it should be: a buyer is accepting an unknown repair bill, a probable failed roadworthiness test, and a car that is harder to resell. But it is a price rather than an artifact.

**At the bottom of the market the damage flag stops carrying information.** Several matched cells show a discount near zero or slightly negative. A 16 to 20 year old Nissan Primera at the mileage cap fetches about EUR 920 undamaged and EUR 970 damaged. Once a car is worth less than the repair, buyers pay scrap-adjacent prices either way.

### 2. One thousand physically impossible listings

A first pass by age band returned a **94.2% discount for cars aged 0 to 3**, which is not a market price. The 200 listings behind it were scrap: **152 claimed a 2016 registration and 131 sat at the 150,000 km cap**, in data crawled in March 2016.

Their own titles say so: *Bastlerfahrzeug* (project car), *zum Ausschlachten* (for stripping), *springt nicht an* (will not start). One offers a "Golf 2", a model built between 1983 and 1992, with a registration year of 2016.

Sellers who left the registration year blank appear to have been defaulted to 2016, which the data then reads as brand new.

The filter needs no judgement about listing text: **a car registered in 2016 and crawled in March 2016 cannot have driven 125,000 km.** That catches 1,000 listings with a median price of EUR 999, and removing them moves the 0-3 band from 94.2% to a plausible 58.9%.

### 3. Other data quality findings

- **`seller`, `offerType` and `nrOfPictures` carry no information.** 49,999 of 50,000 rows are `privat` and `Angebot`; every row has zero pictures. Scraping artifacts, not variables.
- **Prices include 1,421 zeros and values to 99,999,999,** among them 11111111 and 12345678.
- **Registration years run 1000 to 9999,** with 1,966 listings registered after the crawl date.
- **Odometer is banded into 13 values, top-coded at 150,000 km,** where 65% of listings sit, so mileage cannot be controlled finely at the top.
- **19.7% of damage fields are blank,** and those listings resemble damaged cars on age (15.0 vs 15.0 years) and mileage cap rate (75.4% vs 76.9%), not a random sample. The blank rate rises to 48% among the impossible listings.

## Method

1. Parse price and odometer out of their string formats.
2. Keep listings between EUR 100 and 150,000 with a registration year between 1950 and 2016.
3. Drop the physically impossible 2016-plus-125,000 km combination.
4. Group into cells of brand x model x age band x odometer reading, requiring at least five listings on each side.
5. Take the median price on each side of each cell and weight cell discounts by the number of damaged listings.

## What I would not claim

- **These are asking prices, not sale prices.** Nothing shows what any car sold for, or whether it sold.
- **Matching is not causal identification.** Cells do not match on condition, service history, trim, engine, or severity. "Unrepaired damage" covers a scratched bumper and a bent chassis alike.
- **The matched sample is not the whole market.** The five-per-side rule keeps 176 cells covering 2,456 damaged cars, roughly half, weighted toward common mass-market models.
- **Excluding blank damage fields is a choice.** The sensitivity check brackets the naive discount between 67.8% and 71.4% depending on treatment, so it is not load-bearing, but it is a choice.
- **The impossibility filter may catch some genuine listings,** such as a car imported and registered in Germany in 2016 after years abroad. At 1,000 of 46,315 the cost of over-filtering is small against the distortion of leaving them in.

## Files

- `notebook.ipynb`: full analysis, executed, with two charts.

## Stack

pandas, NumPy, Matplotlib (fivethirtyeight style). No seaborn.
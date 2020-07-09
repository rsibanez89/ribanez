---
title: "Starting with Google sheets"
date: 2020-07-08T18:46:24+11:00
---

## Introduction
Google sheet is a powerful tool to analyze the Market and Forex historical data. It provides the `GOOGLEFINANCE` function that allows you to see the values directly in the sheet without you having to import any data.
For example, if we put the following formula in one of the google sheet cells we can see the close price for `GBPUSD` in the last 30 days.
```csharp
=GOOGLEFINANCE("CURRENCY:GBPUSD", "close", TODAY()-30, TODAY())
```
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vRmlIRmRTMa2vEQkwDCz2Ce4icyukrQL3NN4_cxC7n_zuBw33-5XrndyUp63UL-jq-SMsKknhwKIEhL/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="260px" >}}

As you can see, that command returns a table but sometimes we want to get just the price value for an especific date. In this case we need to use the the `INDEX` function with the coordinates we want to get on top of the previous table. The coordinates are specified as row first and then colum.

```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:GBPUSD", "close", TODAY()-30, TODAY()), 5, 2)
```
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vTob0lze1Ny9q7LtkjqBKiXwBic-MErbm7ULICPCcxLP0r7gLO4zA9z68-LTEcYXFa6gwY32yjZikkK/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="260px" >}}

Full documentation can be found {{< blink "src"="https://support.google.com/docs/answer/3093281" "text"="HERE" >}}.

## Customization
You can create a dropdown to select the symbol you want to get. To do this, we need to go to `Data` tab and select `Data validation`.
Then in the floating windows, we need to select `Criteria: List of items` and finally input the list of items we want. In my case `GBPUSD, AUDUSD, EURUSD, USDJPY`.
![Create symbols dropdown](/trading/spreadsheet-symbols-dropdown.png)

Once we have the dropdown we need to change the formula to point to this cell `B1`. As we need to concatenate the word `CURRENCY:` and the content of `B1` we need to use `&`.
```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:"&B1, "close", TODAY()-30, TODAY()), 5, 2)
```
With that formula, we are already pointing to `B1` but in order to fix the value so we can drag the formula to other cells we need to add `$` in front of the column coordinate and the also in front of the row coordinate. The final formula will look like this:
```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:"&$B$1, "close", TODAY()-30, TODAY()), 5, 2)
```
We can also repeat this process to change the close price for the hight or low.

## Monthly value
In this example we are going to generate a matrix with the prices at the beginning of the month from 2006 to 2020.
To do this I've created an empty matrix with the years and months we want to query. These will work as indexes for our formula.
![Create empty matrix](/trading/spreadsheet-empty-matrix.png)

And now, the formula for January 2006 (B6) will look like this:
```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:"&$B$1, $B$2, DATE($A6, B$4, 1), DATE($A6, B$4, 15)), 2, 2)
```
Let's analyze it:
  * `"CURRENCY:"&$B$1`: we already talked about concatenate using `&` and fix using `$`. So this will point always to B1.
  * `$B$2`: same as the previous one this will point always to B2.
  * `DATE($A6, B$4, 1)`: this one needs a bit of explanation. `DATE(2018, 12, 31)` means the date 31st December 2018.
    * `$A6` means that column A is fixed, and the row 6 is variable. So when I drag it, it will generate values like: `A7, A8, A9, A10...`.
    * `B$4` means that column B is variable, and the row 4 is fixed. So when I drag it, it will generate values like: `C4, C5, C6, C7...`.
  * `DATE($A6, B$4, 15)`: same as the previous one. I used 15 instead of 30 to make it faster. DO NOT USE 1 because sometimes the market is closed on the 1st of the month. And you will get no data.
  * Finally, the `2, 2` means the coordinate for the `INDEX` function. In this case, it will return the valuation for the first day that the market is open.

Finally, dragging this cell to the other ones.
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vRIURRrYnZ7NK6_yS5bHMWitNAcUXyhftsU5vqa-KrGtzLWX2uTU6deRM1d54UkzjA2B8Yi6XWqEssA/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="560px" >}}

## Bonus analysis - price variation month by month
Let's create a new matrix, by using the one that we have and compute the difference in between the current month and the previous month.
Then, let's add a conditional formating for prices above 0 in green. To do this, first we select the complete matrix values. Then, we need to go to `Format` tab and select `Conditional formating`. In the rules, we need pick `Custom formula is` and then complete the field with `=B6>0`. As we saw before, column B is variable and row 6 is variable, so the format will apply for all the cells in the selection.
![Create empty matrix](/trading/spreadsheet-conditional-formating.png)
Finally, let's add a second conditional formating for prices below 0. 

This is the result:
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vTnIO6Yng6q7Yu7_NbzfEBAfjDs_FF9zVqZqrYDw1cFugpLMugghdqALE67k3d6_bGhbuLNO1GrZ32b/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="560px" >}}

## Conclusion
*From this table, we can see that May is a really good month to sell `GBPUSD`.*
The prices the fist day of May are lower than the prices the fist of June for most of the years in between 2006 and 2020. That is why we can see a red column.
---
title: "Starting with Google sheets"
date: 2020-07-08T18:46:24+11:00
---

## Introduction
Google sheet is a powerful tool to analyze the Stock Market and Forex historical data. The `GOOGLEFINANCE` function allows you to see the values directly in the sheet without you having to import any data.
For example, if we put the following formula in one of the Google Sheet cells we can see the close price for `GBPUSD` in the last 30 days.
```csharp
=GOOGLEFINANCE("CURRENCY:GBPUSD", "close", TODAY()-30, TODAY())
```
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vRmlIRmRTMa2vEQkwDCz2Ce4icyukrQL3NN4_cxC7n_zuBw33-5XrndyUp63UL-jq-SMsKknhwKIEhL/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="260px" >}}

To see other parameters that can be used in the `GOOGLEFINANCE` function go [{{< blink "src"="https://support.google.com/docs/answer/3093281" "text"="HERE" >}}].

As you can see, the `GOOGLEFINANCE` function returns a table but sometimes we want to get just the price value for an specific date. To do that, we need to use the `INDEX` function on top of the previous table. The `INDEX` function requires you to specify the coordinates for the row and column you want to select.
For example, to get the value in the row 5 and column 2, we will use this formula:

```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:GBPUSD", "close", TODAY()-30, TODAY()), 5, 2)
```
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vTob0lze1Ny9q7LtkjqBKiXwBic-MErbm7ULICPCcxLP0r7gLO4zA9z68-LTEcYXFa6gwY32yjZikkK/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="260px" >}}

## Customization
In order to make the formula generic for multiple symbols, we can create a dropdown. To do this, we need to go to the `Data` tab and select `Data validation`.
Then in the floating windows, we need to select `Criteria: List of items` and finally input the list of items we want in the dropdown. In my case `GBPUSD, AUDUSD, EURUSD, USDJPY`.
![Create symbols dropdown](/trading/spreadsheet-symbols-dropdown.png)

Once we have the dropdown, we need to change the formula to point to the cell `B1`, which is the one containing the symbol. As we need to concatenate the word `CURRENCY:` and the content of `B1` we need to use `&`.
```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:"&B1, "close", TODAY()-30, TODAY()), 5, 2)
```
With that formula, we are already pointing to `B1` but if we drag the formula to other cells we can see that `B1` changes. For example, if we drag the formula one cell to the right the formula will point to `C1`. So to solve this problem, we need to `fix` the cell we are pointing by adding `$` in front of the column coordinate and the row coordinate. The final formula will look like this:
```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:"&$B$1, "close", TODAY()-30, TODAY()), 5, 2)
```
We can also repeat this process to change the `close` price and use the `high` or `low` prices.

## Monthly values
In this example, we are going to generate a matrix with the prices at the beginning of the month from 2006 to 2020.
To do this, I've created an empty matrix with the years and months we want to query. These will work as indexes for our formula.
![Create empty matrix](/trading/spreadsheet-empty-matrix.png)

And now, the formula for January 2006 (`B6`) will look like this:
```csharp
=INDEX(GOOGLEFINANCE("CURRENCY:"&$B$1, $B$2, DATE($A6, B$4, 1), DATE($A6, B$4, 15)), 2, 2)
```
Let's analyze it:
  * `"CURRENCY:"&$B$1`: we already talked about concatenating using `&` and fixing using `$`. So this will point always to `B1`.
  * `$B$2`: same as the previous one this will point always to `B2`.
  * `DATE($A6, B$4, 1)`: this one needs a bit of explanation. `DATE(2018, 12, 31)` means the date 31st December 2018.
    * `$A6` means that column A is fixed, and row 6 is variable. So when I drag it, it will generate values like: `A7, A8, A9, A10...`.
    * `B$4` means that column B is variable, and row 4 is fixed. So when I drag it, it will generate values like: `C4, D4, E4, F4...`.
  * `DATE($A6, B$4, 15)`: same as the previous one. I used 15 instead of 30 to make it faster. *DO NOT USE 1* because sometimes the market is closed on the 1st day of the month and you will get no data.
  * Finally, the `2, 2` means the coordinate for the `INDEX` function. In this case, it will return the valuation for the first day that the market is open.

Finally, dragging this cell to the other ones.
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vRIURRrYnZ7NK6_yS5bHMWitNAcUXyhftsU5vqa-KrGtzLWX2uTU6deRM1d54UkzjA2B8Yi6XWqEssA/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="560px" >}}

## Price variation month by month
Let's create a new matrix by using the one that we have and compute the difference between the current month and the previous month.
Then, let's add a conditional formating for prices above 0 in green. To do this, first we select the whole matrix. Then, we go to the `Format` tab and select `Conditional formating`. In the rules, we need to pick `Custom formula is` and then complete the field with `=B6>0`. As we saw before, column B is variable and row 6 is variable, so the format will apply for all the cells in the selection.
![Create empty matrix](/trading/spreadsheet-conditional-formating.png)
Repeating the process, let's add a second conditional formating for prices below 0. The conditional formula will be `=B6<0`.

The final result will be this colored matrix:
{{< gdocs "src"="https://docs.google.com/spreadsheets/d/e/2PACX-1vTnIO6Yng6q7Yu7_NbzfEBAfjDs_FF9zVqZqrYDw1cFugpLMugghdqALE67k3d6_bGhbuLNO1GrZ32b/pubhtml?gid=0&amp;single=true&amp;widget=true&amp;headers=false" "height"="560px" >}}

## Conclusion
*From this table, we can see that May is a really good month to sell `GBPUSD`.*
The prices on the first day of May are higher than the prices on the first day of June for most of the years between 2006 and 2020. That is why we can see a red column for May.
Similarly, but not as strong, we can see a green column for April, suggesting that April is a good month for buying `GBPUSD`.

## Bonus
Check the final result [{{< blink "src"="https://docs.google.com/spreadsheets/d/1wWbieU7KP8DMfkEiw5lcEDWC0wR1rGTxvzcshLEluHQ/edit?usp=sharing" "text"="HERE" >}}]
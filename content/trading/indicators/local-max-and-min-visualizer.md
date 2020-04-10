---
title: "My fist indicator: Local max and min points in the chart"
date: 2020-04-10T14:13:24+11:00
draft: false
---

The purpose of this indicator is just for me to undestand how to draw indicators in MetaTrader4 and it has no technical meaning.
This indicator shows with blue and red dots the most recent Hights and Lows in the chart considering the last X candles.
And it is inspired on this [tutorial](https://www.youtube.com/watch?v=2inA41MmjTQ&t).

![Local max and min](/trading/indicators-max-min.png)

## Code
```csharp
//+------------------------------------------------------------------+
//|                                               LocalMaxAndMin.mq4 |
//|                                                   Rodrigo Ibanez |
//|                                             https://www.mql5.com |
//+------------------------------------------------------------------+
#property copyright "Rodrigo Ibanez"
#property link      "https://www.mql5.com"
#property version   "1.00"
#property strict
#property indicator_chart_window
#property indicator_buffers 2
#property indicator_color1  clrBlue
#property indicator_color2  clrRed
#property indicator_width1  2
#property indicator_width2  2

static int UpId = 0;
double Ups[];

static int DownId = 1;
double Downs[];

input int LookBackCandles = 55;

//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
{
   IndicatorShortName("Local max and min");
   
   SetIndexBuffer(UpId, Ups);
   SetIndexStyle(UpId, DRAW_ARROW);
   SetIndexArrow(UpId, 159);
   
   SetIndexBuffer(DownId, Downs);
   SetIndexStyle(DownId, DRAW_ARROW);
   SetIndexArrow(DownId, 159);

   return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
{
   for(int i = rates_total - 1; i > 0; i--)
   {
      // Blue dots
      int highestId = iHighest(Symbol(), Period(), MODE_HIGH, LookBackCandles, i);
      
      if (i == highestId)
      {
         Ups[i] = high[i];
      }
      else
      {
         Ups[i] = EMPTY_VALUE;
      }
      
      // Red dots
      int lowestId = iLowest(Symbol(), Period(), MODE_LOW, LookBackCandles, i);
      
      if (i == lowestId)
      {
         Downs[i] = low[i];
      }
      else
      {
         Downs[i] = EMPTY_VALUE;
      }
   }   
   
   return(rates_total);
}
```
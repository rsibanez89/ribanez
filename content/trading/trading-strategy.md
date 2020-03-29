---
title: "My fist trading strategy: the crossing moving averages"
date: 2020-03-29T15:13:24+11:00
draft: false
---

{{< hint danger >}}
**Do not use this strategy on a real account**  
This strategy has been created just for testing purposes.
{{< /hint >}}

# Crossing moving averages strategy
The following strategy consists in defining 2 moving averages and buy or sell when they cross over each other.
To do this, we have defined a fast-moving average that considers the last 40 candles (green), and a slow-moving average that considers the last 80 candles (red).
Then, if the fast moving average crosses down the slow moving average, we consider that the market starts being bearish and it is a good opportunity to sell.
On the other hand, if the fast moving average crosses up the slow moving average, we consider that the market starts being bullish and it is a good opportunity to sell.

![Crossing moving averages](/trading/crossing-moving-averages.png)

## Parameters tested
```
Currency pair: GBPUSD
Period: 15 minutes
From: 2020.01.15 12:45
To: 2020.03.26 14:13
Fast-moving average: 40 candles
Slow-moving average: 80 candles
Stop loss: 70 pips, distance to the price
Take profit: 120 pips, distance to the price
Risk to reward: 1.7%
```

## Results
After some tests I got the best results by using the following parameters on a 1000 dollar account:

| Lot | SL |  TP | %Risk | Maximum money risked by trade |  Maximum money win by trade  | Total profit |
|:---:|:--:|:---:|:-----:|:-----------------------------:|:----------------------------:|:------------:|
| 0.1 | 70 | 120 |  1.7  |   73.6 (stop loss + spread)   | 116.4 (take profit - spread) |    1175.22   |

As a conclusion, from 2020.01.15 to 2020.03.26 (3 months) this strategy would have generated a profit of 175% leaving the account with a total balance of 2175.22.

Report:
![Crossing moving averages](/trading/crossing-moving-averages-report.png)

Graph:
![Crossing moving averages](/trading/crossing-moving-averages-graph.png)

## Code
```csharp
//+------------------------------------------------------------------+
//| DO NOT USE THIS STRATEGY ON A REAL ACCOUNT                       |
//| This strategy has been created just for testing purposes.        |
//|                                                                  |
//|                               CrossingMovingAveragesStrategy.mq4 |
//|                                                   Rodrigo Ibanez |
//|                                       https://www.ribanez.com.ar |
//+------------------------------------------------------------------+
#property copyright "Rodrigo Ibanez"
#property link      "https://www.ribanez.com.ar"
#property version   "1.00"
#property strict

extern int FastMovingAverage = 40;
extern int SlowMovingAverage = 80;

extern double Lot = 0.1;
extern int TakeProfit = 120;
extern int StopLoss = 70;
static int TicketOrder = 0;
static datetime LastOrderOpenedAt = "1930-01-01";

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
{

  return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
{

   if(Bars < 100 || IsTradeAllowed() == false)
   {
      return;
   }
   
   double difference = (TimeCurrent() - LastOrderOpenedAt) / 3600.0;
   if(difference < 1) // Already oppened a trade during this hour, wait at least 1 hour to open the next trade.
   {
      return;
   }
   
  
   double slowMovingAverage = iMA(Symbol(), Period(), SlowMovingAverage, 0, MODE_SMA, PRICE_CLOSE, 0);
   double previousSlowMovingAverage = iMA(Symbol(), Period(), SlowMovingAverage, 0, MODE_SMA, PRICE_CLOSE, 1);
   
   double fastMovingAverage = iMA(Symbol(), Period(), FastMovingAverage, 0, MODE_SMA, PRICE_CLOSE, 0);
   double previousFastMovingAverage = iMA(Symbol(), Period(), FastMovingAverage, 0, MODE_SMA, PRICE_CLOSE, 1);
   
   int result;
      
   // Fast moving average crosses up the slow moving average
   if(previousFastMovingAverage < previousSlowMovingAverage && fastMovingAverage > slowMovingAverage)
   {
      // Check if there is already an open order:
      if(OrderSelect(TicketOrder, SELECT_BY_TICKET) && OrderCloseTime() == 0)
      {
         OrderClose(TicketOrder, Lot, OrderClosePrice(), 10 * 10);
      }
   
      double stopLoss = Bid - StopLoss * Point * 10; // * 10 because the broker has 5 decimal points
      double takeProfit = Bid + TakeProfit * Point * 10; // * 10 because the broker has 5 decimal points
      // Print("OP_BUY: SL" + stopLoss + ", Price:" + Bid + ", TP:" + takeProfit);
      TicketOrder = OrderSend(Symbol(), OP_BUY, Lot, Ask, 0, stopLoss, takeProfit, "Buy - Fast MA crossed up Slow MA");
      LastOrderOpenedAt = TimeCurrent();
      return;
   }
   
   // Fast moving average crosses down the slow moving average
   if(previousFastMovingAverage > previousSlowMovingAverage && fastMovingAverage < slowMovingAverage)
   {
      // Check if there is already an open order:
      if(OrderSelect(TicketOrder, SELECT_BY_TICKET) && OrderCloseTime() == 0)
      {
         OrderClose(TicketOrder, Lot, OrderClosePrice(), 10 * 10);
      }

      double stopLoss = Ask + StopLoss * Point * 10; // * 10 because the broker has 5 decimal points
      double takeProfit = Ask - TakeProfit * Point * 10; // * 10 because the broker has 5 decimal points
      // Print("OP_SELL: SL" + stopLoss + ", Price:" + Bid + ", TP:" + takeProfit);
      TicketOrder = OrderSend(Symbol(), OP_SELL, Lot, Bid, 0, stopLoss, takeProfit, "Buy - Fast MA crossed down Slow MA");
      LastOrderOpenedAt = TimeCurrent();
      return;
   }
 }
```
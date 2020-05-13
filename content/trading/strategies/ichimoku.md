---
title: "Ichimoku"
date: 2020-04-26T18:13:24+11:00
draft: false
---

{{< hint danger >}}
**Do not use this strategy on a real account**  
This strategy has been created just for testing purposes.
{{< /hint >}}

# Ichimoku strategy
The following strategy consists in using the Ichimoku indicator to forecast the future price movement.
To do this, we have defined the following rules based on the indicator:
To open a buy position:
1. The price is above the cloud with a strong up movement.
2. The price has just crossed the cloud.
3. The Kijun-sen shows an up movement in the last 4 candles.
4. The price (26 candles in the past) is below the Chikou Span.
5. The risk to reward ratio is viable.

From 1 to 4, the rules are strict and easy to get from the indicator. The fifth rule is calculated in the following way:
* We will position the stop loss below the cloud with a small margin.
* We have defined a risk to reward ratio of 2.4%, which means that the take profit is 2.4 bigger than the stop loss. In this way, we know where to put our take profits and stop loss based on the current price.
* We get the range in which the price has moved in the last 75 candles. If the range is bigger than our take profit it means that the trade viable and we will open the buy position.

On the other hand, to open a sell position we check the following rules:
1. The price is below the cloud with a strong down movement.
2. The price has just crossed the cloud.
3. The Kijun-sen shows a down movement in the last 4 candles.
4. The price (26 candles in the past) is above the Chikou Span.
5. The risk to reward ratio is viable.

![Ichimoku](/trading/ichimoku.png)

## Parameters used for testing
After some tests I got the best results by using the following parameters on a 1000 dollar account:
```
Currency pair: GBPUSD
Period: 15 minutes
From: 2019.01.01
To: 2019.12.31
Lot: 0.1
LookBackSwingSize: 75 candles
LookBackKijunsen: 4 candles
Risk to reward: 2.4%
```
## Results
See the full report [HERE](../ichimoku-strategy-report.html)

Report:
![Ichimoku](/trading/ichimoku-report.png)

Return graph:
![Ichimoku](/trading/ichimoku-graph.png)

Price during the same period:
![Ichimoku](/trading/GBPUSDDaily-2019.png)

We can see from the graphs that the strategy started generating real profit after 2020.02.20, which is when the market started experiencing higher volatility.
Observing both graps (return and price) during the same period we can see that this strategy performs better when there is a strong trend going down or up. And on the contrary, when the price moves sideways for example in the period from January 16th to April 25th this strategy does not show good results.
As a conclusion, from 2019.01.01 to 2019.12.31 (1 year) this strategy would have generated a profit of 80.91% leaving the account with a total balance of 1809.14.

## Code
```csharp
//+------------------------------------------------------------------+
//| DO NOT USE THIS STRATEGY ON A REAL ACCOUNT                       |
//| This strategy has been created just for testing purposes.        |
//|                                                                  |
//|                                             IchimokuStrategy.mq4 |
//|                                                   Rodrigo Ibanez |
//|                                       https://www.ribanez.com.ar |
//+------------------------------------------------------------------+
#property copyright "Rodrigo Ibanez"
#property link      "https://www.ribanez.com.ar"
#property version   "1.00"
#property strict

extern double Lot = 0.1;
extern double RiskRewardRatio = 2.4;
extern int LookBackSwingSize = 75;
extern int LookBackKijunsen = 4;

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

   if(IsTradeAllowed() == false)
   {
      return;
   }
   
   double difference = (TimeCurrent() - LastOrderOpenedAt) / 3600.0;
   if(difference < 0.5 && OrdersTotal() > 0) // Already oppened a trade during this hour, wait at least 1/2 an hour to open the next trade.
   {
      return;
   }
   
   CheckForBuySetup();
   CheckForSellSetup();
}

//+------------------------------------------------------------------+
//| Returns:                                                         |
//|  -1  When not enough information                                 |
//|  size of the last swing                                          |
//+------------------------------------------------------------------+
double GetSwingSize()
{
   // Number of candles to consider   
   if(iBars(Symbol(), Period()) < LookBackSwingSize) 
   {
      return -1;
   }
   
   int highestIndex = iHighest(NULL, 0, MODE_HIGH, LookBackSwingSize, 0);
   int lowestIndex  = iLowest(NULL, 0, MODE_LOW, LookBackSwingSize, 0);
   
   double highest = iHigh(Symbol(), Period(), highestIndex);
   double lowest  = iLow(Symbol(), Period(), lowestIndex);
   
   // Print("Highest:", highest);
   // Print("Lowest:", lowest);
   
   return highest - lowest;
}

//+------------------------------------------------------------------+
//| Returns:                                                         |
//|  true   if there is a buy setup and the order was opened         |
//|  false  if there is not a buy setup or the order wasn't opened   |
//+------------------------------------------------------------------+
bool CheckForBuySetup()
{
   double chikouspan  = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_CHIKOUSPAN, 26);
   double tenkansen   = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_TENKANSEN, 0);
   double kijunsen    = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_KIJUNSEN, 0);
   double senkouspana = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_SENKOUSPANA, 0);
   double senkouspanb = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_SENKOUSPANB, 0);
   
   
   double price    = iLow(Symbol(), Period(), 0);
   double price_1  = iOpen(Symbol(), Period(), 1);
   double price_26 = iHigh(Symbol(), Period(), 26);
   
   // Conditions for Ichimoku: Up trend
   // 1- Price above the cloud with a strong up movement
   if(!(price > MathMax(senkouspana, senkouspanb) + iATR(Symbol(), 0, 20, 0) * 0.5))
   {
      return false;
   }
   
   // 2- Price just crossed the cloud
   if(!(price_1 < MathMax(senkouspana, senkouspanb)))
   {
      return false;
   }
   
   // 3- Up movement of the Kijunsen
   bool upMovement = false;   
   for(int i = 0; i < LookBackKijunsen; i++)
   {
      if(iIchimoku(Symbol(), 0, 9, 26, 52, MODE_KIJUNSEN, i) > iIchimoku(Symbol(), 0, 9, 26, 52, MODE_KIJUNSEN, i + 1))
      {
         upMovement = true;
         break;
      }
   }
   if(!(upMovement))
   {
      return false;
   }
   
   // 4- Price is below the chikouspan
   if(!(price_26 < chikouspan))
   {
      return false;
   }
   
   // --  Good buy setup --  
   double stopLoss = MathMin(senkouspana, senkouspanb) - iATR(Symbol(), 0, 20, 0) * 0.5;
   double distanceToStopLoss = Bid - stopLoss;
   double distanceToTakeProfit = RiskRewardRatio * distanceToStopLoss;
   double takeProfit = Bid + distanceToTakeProfit;
   
   // -- Check risk to reward ratio
   double lastSignificantMovement = GetSwingSize();
   if(lastSignificantMovement < distanceToTakeProfit)
   {
      return false;
   }
   
   // Check if there is already an open order:
   if(OrderSelect(TicketOrder, SELECT_BY_TICKET) && OrderCloseTime() == 0)
   {
      if(OrderType() == OP_SELL) 
      {
         if(OrderClose(TicketOrder, Lot, OrderClosePrice(), 0))
         {
            Alert("Error Closing OP_SELL");
         }
         TicketOrder = 0;
      }
      // else
      // {
      //    TicketOrder = OrderModify(OrderTicket(), OrderOpenPrice(), stopLoss, takeProfit, 0, Blue);
      //    LastOrderOpenedAt = TimeCurrent();
      //    return true;
      // }
   }

   // Print("OP_BUY: SL" + stopLoss + ", Price:" + Bid + ", TP:" + takeProfit);
   TicketOrder = OrderSend(Symbol(), OP_BUY, Lot, Ask, 0, stopLoss, takeProfit, "Buy setup!");
   if(TicketOrder < 0)
   {
      Alert("Error Seding OP_BUY, SL: ", stopLoss, " - Price: ", Ask, " - TP: ", takeProfit);
      return false;
   }
   LastOrderOpenedAt = TimeCurrent();
   
   return true;
}


//+------------------------------------------------------------------+
//| Returns:                                                         |
//|  true   if there is a sell setup and the order was opened        |
//|  false  if there is not a sell setup or the order wasn't opened  |
//+------------------------------------------------------------------+
bool CheckForSellSetup()
{
   double chikouspan  = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_CHIKOUSPAN, 26);
   double tenkansen   = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_TENKANSEN, 0);
   double kijunsen    = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_KIJUNSEN, 0);
   double senkouspana = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_SENKOUSPANA, 0);
   double senkouspanb = iIchimoku(Symbol(), 0, 9, 26, 52, MODE_SENKOUSPANB, 0);
   
   
   double price    = iHigh(Symbol(), Period(), 0);
   double price_1  = iOpen(Symbol(), Period(), 1);
   double price_26 = iLow(Symbol(), Period(), 26);
   
   // Conditions for Ichimoku: Down trend
   // 1- Price below the cloud with a strong movement
   if(!(price < MathMin(senkouspana, senkouspanb) - iATR(Symbol(), 0, 20, 0) * 0.5))
   {
      return false;
   }
   
   // 2- Price just crossed the cloud
   if(!(price_1 > MathMin(senkouspana, senkouspanb)))
   {
      return false;
   }
   
   // 3- Down movement of the Kijunsen
   bool downMovement = false;   
   for(int i = 0; i < LookBackKijunsen; i++)
   {
      if(iIchimoku(Symbol(), 0, 9, 26, 52, MODE_KIJUNSEN, i) < iIchimoku(Symbol(), 0, 9, 26, 52, MODE_KIJUNSEN, i + 1))
      {
         downMovement = true;
         break;
      }
   }
   if(!(downMovement))
   {
      return false;
   }
   
   // 4- Price is above the chikouspan
   if(!(price_26 > chikouspan))
   {
      return false;
   }
   
   // --  Good sell setup --  
   double stopLoss = MathMax(senkouspana, senkouspanb) + iATR(Symbol(), 0, 20, 0) * 0.5;
   double distanceToStopLoss = stopLoss - Ask;
   double distanceToTakeProfit = RiskRewardRatio * distanceToStopLoss;
   double takeProfit = Ask - distanceToTakeProfit;

   // -- Check risk to reward ratio
   double lastSignificantMovement = GetSwingSize();
   if(lastSignificantMovement < distanceToTakeProfit)
   {
      return false;
   }
   
   // Check if there is already an open order:
   if(OrderSelect(TicketOrder, SELECT_BY_TICKET) && OrderCloseTime() == 0)
   {
      if(OrderType() == OP_BUY) 
      {
         if(OrderClose(TicketOrder, Lot, OrderClosePrice(), 0))
         {
            Alert("Error Closing OP_BUY");
         }
         TicketOrder = 0;
      }
      // else
      // {
      //    TicketOrder = OrderModify(OrderTicket(), OrderOpenPrice(), stopLoss, takeProfit, 0, Red);
      //    LastOrderOpenedAt = TimeCurrent();
      //    return true;
      // }
   }

   // Print("OP_SELL: SL" + stopLoss + ", Price:" + Bid + ", TP:" + takeProfit);
   TicketOrder = OrderSend(Symbol(), OP_SELL, Lot, Bid, 0, stopLoss, takeProfit, "Sell setup!");
   if(TicketOrder < 0)
   {
      Alert("Error Seding OP_SELL, SL: ", stopLoss, " - Price: ", Ask, " - TP: ", takeProfit);
      return false;
   }
   LastOrderOpenedAt = TimeCurrent();
   
   return true;
}
```
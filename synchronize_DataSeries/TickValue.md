### Tick Value
Tick Value refers to the monetary worth of a single tick, representing the smallest possible price movement allowed in a particular market. It is a crucial concept for traders as it quantifies the financial impact of price changes on their positions.
To illustrate, let's consider the E-Mini S&P 500 futures market, where the tick size is 0.25. This means that the price of the futures contract can move in increments of 0.25 points. Now, if the tick value is $12.50, it implies that for every 0.25-point movement in the price of the futures contract, the profit or loss for a trade would change by $12.50.
For instance, suppose a trader buys a single E-Mini S&P 500 futures contract at a price of 3000. If the price increases by one tick to 3000.25, the trader's profit would be $12.50 (assuming a long position). Conversely, if the price decreases by one tick to 2999.75, the trader's loss would also be $12.50.
Understanding the tick value is essential for traders to assess the potential risk and reward of their trades accurately. It allows them to calculate the profit or loss for each price movement and determine the appropriate position size based on their risk tolerance and trading strategy.

Blitz Strategy API provides Event callback OnMarketDataEvent method in your strategy that developers need to override to get notification of change in market data information for subscribed instrument variable

This code snippet demonstrates how to handle various aspects of market data within the OnMarketDataEvent method, including retrieving bid/ask prices, checking for changes in the last trade, calculating VWAP (Volume Weighted Average Price), accessing the best quoted price, and retrieving instrument information.

```
private IVObject _iv = new IVObject( "Instrument",
                                                                    "Tradable Instrument",
                                                                     true,
                                                                     InstrumentType.Futures | InstrumentType.Options,
                                                                     MarketDataType.All,
                                                              OrderEventType.All);

```
```
private IOrderCommand _entryOrderCommand = null;
…….
…….

protected override void OnMarketDataEvent(StrategyMarketDataEventArgs strategyMarketDataEventArgs)
{
    if (strategyMarketDataEventArgs.IVObject == _iv)
    {
        ITouchLineInfo touchLineInfo = strategyMarketDataEventArgs.MarketDataContainerInfo.TouchLineInfo;
        double bestBidPrice = touchLineInfo.BestBidPrice;
        int bestBidSize = touchLineInfo.BestBidSize;
        double bestAskPrice = touchLineInfo.BestAskPrice;
        int bestAskSize = touchLineInfo.BestAskSize;
        double lastPrice = touchLineInfo.LastPrice;
        int lastSize = touchLineInfo.LastSize;
        long lastTradedTime = touchLineInfo.LastTradedTime;
    }
}

```
```
// Check if touchline represents a change in any attribute of the last trade
bool isLastPriceChange = touchLineInfo.IsLastPriceChange;

// Access the previous last price
double lastPrice = touchLineInfo.PreviousLastPrice;

// IMarketDepthInfo interface provides depth level price and size information
IMarketDepthInfo marketDepthInfo = strategyMarketDataEventArgs.MarketDataContainerInfo.MarketDepthInfo;

double totalSumBidPrice = 0;
int totalBidSize = 0;

// Calculating BuySide VWAP price till depth level 5
for (int i = 0; i < marketDepthInfo.MarketDepthLevel; i++)
{
    totalSumBidPrice += marketDepthInfo.GetBidPriceAt(i);
    totalBidSize += marketDepthInfo.GetBidSizeAt(i);
    
    // Break loop conditions
    if (i >= 5 || marketDepthInfo.GetBidSizeAt(i) == 0 || marketDepthInfo.GetBidPriceAt(i) <= 0)
        break;
}

// Calculate VWAP Bid price till 5th depth level
double vwapBidPriceTill5Level = totalSumBidPrice / totalBidSize;
```

```
// Use smart order to access a best quoted price
if (_entryOrderCommand.CurrentOrder == null)
    entryOrderPrice = base.GetBestPrice(_ivInfo.MarketDataContainer, entrySide);
else
    entryOrderPrice = base.GetBestPrice(_ivInfo.MarketDataContainer, _entryOrderCommand.CurrentOrder);

// Access Instrument information
IMarketDepthInfo marketDepthInfo = strategyMarketDataEventArgs.MarketDataContainerInfo.MarketDepthInfo;
QX.Base.Common.InstrumentInfo.Instrument instrument = strategyMarketDataEventArgs.IVInstrument.Instrument;

if (instrument.InstrumentType == InstrumentType.Options)
{
    Options optionInstrument = (Options)instrument;
    double strikePrice = optionInstrument.StrikePrice;
    DateTime contractExpiration = optionInstrument.ContractExpiration;
}
```

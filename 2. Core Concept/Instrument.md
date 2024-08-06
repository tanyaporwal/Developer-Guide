## Instrument
An instrument serves as a fundamental component of any trading system, containing essential information about tradable securities within a given exchange segment. 

The BlitzTrader API provides an abstract Instrument class that represents exchange-traded contracts, including Equities, Futures, Options, Spreads, and Spot (Forex). This class encapsulates all common properties defining the contract, such as name, exchange ID, exchange segment, tick size, lot size, etc. Additional properties specific to certain instrument types, such as expiration dates for derivative contracts (Futures, Options), strike prices, option types (CE, PE, CA, PA) for Options contracts, are embedded into concrete instrument type classes.

Custom properties of an Instrument can be added through an Extended Market Properties mechanism, facilitating the insertion of key-value pairs. BlitzTrader's framework establishes a process for importing tradable instrument definitions and persisting them internally.
The BlitzTrader Adapter, a programmable component representing the exchange segment, provides access to the market for both trading and market data interfaces. It is the responsibility of the Adapter to furnish the BlitzTrader framework with the list of tradable instruments.

| Name | Security Type | Exchange Segment | TickSize | LotSize | Expiry | Strike Price | Option Type |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ESM15 | FUTURES | CME | 0.25 | 1 | June 2015 | - | - |
| GOOG | EQUITY | NASDAQ | 0.01 | 1 | - | - | - | 
| NIFTY | FUTURES | NSEFO | 0.05 | 50 | 26Feb2015 | - | - |
| SBIN | OPTIONS | NSEFO | 0.05 | 500 | 26Feb2015 | 295 | CE |
| RELIANCE | EQUITY | NSECM | 0.05 | 300 | - | - | - |
| USD/INR | FUTURES | NSECD | 0.0025 | 1000 | 26 Feb2015 | - | - |
| EUR/USD | FOREX | FXCM | 0.0001 | 1000 | - | - | - |

For further details, please refer to the Adapter section on how to import Instrument definitions into BlitzTrader. BlitzTrader maintains a unique name for each instrument per exchange segment based on the following nomenclature.

| Instrument Types | Symbology | Example | 
| :--- | :--- | :--- |
| Equity | [Symbol Name] | GOOG |
| Spot | [Symbol Name] | EUR/USD |
| Futures | [Symbol Name] [Expiration Month] [Expiration Year] | NIFTY FEB 2015 |
| Options | [Symbol Name] [Expiration Month] [Expiration Year] [Option Type] [Strike Price] | NIFTY FEB 2015 CE 8800 |

To access the Market Data Container interface of an Instrument from your strategy other than yourInstrument Variable can be done as: 
```
IMarketDataContainerInfo marketDataContainerInfo = base.GetMarketDataContainer(ExchangeSegment.NSEFO, "NIFTY FEB 2015 CE 8800"); 
if (marketDataContainerInfo != null) 
{ 
    ITouchLineInfo touchlineInfo = marketDataContainerInfo.TouchLineInfo; 
    double bestBidPrice = touchlineInfo.BestBidPrice;
     int bestBidSize = touchlineInfo.BestBidSize; 
    double bestAskPrice = touchlineInfo.BestAskPrice; 
    int bestAskSize = touchlineInfo.BestAskSize; 
    double lastPrice = touchlineInfo.LastPrice; 
    int lastSize = touchlineInfo.LastSize; 
    long lastTradedTime = touchlineInfo.LastTradedTime; 
} 
```
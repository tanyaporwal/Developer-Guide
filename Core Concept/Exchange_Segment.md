## Exchange Segment

BlitzTrader empowers you to trade in diverse markets by offering connections to various exchanges and liquidity venues. In BlitzTrader, a liquidity venue is referred to as an Exchange Segment. BlitzTrader offers connectivity to multiple markets, allowing users to utilize them as sources of market data information, order routing, or both. Each connection to an Exchange/Broker gateway system is managed by a programmable component known as an Adapter. BlitzTrader provides a comprehensive API and framework for developing and integrating exchange adapters as plugin components.
Each adapter is uniquely identified by a name identifier called the Exchange Segment. Additionally, every tradable instrument is linked to a specific exchange segment. This linkage enables the BlitzTrader framework to easily identify the adapters responsible for routing orders for a particular instrument.

Below are some enumerated exchanges identifiers based on the exchanges they belong to:

| Exchanges | Exchange Segment Identifier |
| :--- | :--- |
| NSE Futures and Options Segment | NSEFO |
| NSE Equity Segment | NSECM |
| NSE Commodity Segment | NSECOM |
| BSE Futures and Options Segment | BSEFO |
| BSE Equity Segment | BSECM | 
| MCX Futures and Options (Commodities) | MCXFO |
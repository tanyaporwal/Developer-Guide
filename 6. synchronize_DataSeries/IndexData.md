### Index Data

In the realm of the stock market, indexes play a pivotal role as benchmarks for assessing overall market performance. Commonly known indexes include the Dow Jones Industrial Average (DJIA), S&P 500, NASDAQ, CNX NIFTY, and more. Essentially, an index is a statistical measure that tracks the changes in a portfolio of stocks, representing a specific segment or the broader market.

Indexes serve as essential economic indicators, reflecting the collective value of the underlying portfolio of stocks. Although derivative contracts based on indexes are tradable, here we focus on the non-tradable index data, which is disseminated by exchanges at regular intervals.

BlitzTrader offers seamless access to index data through its API model, allowing traders to incorporate this valuable information into their trading strategies. Index data can be retrieved by its unique name, which is typically designated per exchange segment, providing traders with insights into market trends and sentiment.

Understanding the index value is vital for determining the moneyness of options, including whether they are in-the-money (ITM), at-the-money (ATM), or out-of-the-money (OTM) relative to the strike price of the instrument it represents.

```
// Retrieve index data for CNX NIFTY from NSECM exchange segment
IndexData indexDataNifty50 = GetIndexData(ExchangeSegment.NSECM, "NIFTY 50");

if (indexData != null)
{
    // Access index values
    double indexValue = indexDataNifty50.IndexValue;
    double lowIndexValue = indexDataNifty50.LowIndexValue;

    // Additional operations...
}
```

Following are some example of important indexes name as per their exchange segment :

| Index Name | Exchange Segment |
| :--- | :--- |
| NIFTY 50 | NSECM |
| NIFTY BANK | NSECM |
| INDIA VIX | NSECM |
| NIFTY MIDCAP 50 | NSECM |
| SENSEX | BSECM |
| BANKEX	| BSECM |
By incorporating index data into your trading strategies, you can make informed decisions based on market direction and sentiment.

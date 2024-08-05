### Synchronize Data Series with Trading Client Framework
BlitzTrader streamlines the visualization of your strategy's real-time state for traders, enabling seamless synchronization of vital moving state data (DataSeries) from the server to the client for graphical representation in the Trading Dashboard user interface (UI).

Scenario:
Imagine a server-hosted strategy that continually tracks technical indicators, generating fresh values for indicators and occasional signals based on trading rules at every bar close. Traders can utilize Data Series to visually interpret these live indicators and signals graphically.

BlitzTrader provides a mechanism for strategies to define Data Series objects. These series can hold various data types, including:
Integer|
Long
Double
BarData (OHLC - Open, High, Low, Close)
The strategy updates these series on the server-side, and Blitz automatically synchronizes them with the client for real-time display.
Available Data Series Types:
BlitzDoubleDataSeries: Stores double-precision floating-point numbers.
BlitzIntDataSeries: Stores integer values.
BlitzLongDataSeries: Stores long integers.
BlitzBarDataSeries: Stores OHLC BarData objects.
To register a Data Series, use an API function like RegisterDoubleBlitzDataSeries for a specific data type. This function requires a unique name identifier and returns a reference to the created Data Series object. Any subsequent data updates are automatically reflected on the client-side.

```
private BlitzDoubleDataSeries _mtmDataSeries = null;
protected override void OnInitialize()
{
    .......................................
    .......................................
    string errorString = string.Empty;
    if (!base.RegisterDoubleBlitzDataSeries("MTMSeries", out _mtmDataSeries, out errorString))
    {
        TraceLogWarning("BlitzDataSeries [MTMSeries] registration failed. : Reason" + errorString);
    }
}
....................
....................
double currentMTM = GetCurrentTransactionMTM();

// Add new add point to a series.
_mtmDataSeries.Add(currentMTM);
```

This framework ensures that any changes in the server-side state are promptly reflected on the client-side, providing traders with up-to-date graphical representations of critical strategy data.

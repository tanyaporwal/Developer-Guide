### OnInitialize Method
When overridden in a derived class, the OnInitialize method gives the strategy implementation a chance to initialize the strategy-level variables and other important states of the Strategy. This method is called once during the active lifetime of the strategy instance and is akin to the constructor of a class, which is called once to initialize the state of your object.

If the initialization fails, the strategy instance will be in a bad state and will never transition into a Start running state. In such cases, you need to throw an Exception with a reason text to let the trader know the reason for the failure of your Strategy Instance Initialization. Ideally, the OnInitialize method should not fail.

Typical implementation logic placed in the OnInitialize method based on your strategy requirements could include:

```
protected override void OnInitialize()
{
    _ivInfo = base.GetIVInfo(_ivInstrument);

    if (_ivInfo.InstrumentName.ToUpper().StartsWith("BANKNIFTY"))
    {
        _strikeGap = 100;
        _instrumentFutUnderlineName = "BANKNIFTY-I";
        _bbProfitMargin = 100;
                
     }
     else if (_ivInfo.InstrumentName.ToUpper().StartsWith("FINNIFTY"))
     {
        _strikeGap = 100;
        _instrumentFutUnderlineName = "FINNIFTY-I";
        _bbProfitMargin = 50;
     }
     else if (_ivInfo.InstrumentName.ToUpper().StartsWith("NIFTY"))
     {
         _strikeGap = 100;
        _instrumentFutUnderlineName = "NIFTY-I";
        _bbProfitMargin = 50;
     }
     else
        throw new Exception("Underline not supported.");

     _timeDataSeries = LoadHistoricalData(instrumentFutUnderlineName);

    _superTrendIndicatorFast =  QX.FinLib.Data.IndicatorRegistrationManager.Instance.GetIndicator("Supertrend", _ timeDataSeries);
    _superTrendIndicatorFast.SetFieldValue("LengthPeriod", 10);
    _superTrendIndicatorFast.SetFieldValue("Multiplier", 1);

    // Bind a callback method that is called when 5 Minutes realtime Bar will be periodically completed
    _ timeDataSeries.OnBarDataCompletedEvent += TimeDataSeriesFutures_OnBarDataCompletedEvent;
 
}
```

> **Note:** OnInitialization method is called as a first method among all the override methods of Strategy when Strategy Instance is loaded or created first time. Blitz ensures that all the static and preexisting dynamic IVObjects must be mapped to the actual market before OnInitialization method is called.


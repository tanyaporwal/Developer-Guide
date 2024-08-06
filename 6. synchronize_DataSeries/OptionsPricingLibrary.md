### Options Pricing Library

Options represent a highly liquid asset class traded across numerous exchanges globally. Strategies centered around options often demand precise insights into options Greeks for effective risk management and portfolio optimization. BlitzTrader offers a comprehensive Options Pricing Library tailored to enhance your options trading strategies.

For instance, consider a strategy like Implied Volatility (IV) scalping, which allows you to manage directional risk within your portfolio through Delta Hedging. This approach leverages Delta to calculate the necessary hedge position for open option contracts. By adjusting your exposure to the underlying asset based on Delta, you can effectively neutralize the directional risk inherent in options positions.
The BlitzTrader API includes an implementation of the classic Black-Scholes option model, enabling the pricing of both Calls and Puts. Additionally, it provides calculations for IV (Implied Volatility) and essential Greeks such as delta, gamma, theta, vega, and rho. This library plays a pivotal role in strategies like IV Scalping, facilitating the computation of delta positions necessary for hedging open options positions by trading underlying contracts.

The BlackScholes and OptionsGreek classes provide API methods for evaluating options pricing and Greeks, empowering traders to make informed decisions based on accurate pricing models.

```
private IVObject _ivOptionObject = new IVObject("OptionInstrument",
                                                                                          "Option Instrument",
                                                                                           true,
                                                                                           InstrumentType.Options,
                                                                                           MarketDataType.All, OrderEventType.All);

 private IVObject _ivUnderlyingObject = new IVObject("UnderlyingInstrument",
                                                                                          "Underlying Instrument",
                                                                                            true,
                                                                                           InstrumentType.Equity | InstrumentType.Futures,
                                                                                           MarketDataType.All, OrderEventType.All);
    
    private IVInfo _ivInfoOption = base.GetIVInfo(_ivOptionObject);


    private Options _optionInstrument = ((Options)_ivInfoOption.IVInstrument.Instrument);



    private IVInfo _ivInfoUnderlying = base.GetIVInfo(_ivUnderlyingObject);
    
    private int _optionMaturityDays = _optionInstrument.RemainingExpiryDays;
    private double _timeToMaturityInYearsInABS = (double)_optionMaturityDays / 365;
    private double _riskFreeInterestRateInABS = 0.01;
    private double _dividendYieldInABS = 0;

    private double GetCallOptionsImpliedVolatility(double optionPrice, double underlyingPrice)
    {
        double marketVolatility = 0;
        marketVolatility = BlackScholes.GetCallInitialImpliedVolatility(
                                                      underlyingPrice, 
                                                    _optionInstrument.StrikePrice, 
                                                    _timeToMaturityInYearsInABS, 
                                                    _riskFreeInterestRateInABS, 
                                                    optionPrice, 
                                                   _dividendYieldInABS) / 100;
        return marketVolatility;
    }

    private double GetPutOptionsImpliedVolatility(double optionPrice, double underlyingPrice)
    {
        double marketVolatility = 0;
        marketVolatility = BlackScholes.GetPutInitialImpliedVolatility(underlyingPrice, 
                                                                                              _optionInstrument.StrikePrice,
                                                                                              _timeToMaturityInYearsInABS, 
                                                                                              _riskFreeInterestRateInABS, 
                                                                                              optionPrice, 
                                                                                              _dividendYieldInABS) / 100;
        return marketVolatility;
    }

    private double GetUnderlyingDelta(OrderSide underlyingOrderSide, double ivValue)
    {
        double underlyingPrice = 0;
        if (underlyingOrderSide == OrderSide.Buy)
            underlyingPrice = _ivInfoUnderlying.MarketDataContainer.TouchLineInfo.BestAskPrice;
        else if (underlyingOrderSide == OrderSide.Sell)
            underlyingPrice = _ivInfoUnderlying.MarketDataContainer.TouchLineInfo.BestBidPrice;

        if (underlyingPrice <= 0)
            return 0;

        double delta = 0;
        if (_optionInstrument.OptionType == OptionType.CA || _optionInstrument.OptionType == OptionType.CE)
        {
            delta = OptionsGreeks.GetCallOptionDelta(underlyingPrice, 
                                                                                     _optionInstrument.StrikePrice,
                                                                                    _riskFreeInterestRateInABS,
                                                                                     ivValue, 
                                                                                   _timeToMaturityInYearsInABS, 
                                                                                 _dividendYieldInABS);
        }
        else if (_optionInstrument.OptionType == OptionType.PA || _optionInstrument.OptionType == OptionType.PE)
        {
            delta = OptionsGreeks.GetPutOptionDelta(underlyingPrice,
                                                                                 _optionInstrument.StrikePrice, 
                                                                                 _riskFreeInterestRateInABS, 
                                                                                 ivValue, 
                                                                               _timeToMaturityInYearsInABS,
                                                                              _dividendYieldInABS);
        }

        return delta;
    }
```

This robust library, equipped with functionalities like BlackScholes and OptionsGreek classes, empowers traders to make informed decisions and execute strategies with confidence, leveraging accurate pricing models and risk management techniques.
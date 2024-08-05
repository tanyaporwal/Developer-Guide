### IVObjects Getter Override Method

This method is mandatory to implement, and it must return the list of all static IVObjects defined in your strategy. If the strategy does not define any IVObject, this method returns an empty list.

The IVObject represents an abstraction of an instrument or market. During the instance creation of the strategy, the static IVObject defined is bound with a specific market, such as the NIFTY29MAR24FUT Futures contract of NSEFO or ESH4 of CME. Blitz provides over 20 Exchange connectivity adapters, and IVObjects can be bound to any configured market. Any new market can be developed and plugged into the system in a very short time frame.

```
public class MainStrategy  : StrategyBase
{

    private IVObject _ivFuturesLeg = new IVObject("FuturesIV", "Futures Instrument", true,
                                                           InstrumentType.Futures,
                                                           MarketDataType.All, OrderEventType.All);

    private IVObject _ivCallLeg = new IVObject("CallIV", "Call Instrument", true,
                                                           InstrumentType.Options,
                                                           MarketDataType.All, OrderEventType.All);

    private IVObject _ivPutLeg = new IVObject("PutIV", "Put Instrument", true,
                                                           InstrumentType.Options,
                                                           MarketDataType.All, OrderEventType.All);

    private IVInfo _ivInfoFutures, _ivInfoCallOptions, _ivInfoPutOptions;
     

      public override IVObject[] IVObjects
      {
            get
            {
                List<IVObject> _ivObjectArray = new List<IVObject>();
                _ivObjectArray.Add(_ivFuturesLeg);
                _ivObjectArray.Add(_ivCallLeg);
                _ivObjectArray.Add(_ivPutLeg);

                return _ivObjectArray.ToArray();
            }
        }

        protected override void OnInitialize()
        {
            _ivInfoFutures = base.GetIVInfo(_ivFuturesLeg);
           _ ivInfoCallOptions = base.GetIVInfo(_ivCallLeg);
           _ ivInfoPutOptions = base.GetIVInfo(_ivPutLeg);
        }
}
```
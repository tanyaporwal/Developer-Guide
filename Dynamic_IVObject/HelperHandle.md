### IVInfo, The Helper Handle

 The IVInfo serves as a helper object handle, providing essential information related to the bindable instrument of the IVObject handle defined in the strategy.
The information provided by IVInfo is primarily categorized into three types:
Instrument Property: This category allows access to all instrument properties such as Name, InstrumentID, Exchange Segment, TicksPerPoint (TPP), LotSize, Expiry Date, Strike Price, Options Type, etc.
Market Data Price Information: IVInfo facilitates real-time access to price information, including level-I and level-II prices, Best Bid Price, Best Ask Price, Last Traded Price, Full 5 level Depth, etc.
Statistics: In case any trades are executed corresponding to the IVObject, IVInfo provides statistical data such as Net Position, TurnOver, Buy Value, Sell Value, Trade Count, etc.

![IVObject](https://github.com/pcnetworking/Markdown/blob/main/images/IVObject.png?raw=true)

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

If your strategy is designed to create a dynamic IVObject using a helper function like

private void CreateRuntimeIVObject(Options options)

You must add following code snippet in OnInitialize method to get handle of these registers dynamic IVObject

```
        protected override void OnInitialize()
        {
            _ivInfoFutures = base.GetIVInfo(_ivFuturesLeg);
           _ ivInfoCallOptions = base.GetIVInfo(_ivCallLeg);
           _ ivInfoPutOptions = base.GetIVInfo(_ivPutLeg);

           // Reinitialize registered dynamic IVs persisted by the strategy instance
            IVObject[] ivObjectList = base.TradableIVObjects;
            foreach (IVObject optionIVObject in ivObjectList)
            {
                // Add your code to store the dynamic IVObject in internal collection to be used for later trading logics
            }

        }
```
The method TradableIVObjects returns all the IVObject registered by the strategy instance including both static and dynamic IVObject. Blitz system persist all the IVObjects details and available at anytime unless the IVs are not explicitly clean from the Blitz persistence layer. We always recommend cleaning the dynamic IVObject as an EOD process from the persistence layer using Blitz provided DB Utility tool.



The Instrument, representing concrete Market to trade

The Instrument serves as the concrete representation of a tradable market within BlitzTrader. BlitzTrader employs a process to import instrument masters from data sources defined by respective exchanges. Once imported, the Instrument is stored in memory and can be referenced by a Strategy as needed. Since IVObjects are bound to specific Instruments and Strategies often interact with IVObjects. Handling IVObjects is made easier by working with mapped IVInfo objects.

IVInfo provides a method to return the mapped Instrument object. The following code snippet illustrates how to access corresponding Instrument objects and any relevant properties:

```
// Access mapped Instrument from corresponding IVInfo handler
QX.Base.Common.InstrumentInfo.Instrument instrument = _ivInfo.IVInstrument.Instrument;
            
double bpv = instrument.BigPointValue;
string cfiCode = instrument.CfiCode;
int deceimalDisplace = instrument.DecimalDisplace;
string instrumentDisplayName = instrument.DisplayName;
uint exchangeInstrumentID = instrument.ExchangeInstrumentID;
ExchangeSegment exchangeSegment = instrument.ExchangeSegment;
if(exchangeSegment == ExchangeSegment.NSEFO)
{

}

long instrumentID = instrument.InstrumentID;
string instrumentName = instrument.Name;
double tickSize = instrument.TickSize;
      
// Convert Instrument to Options Instrument if type it represent is Options   
if(instrument.InstrumentType == InstrumentType.Options)
{
                QX.Base.Common.InstrumentInfo.Options optionsInstrument = (Options)_ivInfo.IVInstrument.Instrument;
                OptionType optionType = optionsInstrument.OptionType;
                if(optionType == OptionType.CE)
                {
                    // European Call Options
                }

                double strikePrice = optionsInstrument.StrikePrice;
                DateTime contractExpirationDT = optionsInstrument.ContractExpiration;

                double lastTradedPrice = base.GetMarketDataContainer(optionsInstrument.InstrumentID).LastPrice;

                // Access market data price information from Instrument
                IMarketDataContainerInfo marketDataContainerInfo = base.GetMarketDataContainer(optionsInstrument.InstrumentID);
                double bestBidPriceTA4thLevel = marketDataContainerInfo.GetBidPriceAt(4);
}
```

Tick Size - the minimum price change up or down of a trading instrument in a market
The Instrument object is convertible to its respective types, such as Options and Futures. In your Strategy code, you can access a complete Instrument master. The collections of all Instruments can be accessed through the InstrumentProvider handle.

Instrument object is convertible to respective type

Options optionsInstrument = (Options)Instrument;
Futures futuresInstrument = (Futures)Instrument;

From your Strategy code you can access a complete Instrument master. The collections of all Instrument is accessed through handle InstrumentProvider.

IInstrumentProvider instrumentProvider = base.InstrumentProvider;

Following code snippet show to filter call options of specific expiry from Instrument master collection

```
double underlinePrice = _ivInfo.MarketDataContainer.LastPrice;
DateTime futuresLTD = _ivInfo.MarketDataContainer.LastTradedTimeDT;

DateTime dtContractWeeklyExpiration = _optionsInstrumentAccessor.GetContractExpiryForDate(futuresLTD);
string weeklyExpiryDTText = dtContractWeeklyExpiration.ToString("ddMMMyyyy").ToUpper();

IInstrumentProvider instrumentProvider = base.InstrumentProvider;
IOptionsInstrumentContainer optionsInstrumentContainer = instrumentProvider.GetAllOptionInstruments(_ivInfo.IVInstrument.ExchangeSegment);

Options[] optionInstrumentsCall = (Options[])optionsInstrumentContainer.Find(iterator => iterator.Name ==                  
                                             _ivInfo.IVInstrument.Instrument.Name &&
                                                        iterator.OptionType == OptionType.CE &&
                                                        iterator.ContractExpirationString.Equals(weeklyExpiryDTText, StringComparison.InvariantCultureIgnoreCase))
                                                        .AvailableInstrument.OrderBy(iterator => iterator.ContractExpiration)
                                                        .ThenByDescending(iterator => iterator.StrikePrice).ToArray();
```


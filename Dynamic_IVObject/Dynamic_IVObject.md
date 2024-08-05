# Dynamic IVObject

The concept of Dynamic IVObject is necessary when the instrument to be traded is dynamically defined.

For example, let's consider a scenario where you want to trade call and put ATM (At-The-Money) options instruments that exist at a given time, say 09:20 AM based on underline spot price. At this time, you first identify the ATM call and put options instruments using a Blitz options helper library:

Dynamc IVObject concept is required when the instrument going to be traded is dynamically defined.

Let’s say you want to trade a call and put ATM options instrument exist at given time 09:20 AM.
Here at 09:20 AM you first identify the ATM options instrument using a Blitz options helper library.

```
Identify the actual instrument of call and put options
QX.Base.Common.InstrumetInfo.Options callOptionsInstrument = GetATMCallOptions();
QX.Base.Common.InstrumetInfo.Options putOptionsInstrument = GetATMPutOptions();
```
```
Use following helper function to Register the Instrument with dynamic IVObject
CreateRuntimeIVObject(callOptionsInstrument);
CreateRuntimeIVObject(putOptionsInstrument);
```

In the above code snippet:

The GetATMCallOptions() and GetATMPutOptions() functions retrieve the ATM call and put options instruments.
The CreateRuntimeIVObject function creates a new instance of IVObject and registers it using the RegisterRuntimeIVObject helper function, mapping it to the actual instrument it represents as the second argument.
Here's the implementation of the CreateRuntimeIVObject function:

```
private void CreateRuntimeIVObject(Options options)
{
    IVObject optionIVObject = new IVObject(options.DisplayName, options.DisplayName, true,
                                                                    InstrumentType.Options, MarketDataType.All, OrderEventType.All);

    IVInfo optionsIVInfo = base.GetIVInfo(optionIVObject);
    if (optionsIVInfo == null)
    {
         string errorString = string.Empty;
         if (!base.RegisterRuntimeIVObject(optionIVObject, options, out errorString))
         {
             throw (new Exception(string.Format("Could not register Runtime IVObject[{0}].", options.DisplayName)));
         }

         optionsIVInfo = base.GetIVInfo(optionIVObject);
     }
}
```

**In this function:**
-	A new instance of IVObject is created with the necessary details.
-	The RegisterRuntimeIVObject function is called to register the IVObject, mapping it to the specified options instrument.
-	If the registration fails due to the IVObject already being registered with the same name identifier, an exception is thrown. The Blitz system assumes that a strategy should not have multiple IVObjects mapped to the same physical instrument.


> Note: Both dynamically and statically defined IVObjects are persisted by the Blitz trading system and remain available during the next run of the same strategy instance. It is recommended to avoid creating dynamic IVObjects unless you are certain they will be used for trading in subsequent steps. 

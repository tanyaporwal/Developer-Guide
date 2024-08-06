### Setting Exchange Client properties to SmartOrder

In trading, every order sent to the exchange is executed on behalf of a client or a proprietary trading book (prop book). The Strategy framework offers the flexibility to override the default client properties set by the Blitz Admin. Suppose the user account associated with the Strategy instances has default settings provided by the Blitz Admin. In that case, developers can modify any properties as needed, and these properties may influence the communication with the exchange.

Here's how you can set the Exchange Client properties to a SmartOrder:

```
UserExchangeProperty userExchangeProp = base.GetUserExchangeProperty(optionsIVInfo.IVInstrument).Clone();
userExchangeProp.ExchangeClientID = Input_ClientID;

_entryOrderCommand.SetUserExchangeProperty(userExchangeProp);
```

**In this code snippet:** fvf

-	We first retrieve the user's exchange properties for the specified instrument, and then clone them to avoid modifying the original properties directly.
-	Next, we override the default Exchange Client ID with the input Client ID provided by the developer.
-	Finally, we set the modified user exchange properties to the SmartOrder Command using the SetUserExchangeProperty method, ensuring that the SmartOrder executes orders with the updated client properties.
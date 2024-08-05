### OnStart Override Method

The Strategy Instance maintains two major state: Stopped or Started and it toggle between these two state.
Once the Strategy is initialized, it is by default in the Stopped state. Under Stopped state, the strategy will receive real-time market events and can perform any computation but will ignore any order-routing-related. The order routing is only possible if Strategy State is Started. 

The method OnStart is automatically called when the Strategy instance is explicitly started from the trading dashboard. This method enables the Strategy to capture a Started event and perform any necessary business logic to validate if the Strategy is in a suitable state to execute an order-routing mechanism. The method must return true if all states are satisfactory or return false along with the appropriate reason for the start failure.

If it return false from Strategy instance code, the attempt to start the Strategy will fail and User dashboard will displayed the reason of Start Strategy failure.

```
protected override bool OnStart(out string errorString)
{
    errorString = "";
    return true;
}
```
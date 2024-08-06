### Validate Strategy Parameter Override Method

The Strategy's input parameters, controlled by the Trader, reflect any changed values from the trading dashboard in real-time within the Strategy Instance's bound variables. However, there's a possibility that a changed value might not be valid or could pose a risk to Strategy execution if set incorrectly. To mitigate this, the ValidateStrategyParameter method offers the Strategy an opportunity to validate the proposed value before applying it to the Strategy code's bound variable. If the value is found to be inappropriate, the Strategy can reject the change request.


The Strategy input parameters defined from the Strategy are controlled by the Trader, and any changed value from the trading dashboard is reflected in the bound variable of the Strategy Instance in real-time.
There could be a chance that the changed value is not valid or could pose a risk to Strategy execution if wrongly set. The strategy has the opportunity to validate the value of the input parameter change before it is applied to the bound variable of the Strategy code. The Strategy developer can reject the change request if the value is not appropriately set.
The ValidateStrategyParameter method is invoked during a request to change the value of an input variable within the Strategy Instance. This gives the Strategy an opportunity to scrutinize the proposed input value and reject it if it's deemed invalid for application to the mapped variable.

Below is a code snippet demonstrating how to implement the ValidateStrategyParameter method to validate input variables UserBenchmark and OrderLotQuantity :

```
[StrategyParameterAttribute("UserBenchmark ", DefaultValue = 500, Description = "User Benchmark.", CategoryName = "General", DisplayName = "User Benchmark")]
private double Input_UserBenchmarke = 500;

protected override bool ValidateStrategyParameter(string parameterName, object paramterValue, out string errorString)
 {
            errorString = string.Empty;
            bool retValue = false;

            switch (parameterName)
            {
                case  "UserBenchmark":    // UserBenchmark  is defined as Input parameter
                {
                        retValue = paramterValue is double;
                        if (retValue == false || retValue <= 0)
                            errorString = parameterName + " value is invalid";
                 } 
                 break;
                 case  "OrderLotQuantity":   // OrderLotQuantity is defined as Input parameter
                 {
                        retValue = paramterValue is double;
                        if (retValue == false || retValue < 0 || retValue > 50 || retValue %_ivInfo.LotSize != 0)
                            errorString = parameterName + " value is invalid";
                 }
                 break;
                 default:
                    retValue = base.ValidateStrategyParameter(parameterName, paramterValue, out errorString);
                    break;
            }

            return retValue;
}
```

In this implementation :

For the UserBenchmark parameter, the method checks if the proposed value is a positive double. If the conditions are met, it returns true, indicating the value is valid for application to the Strategy input variable. Otherwise, it returns false.

For the OrderLotQuantity parameter, the method verifies if the proposed value is an integer and a multiple of the instrument's lot size. If both criteria are satisfied, it returns true. Otherwise, it returns false.
The method includes a default case to handle other input parameters if necessary, accepting the proposed value by default.

Each input parameter change request should invoke this method to ascertain whether the proposed value is acceptable. If true is returned, the value is applied to the corresponding Strategy input variable; otherwise, it's rejected.

**Validate Strategy Parameter**

The ValidateStrategyParameter method is called every time a Strategy Input Parameter is successfully applied to mapped variables. Here, the Strategy developer has an opportunity to implement post-processing logic based on the changed variable state.

Consider a scenario where the Strategy defines an input parameter to accept user input for squaring off all trading positions, specified in a time format of "HH:MM:SS".

```
[StrategyParameterAttribute("SquareOffTime ", DefaultValue ="15:20:00", Description = "SquareOff time in HH:mm:ss format")]
private string Input_SquareOffTime = "15:20:00";

private DateTime _squareOffTime = DateTime.Now.AddHours(10);
```

The user has defined a local variable named _squareOffTime to store the user input time format in "HH:mm:ss". This input will be converted to system time for easy comparison, allowing for the squaring off of positions once the market data timestamp breaches the square time. The user can utilize the ValidateStrategyParameter method to keep the _squareOffTime value in sync with the Input_SquareOffTime value.

```
protected override void OnStrategyParamterChanged()
{
            base.OnStrategyParamterChanged();

            DateTime today = DateTime.Today;

            // Combine today's date with the timestamp
            string combinedDateTimeString = today.ToString("yyyy-MM-dd") + " " + Input_SquareoffTime;

            // Parse the combined string to a DateTime object
            _squareOffDateTime = DateTime.ParseExact(combinedDateTimeString, "yyyy-MM-dd HH:mm:ss", null);

 }
```


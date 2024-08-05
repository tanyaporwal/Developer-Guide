## Strategy Commands Concept
In the realm of trading strategies, the ability to promptly control operations as they are initiated from the trading dashboard is crucial. Within the Blitz Strategy framework, developers are empowered to articulate bespoke operations known as commands, each imbued with specific logic for execution. However, it is essential to note that these commands must be activated exclusively through the trading dashboard.

For example, a Strategy could define a Command to adjust all tradable positions to a certain percentage, another to promptly execute a square-off position, or even one to retrieve specific states to the Strategy Instance for display on the trading dashboard.

The Strategy provides an override method to inform the framework of the list of commands it supports. Each command must be defined as an ActionCommandInfo object, requiring a unique Guid ID to represent it, along with a display name and optionally a list of input parameters.

Below is code snippet to define a function that return a command with text "Go Flat".

```
private static readonly Guid GoFlatCommandStaticID = new Guid("{4775106D-E7FA-4538-9CD9-304B1BBEE954}");

private ActionCommandInfo GetGoFlatActionCommand()
{
            List<ActionCommandFieldInfo> actionCommandParameterInfoList = new List<ActionCommandFieldInfo>();
            return CreateActionCommand(GoFlatCommandStaticID, "Go Flat", false, actionCommandParameterInfoList.ToArray());
}
```

Let's define a Strategy command that incorporates input parameters. This command is designed to execute an action: reducing the position of all Strategy Instance instruments by a certain percentage. The percentage value is specified by the trader from the trading dashboard and is dictated by the command.

```
private static readonly Guid ReducePositionCommandStaticID = new Guid("{2E5B9278-4745-406E-87EB-9226B03435BB}");

private ActionCommandInfo GetReducePositionActionCommand()
{
            List<ActionCommandFieldInfo> actionCommandParameterInfoList = new List<ActionCommandFieldInfo>();
            actionCommandParameterInfoList.Add(new ActionCommandFieldInfo("Reduced%", FieldDataType.Double, 0));
          
            return CreateActionCommand(ReducePositionCommandStaticID, "Reduce Position", true, actionCommandParameterInfoList.ToArray());
}
```

In the provided code snippet, the command input is defined using ActionCommandFieldInfo. This encapsulates the field name, data type, and a default value for the parameter.

When accessed through the Trading dashboard, this command automatically appears on each Strategy instance row within a context menu. Upon invocation by the user, the command prompts an input parameter box to populate the desired percentage. Once the user initiates execution, the command request is relayed to the Strategy instance for execution.

The developer must implement a generic method for all command invocation and implements the cases for each command execution as their business logic.

To handle all command invocations uniformly, the developer must implement an override method named ExecuteActionCommand. Subsequently, specific cases for each command execution are implemented according to their respective business logic.

```
protected override void ExecuteActionCommand(Guid commandStaticID, ActionCommandFieldInfo[] inputFields)
{
            base.ExecuteActionCommand(commandStaticID, inputFields);

            if (GoFlatCommandStaticID.Equals(commandStaticID))
            {
                // SquareOff all open position based on smart executor
                foreach (IOrderCommandEx orderCommandEx in _orderExecutionMAP.Values)
                {
                    if (orderCommandEx.IVInfo.Statistics.NetPosition != 0)
                    {
                        orderCommandEx.UserText = "Go Flat manually invoked";
                        orderCommandEx.GoFlat();
                    }
                }
            }
            else if (ReducePositionCommandStaticID.Equals(commandStaticID))
            {
                ActionCommandFieldInfo reducedPositionPercentageField = 
                          inputFields.FirstOrDefault(iterator => iterator.Name.Equals("Reduced%n", StringComparison.InvariantCultureIgnoreCase));

                if (reducedPositionPercentageField == null)
                {
                    return;
                }

                double reducedPositionPercentageValue = (double)reducedPositionPercentageField.Value;

                if (reducedPositionPercentageValue > 0)
                {
                    foreach (IOrderCommandEx orderCommandEx in _orderExecutionMAP.Values)
                    {
                        if (orderCommandEx.IVInfo.Statistics.NetPosition != 0)
                        {
                            orderCommandEx.UserText = "Reduce position";
                            int newPosition = GetNewPosition(orderCommandEx);
                            orderCommandEx.SetPosition(0, newPosition);
                        }
                    }
                }

            }
}
```
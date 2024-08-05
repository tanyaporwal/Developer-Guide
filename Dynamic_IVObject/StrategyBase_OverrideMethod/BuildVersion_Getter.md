### BuildVersion Getter Override Method
The Strategy provides an override method to assign a unique version number to your strategy. This allows you to increment the versioning whenever a new change is released in the production state of the strategy. The version number is displayed in logs and the trading dashboard.
The version is represented by four values: Major, Minor, Build, and Revision.

```
public override Version BuildVersion
{
    get
    {
        return new Version(1, 05, 09, 09);
    }
}
```
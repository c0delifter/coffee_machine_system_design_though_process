### A C# thought exercise in inheritance, interface implementation, and dependency injection 🤔

# Questions

1. You are in charge of creating a system that stores coffee machines. Each coffee machine has a brewing unit but only some have grinders.
2. For example, in future I may ask you to extend the design to store coffee machines with a WIFI coffee reordering module and so on.
3. How would an instance of a coffee machine be created? 

# Answers

To design such a system, you can either use inheritance (parent and child classes) or interfaces. 

Personally, I'd mix and match the two to get the most optimal result.

Let's start with a baseline interface. This will be our foundation

```
using System.Threading.Tasks;

public interface ICoffeeMachine
{
    Task BrewCoffee();
    void DisplayInfo();
}
```

We then create base CoffeeMachine class. Every coffee machine will have a `Make` and `Model` property. It will also implement the ICoffeeMachine interface above.

I.e. every coffee machine we produce can brew coffee. If it couldn't, it wouldn't be a very good coffee machine - now would it? :]

```
public class CoffeeMachine : ICoffeeMachine
{
    private readonly string _make;
    private readonly string _model;

    public CoffeeMachine(string make, string model)
    {
        _make = make;
        _model = model;
    }

    public string Make => _make;
    public string Model => _model;

    public virtual void DisplayInfo()
    {
        Console.WriteLine($"Coffee Machine Make: {_make}, Model: {_model}");
    }

    public async Task BrewCoffee()
    {
        Console.WriteLine("Starting the brewing process...");
        await Task.Delay(2000); // Simulate brewing time
        Console.WriteLine("Coffee is ready!");
    }
}
```

If our OEM company wanted to start producing more types of coffee machines, it wouldn't make sense to tinker with the original design. If it ain't broken, why fix it?

Instead, we would built on top of the original model by creating extensions and variations.

The [O] in [SOLID]. 

Open for extension, closed for modification.

Let's imagine we create a new version with a grider functionality. 

First we create a new interface called `IGrinderUnit`

```
using System.Threading.Tasks;

public interface IGrinderUnit
{
    Task Grind();
}
```

Then we create a subclass that inherits the original `CoffeeMachine` class. But expands it by implementing the `IGrinderUnit` interface.

```
public class GrindingCoffeeMachine : CoffeeMachine, IGrinderUnit
{
    public GrindingCoffeeMachine(string make, string model) : base(make, model)
    {
    }

    // Implement Grind method from IGrinderUnit
    public async Task Grind()
    {
        Console.WriteLine("Grinding coffee beans...");
        await Task.Delay(1500); // Simulate grinding time
        Console.WriteLine("Grinding complete!");
    }

    // Override DisplayInfo to include grinding capability
    public override void DisplayInfo()
    {
        base.DisplayInfo();
        Console.WriteLine("This coffee machine also has a grinding unit.");
    }
}
```

Similarly, we can create a 3rd version of the coffee machine with Wi-Fi reordering functionality.

```
using System.Threading.Tasks;

public interface IWiFiReorderingModule
{
    Task Reorder();
}

public class CoffeeMachineWithWiFiReordering : CoffeeMachine, IWiFiReorderingModule
{
    public CoffeeMachineWithWiFiReordering(string make, string model) : base(make, model)
    {
    }

    // Implement Reorder method from IWiFiReorderingModule
    public async Task Reorder()
    {
        Console.WriteLine("Connecting to the WiFi reordering system...");
        await Task.Delay(1500); // Simulate connection time
        Console.WriteLine("Order placed for coffee beans!");
    }

    // Override DisplayInfo to include WiFi reordering capability
    public override void DisplayInfo()
    {
        base.DisplayInfo();
        Console.WriteLine("This coffee machine is equipped with a WiFi reordering module.");
    }
}
```

As for how we would create a new instance of a CoffeeMachine, we've got several options.

Either instantiate a new object of the desired variety right in your code.

```
CoffeeMachineWithWiFiReordering myWiFiMachine = new CoffeeMachineWithWiFiReordering("Nespresso", "WiFi Pro");
```

The problem with that is that it creates tight coupling.

A better approach is to utilize `dependency injection`.

That's the [D] in [SOLID].

An example of that would be something like 👇

```
using System;
using System.Threading.Tasks;

public class CoffeeMachineController
{
    private readonly ICoffeeMachine _coffeeMachine;

    public CoffeeMachineController(ICoffeeMachine coffeeMachine)
    {
        _coffeeMachine = coffeeMachine;
    }

    public async Task UseCoffeeMachine()
    {
        _coffeeMachine.DisplayInfo();
        await _coffeeMachine.BrewCoffee();

        // If additional functionality is needed, use downcasting safely
        if (_coffeeMachine is GrindingCoffeeMachine grinder)
        {
            await grinder.Grind();
        }
        else if (_coffeeMachine is CoffeeMachineWithWiFiReordering wifiMachine)
        {
            await wifiMachine.Reorder();
        }
    }
}
```

Then in some other part of the code you would inject the service like this:

```
ICoffeeMachine coffeeMachine = new GrindingCoffeeMachine("Breville", "Barista Express");
var consumer = new CoffeeMachineConsumer(coffeeMachine);

await consumer.UseCoffeeMachine();
```


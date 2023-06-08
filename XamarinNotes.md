# Binding

## Absolute Vanilla .NET
Objects that want to notify other parts of the system when their properties change must implement `INotifyPropertyChanged`.
`INotifyPropertyChanged` look like:
```c#
public interface INotifyPropertyChanged
{
    // Occurs when a property value changes.
    event PropertyChangedEventHandler PropertyChanged;
}
```

In the most raw form, you could create the event handler yourself and run it at the appropriate time:
```c#
public class MyClassThatNotifiesOthers
{
    private event PropertyChangedEventHandler EventHandlerWhenNameChanges;
    private string _name;
    public string Name
    {
        get => this._name;
        set
        {
            this._name = value;
            this.EventHandlerWhenNameChanges?.Invoke(this, new PropertyChangedEventArgs(nameof(Name)));
        }
    }
}
```

The most basic (and bad) way of binding a view to some object property is to do it in the view's code-behind.
So, say I am creating a `HomePage` component. It will extend from `Xamarin.Forms.ContentPage`, which is a subclass of `Xamarin.Forms.Element`, which is a subclass of `Xamarin.Forms.BindableObject`, which implements `INotifyPropertyChanged`.
`BindableObject` implements `INotifyPropetyChanged` like so:
```c#
public class BindableObject : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;
    //...
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
```

_That means we have an easy way for our components to logically implement `INotifyPropertyChanged`, via `BindableObject.OnPropertyChanged()`._

We make our viewmodels extend from that base class and call that method when its properties actually change.
Then when we bind views to the viewmodel's properties, Xamarin can handle the visual updating.
```c#
public class MyViewModel : BindableObject
{
    private string displayString;
    public string DisplayString
    {
        get => this.displayString;
        set
        {
            if (this.displayString != value)
            {
                this.displayString = value;
                this.OnPropertyChanged(nameof(this.DisplayString));
            }
        }
    }
}
```

## Ok Xamarin provides an implementation of `INotifyProperyChanged`, so why do we use Prism and/or ReactiveUI?
Prism, ReactiveUI, and MVVMHelpers all offer their own base classes that implement `INotifyProperyChanged`.
Each also comes with more OOTB features, but they are very similar in how they go about `INotifyProperyChanged` implementation.

|Library|class to extend|extended class method to call in your VM's setter to trigger the `INotifyProperyChanged.PropertyChanged` event|
|-|-|-|
|Xamarin.Forms|`BindableObject`|`OnPropertyChanged()`|
|ReactiveUI|`ReactiveObject`|`RaiseAndSetIfChanged()`|
|Prism|`BindableBase`|`SetProperty()`|

## Alright, but what's this `[Reactive]` attribute on all the bound-to properties?
This is a Fody thing.
Fody is an extensible tool for weaving .NET assemblies, and they'll inject INotifyPropertyChanged code into properties at compile time for you.
It reduces rendundant boilerplate code, simplifying this...
```c#
private string name;
public string Name 
{
    get => name;
    set => this.RaiseAndSetIfChanged(ref name, value);
}
```

...into this
```c#
[Reactive]
public string Name { get; set; }
```
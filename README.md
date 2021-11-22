# Signals
Event-based communication with [UniDi](https://github.com/UniDi/UniDi) integration.

## Table Of Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<details>
<summary>Details</summary>
</details>
<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## What is Signal-Bus?
Signal-Bus is an event-based communication system. It allows communication between components in a loosely coupled manner, where each component never needs to know or hold explicitly each other.

## Dependencies
- [UniDi](https://github.com/UniDi/UniDi)

## Installation

### Install via Git URL
TODO --> WIP <--

### Install from file 
Download and extract a [release](https://github.com/UniDi/UniDi/releases) to your machine. Press 'Add package from disk' in the Unity Package Manager and select the ``package.json`` file in the extracted folder.

### OpenUPM
TODO: --> WIP <--

### Example usage

```csharp
public class UserJoinedSignal
{
    public string Username;
}

public class GameInitializer : IInitializable
{
    readonly SignalBus _signalBus;

    public GameInitializer(SignalBus signalBus)
    {
        _signalBus = signalBus;
    }

    public void Initialize()
    {
        _signalBus.Fire(new UserJoinedSignal() { Username = "Bob" });
    }
}

public class Greeter
{
    public void SayHello(UserJoinedSignal userJoinedInfo)
    {
        Debug.Log("Hello " + userJoinedInfo.Username + "!");
    }
}

public class GameInstaller : MonoInstaller<GameInstaller>
{
    public override void InstallBindings()
    {
        SignalBusInstaller.Install(Container);

        Container.DeclareSignal<UserJoinedSignal>();

        Container.Bind<Greeter>().AsSingle();

        Container.BindSignal<UserJoinedSignal>()
            .ToMethod<Greeter>(x => x.SayHello).FromResolve();

        Container.BindInterfacesTo<GameInitializer>().AsSingle();
    }
}
```

To run, just copy and paste the code above into a new file named GameInstaller then create an empty scene with a new scene context and attach the new installer.

There are several ways of creating signal handlers. Another common approach is by subscribing and unsubscribing. 

```csharp
public class Greeter : IInitializable, IDisposable
{
    readonly SignalBus _signalBus;

    public Greeter(SignalBus signalBus)
    {
        _signalBus = signalBus;
    }

    public void Initialize()
    {
        _signalBus.Subscribe<UserJoinedSignal>(OnUserJoined);
    }

    public void Dispose()
    {
        _signalBus.Unsubscribe<UserJoinedSignal>(OnUserJoined);
    }

    void OnUserJoined(UserJoinedSignal args)
    {
        SayHello(args.Username);
    }

    public void SayHello(string userName)
    {
        Debug.Log("Hello " + userName + "!");
    }
}

public class GameInstaller : MonoInstaller<GameInstaller>
{
    public override void InstallBindings()
    {
        SignalBusInstaller.Install(Container);

        Container.DeclareSignal<UserJoinedSignal>();

        Container.BindInterfacesTo<Greeter>().AsSingle();
        Container.BindInterfacesTo<GameInitializer>().AsSingle();
    }
}
```
As you can see in the the above examples, you can either directly bind a handler method to a signal in an installer using BindSignal (first example) or you can have your signal handler attach and detach itself to the signal. 


## Usage 
TODO

## Contributing
TODO

## Credits
TODO: Include a section for credits in order to highlight and link to the authors of your project.

## License
UniDi contributions are licensed under the Apache 2.0 license, except for contributions copied from Extenject. See [LICENSE](https://github.com/UniDi/UniDi/blob/master/LICENSE.md) for details.

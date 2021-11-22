# UniDi Signals
Event-based communication for [UniDi](https://github.com/UniDi/UniDi).

## Table Of Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Details

- [What is Signal-Bus?](#what-is-signal-bus)
- [Dependencies](#dependencies)
- [Installation](#installation)
  - [Install via Git URL](#install-via-git-url)
  - [Install from file](#install-from-file)
  - [OpenUPM](#openupm)
  - [Example usage](#example-usage)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## What is Signal-Bus?
Signal-Bus is an event-based communication system. It allows communication between components in a loosely coupled manner, where each component never needs to know or hold explicitly each other.

## Dependencies
- [UniDi](https://github.com/UniDi/UniDi)

## Installation

Before you install this UniDi extension, please make sure that you have already installed UniDi into your project. Unity Package Manager does not do this automatically. So you need to have [UniDi](https://github.com/UniDi/UniDi) preinstalled.
Link to UniDi: [https://github.com/UniDi/UniDi](https://github.com/UniDi/UniDi)

Installation instructions are brief in this readme. As it's pretty straightforward. If you need a reminder of all the steps in deptht please visit: [UniDi Installation](https://github.com/UniDi/UniDi/blob/master/README.md#installation).

### Install via Git URL
Open the Package Manager. Select ``Add package from Git URL...``
Enter ``https://github.com/UniDi/UniDi-Signals.git`` and press ``add``.

### Install from file 
Download and extract a [release](https://github.com/UniDi/UniDi-Signals/releases) to your machine. Press 'Add package from disk' in the Unity Package Manager and select the ``package.json`` file in the extracted folder.

## Usage 

### Example

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

## Contributing
Contributing is welcome! Create a draft when you are still working on your contribution (and don't want to have it merged) or a PR to be reviewed. [Contributing guidelines](https://github.com/UniDi/UniDi/blob/master/CONTRIBUTING.md)

## License
UniDi contributions are licensed under the Apache 2.0 license, except for contributions copied from Extenject. See [LICENSE](https://github.com/UniDi/UniDi/blob/master/LICENSE.md) for details.

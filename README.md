# Diassoft Commanding - v1.0.0
[![nuget](https://img.shields.io/nuget/v/Diassoft.Commanding.svg)](https://www.nuget.org/packages/Diassoft.Commanding/) 
![GitHub release](https://img.shields.io/github/release/diassoft/Commanding.svg)
![NuGet](https://img.shields.io/nuget/dt/Diassoft.Commanding.svg)
![license](https://img.shields.io/github/license/diassoft/Commanding.svg)

The Diassoft Commanding is a component to allow the creation of commands by decorating methods with command attributes.

It provides a `CommandProcessor` base class that allows the registration and execution of commands. Other classes can be created based on the `CommandProcessor`. 
There is a standard component, `NetworkCommandListener`, which creates a listener to intercept commands sent thru a TCP/IP connection.

## In this repository

* [Target Frameworks](#target-frameworks)
* [Main Classes](#main-classes)
* [Creating Command Containers](#creating-command-containers)
* [Listening to Network Commands](#listening-to-network-commands)
    * [Additional Features of the Network Command Listener](#additional-features-of-the-network-command-listener)

## Target Frameworks

| Version | Compatibiliy | 
| :-- | :-- | 
| 1.0.0 | `netstandard2.0` |

## Main Classes

The Commanding Framework relies on three types of objects:

| Classes | Usage |
| :-- | :-- |
| `CommandProcessor` | A base class that registers, executes commands and manage sessions |
| `NetworkCommandListener` | A class that derives from the `CommandProcessor` and listens to network commands |
| `CommandClassAttribute` | Defines a class as a command container |
| `CommandAttribute` | Defines a method as a command |
| `CommandHelpAttribute` | Defines the syntax help for a command |

## Creating Command Containers

A command container is a class that contains commands. 
This class has decorated with the `CommandClassAttribute`.

> We recommend to use a proxy class instead of decorating an existing class, although both ways would work fine.

A Command Container can have different lifetimes:

| Lifetime | Description |
| :-- | :-- |
| Singleton | One single instance of the command container is created for the entire lifetime of the `CommandProcessor` |
| Scoped | One instance is created for each session.  |
| Transient | Everytime the command is called, one instance of the command container is created |

> The default Lifetime is **Scoped**

The code below represents a command container with a `GETTIME` command, which returns the current time:

```cs
[CommandClass(InstanceLifetimes.Singleton)]
public class CommandContainer
{
    [Command("GETTIME", "Gets the current time")]
    [CommandHelp(
        "Usage:\n" +
        "  GETTIME <format>\n\n" +
        "Examples:\n" +
        "  GETTIME \"yyyy-MM-dd\" --> 2019-01-01\n" +
        "  GETTIME \"MM-dd-yyyy HH:mm:ss\" --> 2019-01-01 10:00:00")]
    public string GetTime(string format)
    {
        return DateTime.Now.ToString(format);
    }
}
```

You can use the `CommandHelpAttribute` to inform the command usage example. This attribute can be setup by `CultureInfo`, for the case when the help message should be displayed in different languages.

When using a proxy class, we recommend the return type of the method to be a `CommandExecutionResponse`, to provide better control of the status, exceptions and data processed.

## Listening to Network Commands

By default, the framework exposes a `NetworkCommandListener`, which is a class that derives from the `CommandProcessor` class. 
The listener will be listening to commands into a specific port, create sessions and process commands.

The code below shows an example of how to create a `NetworkCommandListener`:

```cs
// Initialize Command Listener on port 42000
var cmd = new NetworkCommandListener(42000);

// Registers a Command Container
cmd.RegisterCommandClass<CommandContainer>();

// Start Listening to Commands
await cmd.StartAsync(CancellationToken.None);
```

The `NetworkCommandListener` expose the following events:

| Event | Usage |
| :-- | :-- |
| `ListeningStarted` | Event fired when the listening Started |
| `SessionStarted` | Event fired when a session is started |
| `CommandReceived` | Event fired when a command is received |
| `SessionEnded` | Event fired when a session ends |

### Additional Features of the Network Command Listener

By default, the `NetworkCommandListener` accept the following standard commands:

| Command | Usage |
| :-- | :-- |
| SET TIMER [ON/OFF] | When the timer is activated, the system will output a sequence of characters to the network stream to indicate the command is processing |
| SET PROMPT [ON/OFF] | Displays a character to identify that the session is awaiting for an user input |
| QUIT | Ends the communication session |

The following properties can be set to override the default behavior of the `NetworkCommandListener`:

| Property | Usage |
| :-- | :-- |
| **ShowTimer** | Displays a sequence of characters on the network stream to indicate that the command is processing |
| **ShowPrompt** | Displays a character to idenfity that the session is awaiting for an user input |
| **ResponseFormatter** | Defines a formatter for the command response. By default, the component offers the `DefaultResponseFormatter` and `SimplifiedResponseFormatter`. |

> You can create custom formatters by creating custom class that implements the `IResponseFormatter` interface
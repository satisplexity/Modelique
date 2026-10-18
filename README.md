<div align="center">
  <img src="Modelique/Assets/ModeliqueIcon.png" alt="Modelique logo" width="128" />

  # Modelique

  **A lightweight MVVM framework for .NET applications.**

  [![.NET 10](https://img.shields.io/badge/.NET-10-512BD4)](https://dotnet.microsoft.com/)
  ![Version](https://img.shields.io/badge/version-0.1.0-blue)
</div>

Modelique provides the building blocks for the Model–View–ViewModel (MVVM) pattern: observable objects, synchronous and asynchronous commands, and ViewModel-based navigation. It is designed to keep application logic separate from the UI without imposing a particular UI framework.

> **Development status:** Modelique is in initial development (`0.x`). Its public API may change before version 1.0.0.

## Features

- **Observable state:** `ObservableObject` implements `INotifyPropertyChanged`, with `SetProperty` to update values only when they change.
- **ViewModel foundation:** `ViewModelBase` provides property-change notifications for application ViewModels.
- **Commands:** `RelayCommand` implements `ICommand` for synchronous actions, including optional execution conditions and parameters.
- **Asynchronous commands:** `AsyncRelayCommand` supports `Task`-based operations, prevents concurrent execution of the same command, and reports execution failures.
- **ViewModel navigation:** `NavigationService` and `NavigationStore` manage the active ViewModel, navigation history, and back navigation.
- **Dependency injection:** Navigation resolves ViewModels using `IServiceProvider` and `Microsoft.Extensions.DependencyInjection`.

## Requirements

- .NET 10
- A compatible UI framework if you intend to bind ViewModels and commands to a user interface

## Installation

Once the package is published to NuGet:

```bash
dotnet add package Modelique --version 0.1.0
```

Or add the package to your project file:

```xml
<PackageReference Include="Modelique" Version="0.1.0" />
```

To use Modelique directly from source, add a project reference instead:

```xml
<ProjectReference Include="path/to/Modelique/Modelique.csproj" />
```

## Quick start

### Observable ViewModels

Inherit from `ViewModelBase` and use `SetProperty` to raise change notifications when a property's value changes:

```csharp
using Modelique.Core;

public sealed class CounterViewModel : ViewModelBase
{
    private int _count;

    public int Count
    {
        get => _count;
        set => SetProperty(ref _count, value);
    }
}
```

### Synchronous commands

Use `RelayCommand` to expose an action through `ICommand`:

```csharp
using Modelique.Commands;
using Modelique.Core;

public sealed class CounterViewModel : ViewModelBase
{
    private int _count;

    public int Count
    {
        get => _count;
        private set => SetProperty(ref _count, value);
    }

    public RelayCommand IncrementCommand { get; }

    public CounterViewModel()
    {
        IncrementCommand = new RelayCommand(() => Count++);
    }
}
```

You can also pass a `canExecute` predicate. Call `RaiseCanExecuteChanged()` when conditions affecting command availability change.

### Asynchronous commands

`AsyncRelayCommand` runs asynchronous work and prevents overlapping executions of the same command instance:

```csharp
using Modelique.Commands;
using Modelique.Core;

public sealed class DataViewModel : ViewModelBase
{
    private bool _isLoaded;

    public bool IsLoaded
    {
        get => _isLoaded;
        private set => SetProperty(ref _isLoaded, value);
    }

    public AsyncRelayCommand LoadCommand { get; }

    public DataViewModel()
    {
        LoadCommand = new AsyncRelayCommand(LoadAsync);
        LoadCommand.ExecutionFailed += (_, args) =>
            System.Diagnostics.Debug.WriteLine(args.Exception);
    }

    private async Task LoadAsync()
    {
        await Task.Delay(500); // Replace with your actual I/O operation.
        IsLoaded = true;
    }
}
```

The command exposes `IsExecuting` and automatically raises `CanExecuteChanged` when execution begins or ends. When calling `await LoadCommand.ExecuteAsync()` directly, exceptions propagate to the caller. When invoked through `ICommand.Execute`, failures are reported through `ExecutionFailed`.

## Navigation

Modelique provides navigation between ViewModels rather than UI pages or windows.

| API | Behavior |
| --- | --- |
| `NavigateTo<TViewModel>()` | Resolve and activate a ViewModel, adding the previous one to history. |
| `NavigateTo<TViewModel, TParameter>(parameter)` | Create a ViewModel with an additional constructor parameter. |
| `ReplaceWith<TViewModel>()` | Activate a ViewModel without pushing the current one onto history. |
| `ReplaceWith(ViewModelBase)` | Activate an existing ViewModel instance. |
| `GoBack()` | Restore the most recent ViewModel from navigation history, if any. |
| `CanGoBack` | Indicate whether the navigation history contains a previous ViewModel. |

`NavigationStore` exposes `CurrentViewModel` and raises change notifications when it changes. `NavigationService` is an **abstract** class implementing `INavigationService`; create a concrete subclass and register it together with the store and your ViewModels in your application's dependency-injection container.

For example, from a ViewModel that receives `INavigationService` through its constructor:

```csharp
using Modelique.Navigation;

public sealed class HomeViewModel : Modelique.Core.ViewModelBase
{
    private readonly INavigationService _navigation;

    public HomeViewModel(INavigationService navigation)
    {
        _navigation = navigation;
    }

    public Task OpenSettingsAsync() =>
        _navigation.NavigateTo<SettingsViewModel>();

    public Task GoBackAsync() => _navigation.GoBack();
}
```

`SettingsViewModel` is an application-defined ViewModel and must be registered with the application's service provider. The UI is responsible for displaying the active ViewModel.

## Project structure

```text
Modelique/
├── Assets/
│   └── ModeliqueIcon.png
├── Commands/
│   ├── AsyncCommandExecutionFailedEventArgs.cs
│   ├── AsyncRelayCommand.cs
│   ├── CommandBase.cs
│   └── RelayCommand.cs
├── Core/
│   ├── ObservableObject.cs
│   └── ViewModelBase.cs
├── Navigation/
│   ├── INavigationService.cs
│   ├── NavigationService.cs
│   └── NavigationStore.cs
└── Modelique.csproj
```

## Projects using Modelique

Applications built with Modelique can be featured here. If you're using it in a public project, consider opening a pull request to add a link and a brief description.

## Contributing and feedback

Bug reports, feature requests, and contributions are welcome through [GitHub Issues](https://github.com/satisplexity/Modelique/issues) and pull requests.

## Author

Created and maintained by [satisplexity](https://github.com/satisplexity).

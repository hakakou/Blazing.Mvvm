﻿# Blazor Extension for the MVVM CommunityToolkit

## Introduction

This is an expansion of the [blazor-mvvm](https://github.com/IntelliTect-Samples/blazor-mvvm) repo by [Kelly Adams](https://github.com/adamskt) that implements full MVVM support via the [CommunityToolkit.Mvvm](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/). Minor changes were made to prevent cross-thread exceptions, added extra base class types, Mvvm-style navigation, and converted into a usable library.

The library packages the support into a resuable library and includes a new `MvvmNavigationManager` class and the `MvvmNavLink` component for Mvvm-style navigation, no more hard-coded paths.

Library supports the following hosting models:
* Blazor Server App
* Blazor WebAssembly App (WASM)
* Blazor Hybrid - Wpf, WinForms, MAUI, and Avalonia (Windows only)
* Blazor Web App (.net 8.0)

There are two additional sample projects in seperate GitHub repositories:
1. [Blazor MVVM Sample](https://github.com/gragra33/MvvmSampleBlazor) - takes Micrsoft's [Xamarin Sample](https://github.com/CommunityToolkit/MVVM-Samples) project for the [CommunityToolkit.Mvvm](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/) and is converted to: Blazor Wasm & Blazor Hybrid for Wpf & Avalonia. Minimal changes were made.
2. [Dynamic Parent and Child](https://github.com/gragra33/Blazing.Mvvm.ParentChildSample) - demonstrates loose coupling of a parent component/page and an unknown number of child components using [Messager](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/messenger) for interactivity.

## Getting Started

1. Add the [Nuget package](https://www.nuget.org/packages/Blazing.Mvvm) to your project

2. Enable MvvmNavigation support in your `Program.cs` file

2-1. Blazor Server App:

```csharp
builder.Services.AddMvvmNavigation(options =>
{ 
    options.HostingModel = BlazorHostingModel.Server;
});
```

2-2. Blazor WebAssembly App:

```csharp
builder.Services.AddMvvmNavigation(options =>
{ 
    options.HostingModel = BlazorHostingModel.WebAssembly;
});
```

2-3. Blazor WebApp:

```csharp
builder.Services.AddMvvmNavigation(options =>
{ 
    options.HostingModel = BlazorHostingModel.WebApp;
});
```

2-4. MAUI Hybrid:

```csharp
builder.Services.AddMvvmNavigation(options =>
{ 
    options.HostingModel = BlazorHostingModel.HybridMaui;
});
```

3. Create a `ViewModel` inheriting the `ViewModelBase` class

```csharp
public partial class FetchDataViewModel : ViewModelBase
{
    [ObservableProperty]
    private ObservableCollection<WeatherForecast> _weatherForecasts = new();

    public override async Task Loaded()
        => WeatherForecasts = new ObservableCollection<WeatherForecast>(Get());

    private static readonly string[] Summaries =
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    public IEnumerable<WeatherForecast> Get()
        => Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
}
```

4. Register the `ViewModel` in your `Program.cs` file

```csharp
builder.Services.AddTransient<FetchDataViewModel>();
```

5. Create your Page inheriting the `MvvmComponentBase<TViewModel>` component

```xml
@page "/fetchdata"
@inherits MvvmComponentBase<FetchDataViewModel>

<PageTitle>Weather forecast</PageTitle>

<h1>Weather forecast</h1>

<p>This component demonstrates fetching data from the server.</p>

@if (!ViewModel.WeatherForecasts.Any())
{
    <p><em>Loading...</em></p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Temp. (C)</th>
                <th>Temp. (F)</th>
                <th>Summary</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var forecast in ViewModel.WeatherForecasts)
            {
                <tr>
                    <td>@forecast.Date.ToShortDateString()</td>
                    <td>@forecast.TemperatureC</td>
                    <td>@forecast.TemperatureF</td>
                    <td>@forecast.Summary</td>
                </tr>
            }
        </tbody>
    </table>
}
```

6. Optionally, modify the `NavMenu.razor` to use `MvvmNavLink` for Navigation by ViewModel

```xml
<div class="nav-item px-3">
    <MvvmNavLink class="nav-link" TViewModel=FetchDataViewModel>
        <span class="oi oi-list-rich" aria-hidden="true"></span> Fetch data
    </MvvmNavLink>
</div>
```

Now run the app.

Navigating by ViewModel using the `MvvmNavigationManager` from code, inject the class into your page or ViewModel, then use the `NavigateTo` method

```csharp
mvvmNavigationManager.NavigateTo<FetchDataViewModel>();
```

The `NavigateTo` method works the same as the standard Blazor `NavigationManager` and also support passing of a Relative URL &/or QueryString.

If you are into abstraction, then you can also navigate by interface

```csharp
mvvmNavigationManager.NavigateTo<ITestNavigationViewModel>();
```

The same principle worhs with the `MvvmNavLink` component

```xml
<div class="nav-item px-3">
    <MvvmNavLink class="nav-link"
                 TViewModel=ITestNavigationViewModel
                 Match="NavLinkMatch.All">
        <span class="oi oi-calculator" aria-hidden="true"></span>Test
    </MvvmNavLink>
</div>
<div class="nav-item px-3">
    <MvvmNavLink class="nav-link"
                 TViewModel=ITestNavigationViewModel
                 RelativeUri="this is a MvvmNavLink test"
                 Match="NavLinkMatch.All">
        <span class="oi oi-calculator" aria-hidden="true"></span>Test + Params
    </MvvmNavLink>
</div>
<div class="nav-item px-3">
    <MvvmNavLink class="nav-link"
                 TViewModel=ITestNavigationViewModel
                 RelativeUri="?test=this%20is%20a%20MvvmNavLink%20querystring%20test"
                 Match="NavLinkMatch.All">
        <span class="oi oi-calculator" aria-hidden="true"></span>Test + QueryString
    </MvvmNavLink>
</div>
<div class="nav-item px-3">
    <MvvmNavLink class="nav-link"
                 TViewModel=ITestNavigationViewModel
                 RelativeUri="this is a MvvmNvLink test/?test=this%20is%20a%20MvvmNavLink%20querystring%20test"
                 Match="NavLinkMatch.All">
        <span class="oi oi-calculator" aria-hidden="true"></span>Test + Both
    </MvvmNavLink>
</div>

```


## How MVVM Works

There are two parts:
1. The `ViewModelBase`
2. The `MvvmComponentBase`

The  `MvvmComponentBase` handles wiring up the `ViewModel` to the component.

EditForm Validation and Messaging are also supported. See the sample code for examples of how to use.


## How the MVVM Navigation Works

No more magic strings! Strongly-typed navigation is now possible. If the page URI changes, there is no more need to go hunting through your source code to make changes. It is auto-magically resolved at runtime for you!

When the  `MvvmNavigationManager` is initialized by the IOC container as a Singleton, the class will examine all assemblies and internally caches all ViewModels (classes and interfaces) and the page it is associated with. Then when it comes time to navigate, a quick lookup is done and the Blazor `NavigationManager` is then used to navigate to the correct page. IF any Relative Uri &/or QueryString was passed in via the `NavigateTo` method call, that is passed too.

**Note:** The `MvvmNavigationManager` class is not a total replacement for the Blazor `NavigationManager` class, only support for MVVM is implemented.

The `MvvmNavLink` component is based on the blazor `Navlink` component and has an extra `TViewModel` and `RelativeUri` properties. Internally, uses the `MvvmNavigationManager` to do the navigation.

## Updates

### v1.0.0 10 May, 2023

* Initial release.

### v1.0.1 19 May, 2023

* Added non-generic `RecipientViewModelBase`
* Added `ValidatorViewModelBase`

### v1.0.2 25 July, 2023

* Added Added logging at start and end of `MvvmNavigationManager` cache generation for improved debugging experience

### v1.0.2 27 July, 2023

* Fixed rare crossthread issue in MvvmComponentBase 

### v1.1.0 9 October, 2023

* Added `MvvmLayoutComponentBase` to support MVVM in the MainLayout.razor
* Updated sample project with example of `MvvmLayoutComponentBase` usage

### 26 October, 2023

* pre-release of .Net 7.0+ `Blazor Server App` support
* pre-release of .Net 8.0 RC2 `(Auto) Blazor WebApp` support

### v1.2.1 1 November, 2023

* added .Net 7.0+ `Blazor Server App` support
* new hosting model configuration support added. Special thanks to [@bbunderson](https://github.com/bbunderson) for implementation.

### v1.3.0 (beta) 1 November, 2023

* pre-release of .Net 8.0 RC2 `(Auto) Blazor WebApp` with new hosting model configuration support

### v1.4.0 21 November, 2023

* Now officially supports .Net 8.0 & .Net 7.0

## Support

If you find this library useful, then please consider [buying me a coffee ☕](https://bmc.link/gragra33).
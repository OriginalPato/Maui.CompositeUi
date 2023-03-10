# Maui Blazor Composite UI

## Getting started

1) Add a reference to `OriginalPato.Maui.Blazor.CompositeUi`
2) Register the Module services using the helper method `RegisterModuleServices` when setting up the `MauiApp` Builder in `MauiProgram.cs`. e.g 
    ```csharp
        var builder = MauiApp.CreateBuilder()
            .UseMauiApp<App>()
            .RegisterModuleServices()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            });
    ```
3) `Main.razor` will need to inject the `IModuleAssemblyService` and set the Router `AdditionalAssemblies` to be the `Assemblies` property from the `IModuleAssemblyService`.
   e.g
   ```csharp
   @using Maui.Blazor.CompositeUi
   @inject IModuleAssemblyService ModuleAssemblyService
   <Router AppAssembly="@typeof(Main).Assembly" AdditionalAssemblies="@ModuleAssemblyService.Assemblies">
   ```
4) Implement a component which will render if the component can't be found. This is where you would make the call to fetch the Dll if you are seeking to fetch remote modules.
   e.g Create a component called `LoadAssembliesComponent.razor`
   ```csharp
   @using Maui.Blazor.CompositeUi
   @inject IModuleAssemblyService ModuleAssemblyService
   @inject NavigationManager NavigationManager
   <div class="spinner-grow" role="status"></div>
   
   @code
   {
      protected override async Task OnInitializedAsync()
      {
         await base.OnInitializedAsync();
         using var client = new HttpClient();
         // Would be a good idead to fetch 'modules' from a web api
         var features = new List<Module>()
         {
            new()
            {
               ModuleName = "TestFeature1",
               DownloadUrl = "https://blazorhostedassembly.blob.core.windows.net/testing/TestingModule.dll", // REPLACE ME. TESTING ONLY
               ModuleRootPage = "/Counter3"
            }
         };
         if (await ModuleAssemblyService.GetAssemblies(features))
         {
            NavigationManager.NavigateTo(NavigationManager.Uri, forceLoad: true);
         }
      }
   }
   ```
5) Add a way to navigate to the module root pages. This is up to you :)
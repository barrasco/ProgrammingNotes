# UE4 General Tips

## Code snippets

* ### Example to hide world context pin in blueprints
```c++
UFUNCTION(BlueprintCallable, meta = (HidePin = "WorldContextObject", WorldContext = "WorldContextObject")) 
static void AddWidgetToViewport(const UObject* WorldContextObject, class UUserWidget* Widget);
```
* ### On editor context menu custom actions, make sure the the selected objects ptrs are still valid with :

```c++
auto TextAssets = GetTypeWeakObjectPtrs<UTextAsset>(InObjects);
```  

* ### Get all actors in editor level  
```c++
UEditorLevelLibrary::GetAllLevelActors()
```

* ### Copy to clipboard
```c++
FPlatformApplicationMisc::ClipboardCopy(*String);
```

* ### Quickly Read / Write ini settings on save folder
```c++
// Write (string for example, there are other functions for other var types)
GConfig->SetString(TEXT("SectionNameExample"), TEXT("VariableName"), TEXT("MyValue"), GEngineIni);

// Read the values
FString MyValue;
GConfig->GetString(TEXT("SectionNameExample"), TEXT("VariableName"), MyValue, GEngineIni);

```

* ### Get all properties from Struct / Class
```c++
// Get properties
for (UProperty* Property : TFieldRange<UProperty>(InStruct, EFieldIteratorFlags::IncludeSuper))
{
    UE_LOG(LogCategory, Log, TEXT("Property name: %s"), *Property->GetName());
}
```

* ### Custom asset actions, create a new derived class from `FAssetTypeActions_Base` and override required functions.

:::tip  When to use *transient* specifier in UPROPERTY ? 
Transient properties are always initialized to zero and are not serialized to disk. The time to use one is if the object at runtime will set the variable. 

For instance, let’s suppose you had a character that has `MaxHealth` as a value setup by the designer, which you treat as const at runtime. At runtime, you’d copy that value into a transient called `CurrentHealth`. `CurrentHealth` would go up or down and can be compared to `MaxHealth`.
:::

***

## [Build configurations](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/DevelopmentSetup/BuildConfigurations/):

### 1. Build States:
  - ### **Debug** : 
    - This configuration contains **symbols for debugging**. 
    - This configuration builds both **engine** and **game** code in debug configuration. 
    - If you compile your project using the Debug configuration and want to open the project with the Unreal Editor, you must use the "-debug" flag in order to see your code changes reflected in your project.
  - ### **DebugGame** : 
    - Same as previous, but without symbols for the Engine. This configuration builds the engine as optimized, but leaves the game code debuggable. 
    - This configuration is ideal for debugging only game modules.
  - ### **Development** : 
    - Ability to have break points and stuff in editor. 
    - This configuration enables all but the most time-consuming engine and game code optimizations, which makes it ideal for development and performance reasons. **Unreal Editor uses the Development configuration by default**. 
    - **Compiling** your project using the Development configuration e**nables you to see code changes** made to your project reflected **in the editor**.
  - ### **Shipping** : 
    - Everyhting is stripped and optimized out. You can basically just execute the game. 
    - This is the configuration for optimal performance and shipping your game. This configuration strips out console commands, stats, and profiling tools.
  - ### **Test** : 
    - This configuration is the **Shipping** configuration, but with some **console commands**, **stats**, and **profiling tools** enabled.

### 2. Build States:
  - ### **Empty** : 
    - This configuration builds a stand-alone executable version of your project, but requires cooked content specific to the platform.
    - Keep in mind that when testing a build in Development only, you should build the project only (and not solution) in your IDE, to make sure you catch all the Errors that will fire while running the RunUAT BuildPlugin command.
  - ### **Editor**:
    - To be able to open a project in Unreal Editor and see all code changes reflected, the project must be built in an Editor configuration.
  - ### **Client**:
    - On a Multiplayer project using UE4 networking features, this target designates the specified project as being a Client in UE4's client-server model for multiplayer games. 
    - If there is a `<Game>Client.Target.cs` file, the Client build configurations will be valid.
  - ### **Server**:
    - On a Multiplayer project using UE4 networking features, this target designates the specified project as being a Server in UE4's client-server model for multiplayer games. If there is a `<Game>Server.Target.cs` file, the Server build configurations will be valid.

***

## Managing Engine Builds
 - Minimize Engine changes at first
 - Use plugins where possible
 - Engine upgrades, keep doing them. But maybe stop doing updates some months before your product release.

## Naming conventions and Standards
  - Check this [Repository](https://github.com/Allar/ue5-style-guide)
  - Developer folders: each team member can have his own folder, but don't reference any anything from those, or the cook will fail,as these folders are ignored.

## Asset pipeline and repositories
  - Art Assets outside of project folder, and are imported in project.
  - Auto reimport, when unreal detects changed features
  - Manually Reimport, in context menu of folders
  - Bluetility and Python (editor only) operations can help

## Cooking and Building
  - Cooking options:
    - CookAll / Iterate / CookOnTheFly
    - Maps 
      - Specify what maps to include
      - Referenced content gets cooked, so with fewer levels included cooks may get much faster
  - You could turn off packing (on project settings)
    - During development mode it may be usefull to see exactly what assets where included, instead of just one packed asset.

***

## Performance tools: 
* GPU benchmark: 
  * Editor GPU Visualizer: `Ctrl + Shift + ,` OR console `r.ProfileGPU.ShowUI`
  * Check draw calls : `stat SceneRendering`
:::tip Reduce draw calls
    - Removing / Merging objects
    - Removing extraneous material channels
    - Reducing the ammount of shadows casted from dynamic lights
    - Reducing ammount of objects visible in your viewport
:::
  * Use `Optimization Viewmodes` to check for scene visual complexity clues:
    * `LightComplexity`
    * `ShaderComplexity`
    * `QuadOverdraw`
* CPU benchmark visualizer: `stat game`
* Memory benchmark:
  * General memory usage: `stat memory`
  * If you have some warnings :  `Window -> Statistcs` 
  * Choose `Texture Stats` in dropdown
  * You can sort by `Fully Loaded Memory` to search for problematic big textures and compare that size with LODs where it is visible
  * If some textures need resizing, you can change the `Maximum Texture Size` value on the texture property drawer, to be a smaller **Power of Two** value.
  * If textures are not in Power of Two sizes, they can impact the memory because they don't have lower mipmap textures to switch to.
    * You can fix this on the texture, by setting `Power of two Mode` , or alternatively alter the texture on your texture editing software and reimport it.
  * Alternatively you can increase your streaming memory pool size with the command : `streaming.poolsize XXXXX`
  
***

## Unreal custon engine build (github)

1. Get git clone from [Github](https://github.com/EpicGames/UnrealEngine). try using --depth 1 to avoid downloading too many history data.
2. Run `.\Setup.bat`
3. Run `\Engine\Binaries\DotNET\GitDependencies.exe --force` (4.27) or `\Engine\Binaries\DotNET\GitDependencies\win-x64\GitDependencies.exe --force` (5.x)
4. Run `.\GenerateProjectFiles.bat`
5. Run `.\Engine\\Build\BatchFiles\RunUAT.bat BuildGraph -target="Make Installed Build Win64" -script="Engine/Build/InstalledEngineBuild.xml" -set:HostPlatformOnly=true -clean -set:VS2019=true`. -set:VS2019=true is required just for 4.27.

Notes:
1. Make sure you clone this to a root folder 'c:\', so we dont have paths too long.
2. Normally use tag oficial releases branches, its safer.

## Useful links

* [UE4 Coding Standard](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/DevelopmentSetup/CodingStandard/)
* [Cpp Coding Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
* [UE4 Cookbook](https://unreal.gg-labs.com/)
* [Slate Examples](https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/Slate/Widgets/)
* [Good UPROPERTY specifiers list](https://benui.ca/unreal/uproperty/)
* [UE4 Delegates, Async, Subsystems](https://drive.google.com/file/d/1ovIX17WXnjHdfdtP9JCJ92mPZXAkrgaz/view?usp=sharing)

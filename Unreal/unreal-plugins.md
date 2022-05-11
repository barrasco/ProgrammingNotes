
# Unreal Editor / Plugins

## Plugin - Create Editor Custom Assets
1. The asset itself is a simple subtype of UObject
```c++
UCLASS(...)
class MY_MODULE_API MyCustomAsset : public UObject

GENERATED_BODY()

public:

  UPROPERTY(...)
  FText Text;
```
2. Create new subtype of UFactory for custom Editor integration (One for *Create New* and other for *Drag Drop* operations), and override any virtual funcion required for your requirements:

3.  Create New Factory Example:
```c++
.h

UCLASS(...)
class MY_MODULE_API UMyAssetFactoryNew : public UFactory
{
  GENERATED_BODY()
...
}
 // overrides declaration
 

.cpp

UMyAssetFactoryNew::UMyAssetFactoryNew(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer)
{
  // We must indicate explicitly the supported type
  SupportedClass = UMyCustomAsset::StaticClass();
  // this factory will create a new instance, rather than drag & drop
  bCreateNew = true; 
  //opens editor for asset
  bEditAfterNew = true; 
}

UObject* UMyAssetFactoryNew::FactoryCreateNew(UClass* InClass, UObject* InParent, FName InName, EObjectFlags Flags, ...)
{
  return NewObject<UMyCustomAsset>(InParent, InClass,InName, Flags);
}

bool UMyAssetFactoryNew::ShouldShowInNewMenu() const
{
  return true;
}
```

4.  Drag Drog Factory Example:

```c++
.cpp

UMyAssetFactory::UMyAssetFactory(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer)
{
  // Sets supported files to import to editor
  Formats.Add(FString(TEXT("txt;")) + NSLOCTEXT("UMyAssetFactory", "FormatTxt", "Text File").ToString();
  // We must indicate explicitly the supported type
  SupportedClass = UMyCustomAsset::StaticClass();
  // this factory will create a new instance, rather than drag & drop
  bCreateNew = true; 
  //Is a drag n drop import factory
  bEditorImport = true;
}

// Check also the other possible virtual create functions that you can use,
// like FactoryCreateBinary for instance:

UObject* UMyAssetFactory::FactoryCreateFile(UClass* InClass, UObject* InParent, FName InName, EObjectFlags Flags, const FString& FileName ...)
{
  //example: parse text file to custom object field
  UMyCustomAsset * MyAsset = nullptr;
  FString TextString;
  
  if(FFileHelper::LoadFileToString(TextString, *FileName))
  {
    MyAsset = NewObject<UMyCustomAsset>(InParent,Class,Name,Flags);
    MyAsset.Text = FText::FromString(TextString);
  }
  return MyAsset;
}
```

5. Customize Assets with Asset actions

```c++
class FMyCustomAssetActions_Base : public FAssetTypeActions_Base
{

  FMyCustomAssetActions_Base(const TSharedRef<ISlateStyle>& InStyle); 

public:
  
	// IAssetTypeActions interface
	virtual FText GetName() const override;
	virtual FColor GetTypeColor() const override;
	virtual UClass* GetSupportedClass() const override;
	virtual uint32 GetCategories() override;
  virtual void GetActions(const TArray<UObject*>& InObjects, FMenuBuilder& MenuBuilder) override;
  virtual bool HasActions(const TArray<UObject*>& InObjects) override;
	virtual void OpenAssetEditor( const TArray<UObject*>& InObjects, TSharedPtr<class IToolkitHost> EditWithinLevelEditor = TSharedPtr<IToolkitHost>()) override;
  ...
	// End of IAssetTypeActions interface
 
 .cpp example
 
void FMyCustomAssetActions_Base::GetActions(const TArray<UObject*>& InObjects, FMenuBuilder& MenuBuilder)
{
  // note: can't call Super::GetActions(...) because this is a interface derived class, not a uobject
  FAssetTypeActions_Base::GetActions((InObjects, Section);
  
  // We need this conversion so we make sure our selected objects reference is not deleted by GC meanwhile
  auto MyAssets = GetTypedWeakObjectPtrs<UMyCustomAsset>(InObjects);
  
  MenuBuilder.AddMenuEntry(
    LOCTEXT("MyCustomAsset_ReverseText", "Reverse Text"),
    LOCTEXT("MyCustomAsset_ReverseTextToolTip", "Reverse Text Tooltip text"),
    FSlateIcon(),
    FUIAction(
      FExecuteAction::CreateLambda([=]{
        // do stuff with MyAssets
      }),
      FCanExecuteAction::CreateLambda([=]{
        //if the action is enabled on menu
        //do stuff return value
        return false;
      })
    )
  );
}
```
6. Register on the Module your actions and styles

```c++
virtual void StartupModule() override
{
  RegisterAssetTools(); // an example of a function
}

virtual ShutdownModule() override
{
  UnRegisterAassetTools();
}

void RegisterAssetTools()
{
  // this module maintains a registry of all the asset tools that are knowned in the editor
  IAssetTools& AssetTools = FModuleManager::LoadModuleChecked<FAssetToolsModule>("AssetTools").Get();

// you can register your own asset category : optional
  const auto MyAssetCategory = AssetTools.RegisterAdvancedAssetCategory(FName(TEXT("MyAssets")), LOCTEXT("MyAssetCategory", "My Assets"));
  
  const auto MyAction = MakeShareable(new FMyCustomAssetActions_Base(MyAssetCategory));
	AssetTools.RegisterAssetTypeActions(MyAction);
	CreatedAssetTypeActions.Add(MyAction); // our property
  ...
}

void UnRegisterAssetTools()
{
  // AssetTools should be loaded at this point
	if (FModuleManager::Get().IsModuleLoaded("AssetTools"))
	{
		IAssetTools& AssetTools = FModuleManager::GetModuleChecked<FAssetToolsModule>("AssetTools").Get();
		for (const auto AssetTypeAction : CreatedAssetTypeActions)
		{
			AssetTools.UnregisterAssetTypeActions(AssetTypeAction.ToSharedRef());
		}

		CreatedAssetTypeActions.Empty();
	}
}
```
7. Custom Editor UI for Asset
  - By default, Unreal engine will create a default property inspector for your custom asset, based on your uproperties that are visible/editable.
  - If you want a different UI, you can create your own Slate UI, like this example:

```c++
class SMyButton : public SCompoundWidget
{
public:
  SLATE_BEGIN_ARGS(SMyButton)
  {}
    // The label to display on the button
    SLATE_ATTRIBUTE(FText, Text)

    // Called when the button is clicked
    SLATE_EVENT(FOnClicked, OnClicked)
  SLATE_END_ARGS()

  void Construct(const FArguments& InArgs);
};

//.cpp

void SMyButton::Construct(const FArguments& InArgs)
{
  ChildSlot
  [
    SNew(SButton)
      .OnClicked(InArgs._OnClicked)
      [
        SNew(STextBlock)
          .Font(FMyStyle::GetFontStyle("TextButtonFont"))
          .Text(InArgs._Text)
          .ToolTip(LOCTEXT("MyButtonToolTip", "Click me!"))
      ];
  ];
}
```

8. Apply your custom UI slate (override additional function in FMyCustomAssetActions_Base):
```c++
  //Override on your FMyCustomAssetActions_Base
virtual void OpenAssetEditor( const TArray<UObject*>& InObjects, TSharedPtr<class IToolkitHost> EditWithinLevelEditor = TSharedPtr<IToolkitHost>()) override
{
  const EToolkitMode::Type Mode = EditWithinLevelEditor.IsValid() ? EToolkitMode::WorldCentric : EToolkitMode::Standalone;

    for (auto ObjIt = InObjects.CreateConstIterator(); ObjIt; ++ObjIt)
    {
      if (UMyCustomAsset* CustomAsset = Cast<UMyCustomAsset>(*ObjIt))
      {
        TSharedRef<FCustomAssetEditor> EditorToolkit = MakeShareable(new FCustomAssetEditor(Style));
        EditorToolkit->Initialize(CustomAsset, Mode, EditWithinLevelEditor);
      }
    }
  }

```

## Custom Editor Tabs
Tab can be registered at StartupModule()
```c++
 //Research: Engine\UE4\Source\Editor\ContentBrowser\Private\ContentBrowserSingleton.cpp:56
 
 void StartupModule()
 {
  ...
	FGlobalTabmanager::Get()->RegisterNomadTabSpawner( TabID, FOnSpawnTab::CreateRaw(this, &FContentBrowserSingleton::SpawnContentBrowserTab, BrowserIdx) )
	.SetDisplayName(DefaultDisplayName)
	.SetTooltipText( LOCTEXT( "ContentBrowserMenuTooltipText", "Open a Content Browser tab." ) )
	.SetGroup( ContentBrowserGroup )
	.SetIcon(ContentBrowserIcon);
}
...
void ShutdownModule()
{
  ...
  FGlobalTabmanager::Get()->UnregisterNomadTabSpawner(TabID);
}
```

#### TIP : Custom Asset thumbnails
> Thumbnails can be any UI widget, including 3D Viewports
> Search the Engine codebase for uses of `UThumbnailRenderer`, `UDefaultSizedThumbnailRenderer` 


#### TIP : Widget Reflector
> You can use this editor tool to checkout the Slate code for any widget visible on screen, including jumping directly for the code where each slate component was created.(*Window -> Developer Tools -> Widget Reflector*)

#### TIP : Unreal Message Box System (Message Dialog Pop-up, MessageBox)
> Unreal uses custom popup message boxes that can be called using:
> ```c++
> FMessageDialog::Open(EAppMsgType::Ok, EAppReturnType::Type::Ok, FText::FromString(*OutErrorMessage));
> ```
> If we want a more custom modal windows solutions. There are a number of options check :
> https://docs.unrealengine.com/5.0/en-US/API/Editor/UnrealEd/Dialogs/

#### TIP : Unreal Notification Message System (Bottom-Right notification messages)
> Unreal uses custom Notification messages using:
> ```c++
> NotificationItem = FSlateNotificationManager::Get().AddNotification(Info);
> if (auto NotificationPtr = NotificationItem.Pin())
> {
>	NotificationPtr->SetFadeOutDuration(0.5f);
>	NotificationPtr->SetExpireDuration(40.0f);
>	NotificationPtr->SetCompletionState(SNotificationItem::CS_Pending);
>	NotificationPtr->SetHyperlink(FSimpleDelegate::CreateLambda([]() { FGlobalTabmanager::Get()->TryInvokeTab(FName("OutputLog")); }),
>	FText::FromString("Show Output Log"));
> }
> ```

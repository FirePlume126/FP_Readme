
# 项目框架

* **项目未开源**
* **所有模块都支持多人联机游戏**

## 作者信息
Copyright FirePlume, All Rights Reserved. Email: fireplume@126.com
 
作者网址：
[Bilibili](https://space.bilibili.com/395084718)、
[YouTube](https://www.youtube.com/@FirePlume126)、
[GitHub](https://www.github.com/FirePlume126)

## 框架目录

**框架目录定义**

- 类别
	- 插件名称
		- 模块名称(只有一个模块且与插件同名时，不显示此行)

* Utility：此类插件是存放工具的插件
	- [FPUtility](#fputility)：工具函数库

- Common：此类插件包含通用基础功能
	- [FPCommon](#fpcommon)：包含通用基础功能的插件
		- [FPInput](#fpcommon-fpinput)：管理输入(仅用于C++)
		- [FPUI](#fpcommon-fpui)：管理用户界面
		- [FPLoadingScreen](#fpcommon-fploadingscreen)：游戏启动或地图转换期间的加载界面

* System：此类插件为独立系统，可根据游戏类型选择使用
	- [FPMovementSystem](#fpmovementsystem)
		- [FPMovementSystem](#fpmovementsystem-fpmovementsystem)：角色基础运动系统
		- [FPMovementSystemEditor](#fpmovementsystem-fpmovementsystemeditor)：用来存放动画修改器
	- [FPOnlineSystem](#fponlinesystem)：管理服务器和会话，处理服务器玩家存档并生成玩家
	- [FPAbilitySystem](#fpabilitysystem)：管理玩家属性、能力和技能组合

- Misc：此类插件包含简单游戏功能和用于存储资产
	- [FPFeatures](#fpfeatures)：包含简单的游戏功能模块
		- [FPInteraction](#fpfeatures-fpinteraction)：交互
		- [FPInventory](#fpfeatures-fpinventory)：库存
	- [FPAssets](#fpassets)：用于存储资产
	
* [Non self-made assets](#non-self-made-assets)：非自制资产，除此之外的**美术**和所有**程序**均为自制

---

<a name="fputility"></a>
## FPUtility

工具函数库

`UFPUtilityFunctionLibrary`由函数模板、工具函数和数学函数组成

<a name="fpcommon"></a>
## FPCommon

包含通用基础功能的插件

<a name="fpcommon-fpInput"></a>
### FPInput

管理输入(仅用于C++)，绑定输入、更改绑定按键。
`UFPInputComponent`是该模块的主要类，它继承于`UEnhancedInputComponent`，在`UFPInputComponent`中管理绑定的游戏标签、触发状态和委托，重复绑定时(`UInputAction`和`ETriggerEvent`相同)，会覆盖旧的绑定(旧的绑定会移除)。

* **使用指南**

1、在项目设置中把`DefaultInputComponentClass`的默认值设置为`UFPInputComponent`

2、**FPInput**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|DefaultInputConfig|`FFPInputConfig`|在`APlayerController`中绑定输入时，会自动调用`DefaultInputConfig`，生命周期和`APlayerController`一样，不建议手动解绑|
|AbilityInputConfig|`FFPInputConfig`|在`APawn`中绑定输入时，会自动调用`AbilityInputConfig`|
|Context|`UInputMappingContext`|输入映射上下文|
|Priority|`int32`|优先级|
|InputActions|`TMap<FGameplayTag, UInputAction*>`|`UInputAction`要和`Context`绑定|

![FPInputSettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPInputSettings.png)

3、调用`UFPInputFunctionLibrary`的函数绑定输入、更改按键绑定，按键绑定保存在“GameUserSettings.ini”

```c++
// 绑定输入映射上下文，重复绑定时，会忽略新绑定
// @param InActor 输入APlayerController调用DefaultInputConfig，输入输入APawn调用AbilityInputConfig
static void AddInputMappings(AActor* InActor);

// 解除输入映射上下文
// @param InActor 输入APlayerController调用DefaultInputConfig，输入输入APawn调用AbilityInputConfig
static void RemoveInputMappings(AActor* InActor);

// 绑定输入动作，重复绑定时(InputTag和TriggerEvent相同)，会覆盖旧的绑定(旧的绑定会移除)
// @param InputTag 输入标签需要在项目设置DefaultInputConfig或AbilityInputConfig中设置
// @param TriggerEvent 触发状态
// @param Object 绑定函数的对象类
// @param Func 绑定的函数
template<class UserClass, typename FuncType>
static void BindInputAction(const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserClass* Object, FuncType Func);

// 移除能力输入已绑定的动作
static void RemoveAbilityInputActionBinding(APawn* InPawn, const FGameplayTag& InInputTag, ETriggerEvent InTriggerEvent);

// 解除所有能力输入动作绑定
static void RemoveAbilityBinds(APawn* InPawn);

// 更改多个绑定，并应用保存
// @param InMapping FName是映射名称，FKey是对应的按键
UFUNCTION(BlueprintCallable, Category = "FPInput")
static void ChangeBindings(TMap<FName, FKey> InMappings);

// 通过映射名称获取按键
UFUNCTION(BlueprintPure, Category = "FPInput")
static FKey GetKeyForName(const FName& InMappingName);

// 通过按键获取映射名称，可以用来检测按键有没有被使用
static TArray<FName> GetMappingNamesForKey(const FKey& InKey);
```

<a name="fpcommon-fpui"></a>
### FPUI

管理用户界面，此模块参考[Lyra Starter Game](https://www.unrealengine.com/marketplace/product/lyra)，用四个堆栈管理控件，聚焦控件时会设置输入模式和显示鼠标光标

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPUIHUD|管理控件，把`W_UILayoutManager`添加到视口，用来添加移除控件|
|FPUILayoutManager|UI布局管理：用来管理UCommonActivatableWidgetContainerBase堆栈控件|
|W_UILayoutManager|继承`FPUILayoutManager`的控件，添加了四个堆栈控件|
|FPUIUserWidget|继承于`UCommonActivatableWidget`，聚焦`FPUIUserWidget`控件时，会设置输入模式和显示鼠标光标，默认游戏输入模式且不显示鼠标。可以通过`InputMode`设置控件的输入模式 ![FPUI_InputMode](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPUI_InputMode.png)|
|FPUIMenuWidget|继承于`FPUIUserWidget`，默认UI输入模式且显示鼠标，可以被Escape按键关闭|

* **使用指南**

1、跟据[CommonUI官方文档](https://dev.epicgames.com/documentation/unreal-engine/common-ui-quickstart-guide-for-unreal-engine)设置，已在插件中创建了`BP_CommonInputData`和`BP_CommonInput_KeyboardMouse`

`Engine`>`GeneralSettings`>`DefaultClasses`>`GameViewportClientClass`设置为`CommonGameViewportClient`；
`Game`>`CommonInputSettings`>`InputData`设置为`BP_CommonInputData`；
`Game`>`CommonInputSettings`>`PlatformInput`>`Windows`>`Default`>`ControllerData`设置为`BP_CommonInput_KeyboardMouse`

2、创建控件，建议继承`FPUIUserWidget`类。聚焦`FPUIUserWidget`控件时，会设置输入模式和显示鼠标光标

3、在**FPUI**项目设置中添加控件

|UI层堆栈|描述|
|:-:|:-:|
|GameLayerWidgetClasss|游戏层控件类：HUD控件、背景控件|
|GameMenuLayerWidgetClasss|游戏菜单层控件类：与游戏菜单相关控件，如库存控件|
|MenuLayerWidgetClasss|菜单层控件类：设置控件|
|ModalLayerWidgetClasss|模态层控件类：对话框|

![FPUISettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPUISettings.png)

4、继承`FPUIHUD`创建自己的HUD，并添加给`AGameModeBase::HUDClass`

5、添加移除控件
![FPUI_Widget](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPUI_Widget.png)

```c++
// 添加控件
// @param InSlotTag 控件对应的标签，在项目设置FPUI中设置
// @return 如果控件已添加，会返回旧控件指针
UFUNCTION(BlueprintCallable, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static UCommonActivatableWidget* AddWidget(const UObject* WorldContextObject, UPARAM(meta = (Categories = "FPUI.Slot")) FGameplayTag InSlotTag);

// 移除控件
// @param InSlotTag 控件对应的标签，在项目设置FPUI中设置
UFUNCTION(BlueprintCallable, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static void RemoveWidget(const UObject* WorldContextObject, UPARAM(meta = (Categories = "FPUI.Slot")) FGameplayTag InSlotTag);

// 移除所有控件
UFUNCTION(BlueprintCallable, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static void RemoveAllWidget(const UObject* WorldContextObject);

// 查找已添加到屏幕的控件
UFUNCTION(BlueprintPure, BlueprintCosmetic, meta = (WorldContext = "WorldContextObject"), Category = "FPUI")
static UCommonActivatableWidget* FindWidget(const UObject* WorldContextObject, UPARAM(meta = (Categories = "FPUI.Slot")) FGameplayTag InSlotTag);
```

<a name="fpcommon-fploadingscreen"></a>
### FPLoadingScreen

游戏启动或转换地图期间的加载界面，该模块基于`MoviePlayer`。
在`MoviePlayer`基础上添加了Slate，层级由低到高分别是：影片，背景控件，加载控件，提示框控件，渐变控件

* **使用指南**

1、开启`bStartupLoadingScree`仅在首次打开游戏时显示一次，对`StartupLoadingScreen`进行设置；开启`bSwitchMapLoadingScreen`，每次切换地图时都会显示，对`SwitchMapLoadingScreen`进行设置。				
![FPLoadingScreen](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen.png)

2、设置影片：将影片文件复制到“Content/Movies/...”文件夹中，然后将影片路径填入`MoviePaths`。
![FPLoadingScreen_Movie](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen_Movie.png)

3、设置背景控件：开启`bShowBackgroundWidget`启动背景控件，背景控件主要是由`SBorder`和`SImage`组成。`SBorder`用来设置背景颜色，`SImage`可以设置图片的位置和大小，`Images`如果添加多张图片，会随机获取图片。
![FPLoadingScreen_Background](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen_Background.png)

4、设置加载控件：开启`bShowLoadingWidget`启动加载控件，加载控件提供了三种样式，分别是`SCircularThrobber`、`SThrobber`和图像序列，将图像序列的图像添加到`Images`中，由`SImage`播放。
![FPLoadingScreen_Loading](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen_Loading.png)

5、设置提示控件：开启`bShowTipWidget`启动提示控件，提示控件主要是由`SBorder`和`STextBlock`组成。
![FPLoadingScreen_Tip](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen_Tip.png)

6、设置渐变控件：开启`bShowGradientWidget`启动渐变控件，渐变控件会在加载结束后启动，先让屏幕变黑，然后缓慢让屏幕变量。`BlackScreenTime`为黑屏时间，`GradientTime`为渐变时间。
![FPLoadingScreen_Gradient](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen_Gradient.png)

<a name="fpmovementsystem"></a>
## FPMovementSystem

此插件参考[Advanced Locomotion System V4](https://www.unrealengine.com/marketplace/product/advanced-locomotion-system-v1)做的C++插件，支持网络同步。
我对这个插件并不满意，本意想做一个支持所有骨架的运动系统。我尝试把**状态机**写在**动画蓝图模板**中，然后通过**动画层接口**调用**动画层**，但**动画蓝图模板**不支持**混合配置**。
目前插件只支持我自己的骨架，如果需要更换骨架，更换“Content\SkeletalMesh”文件夹中资产(骨架、重定向动画)。最后修改ABP_Character的骨架和动画。

<a name="fpmovementsystem-fpmovementsystem"></a>
### FPMovementSystem

角色基础运动系统

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPMovementComponent|运动系统组件，仅支持添加给`ACharacter`，会对`ACharacter`做默认设置，包括添加`ABP_Character`，计算运动数据并进行网络同步|
|FPMovementAnimInstance|运动系统动画实例，计算控制状态机的数据|
|ABP_Character|继承`UFPMovementAnimInstance`的动画蓝图，处理状态机|
|FPMovementPlayerCameraManager|玩家摄像机管理器，通过动画曲线设置摄像机位置旋转|
|FPMovementAnimInstance_Camera|摄像机动画实例，通过`UFPMovementComponent`获取玩家状态|
|ABP_Camera|继承`FPMovementAnimInstance_Camera`的动画蓝图，通过玩家状态设置动画曲线|

* **使用指南**

1、给`ACharacter`添加`FPMovementComponent`组件。

2、对`ACharacter`的以下同名函数进行重载，并调用`FPMovementComponent`的这几个函数。
```c++
// 组件初始化前调用，在ACharacter::PreInitializeComponents()中调用此函数初始化
void PreInitializeComponents();

// 移动模式变化时，设置移动状态
void OnMovementModeChanged(EMovementMode NewPrevMovementMode, uint8 InPreviousCustomMode = 0);

// 开始蹲伏，设置站立蹲伏姿势
void OnStartCrouch(float InHalfHeightAdjust, float InScaledHalfHeightAdjust);

// 开始蹲伏，设置站立蹲伏姿势
void OnEndCrouch(float InHalfHeightAdjust, float InScaledHalfHeightAdjust);

// 着路时
void Landed(const FHitResult& InHit);

// 跳跃时	
void OnJumped();
```

3、调用`FPMovementComponent`的输入事件。
```c++
// 移动
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Move(const FInputActionValue& InValue);

// 查看
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Look(const FInputActionValue& InValue);

// 行走
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Walk();

// 蹲伏
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Crouching();

// 跳，bIsMantleCheck=true时可以检测攀爬
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Jump();

// 停止跳，建议直接使用ACharacter::StopJumping
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void StopJump();

// 冲刺
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Sprint();

// 停止冲刺
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void StopSprint();

// 瞄准
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Aim();

// 停止瞄准
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void StopAim();

// 翻滚
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Roll();
```

4、调用`FPMovementComponent`的设置事件，为了方便在.ini保存本地设置，我将一些枚举参数改成了int32，可以直接调用去掉Event_的函数，输入枚举参数。
```c++
// 设置鼠标灵敏度
// @param InTurnSensitivity 转弯灵敏度
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetTurnSensitivity(float InTurnSensitivity);

// 设置左肩摄像机
// @param bInRightShoulder 为true时摄像机在左肩；为false时在右肩
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetRightShoulderCamera(bool bInRightShoulder);

// 设置第一人称视角
// @param bInFirstPerson 为true时切换第一人称视角；为false时切换第三人称视角
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetFirstPersonCamera(bool bInFirstPerson);

// 通过整数设置旋转模式
// @param InRotationModeInt 0、查看方向，1、速度方向
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetRotationModeFromInt(int32 InRotationModeInt);

// 设置移动模型
// @param InModelNum 0、正常，1、敏捷，2、迟缓
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetMovementModel(int32 InModelNum);

// 设置重叠状态
// @param InOverlayState 重叠状态
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetOverlayState(EFPMovementOverlayState InOverlayState);

// 设置布娃娃状态
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetRagdollState(bool bInRagdoll);

// 设置速度比例
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetSpeedRatio(float InRatio);

// 设置缩放比例
// @param InScaleTime 执行缩放的时间
// @param InRatioX X轴的缩放比例
// @param InRatioY Y轴的缩放比例，等于0时Y=X
// @param InRatioZ Z轴的缩放比例，等于0时Z=X
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetScaleRatio(float InScaleTime, float InRatioX, float InRatioY = 0.0f, float InRatioZ = 0.0f);

// 设置跳高比例
UFUNCTION(BlueprintCallable, Category = "FPMovement|Event")
void Event_SetJumpHeightRatio(float InRatio);
```

5、设置摄像机管理器，在`APlayerController`中添加`AFPMovementPlayerCameraManager`
```c++
// .h

virtual void OnPossess(APawn* NewPawn) override;

virtual void OnRep_Pawn() override;

// 玩家控制的APawn时，会在服务器和客户端执行
void OnPossessedPawn(APawn* NewPawn);

// .cpp

AMyPlayerController::AMyPlayerController
{
	PlayerCameraManagerClass = AFPMovementPlayerCameraManager::StaticClass();
}

void AMyPlayerController::OnPossess(APawn* NewPawn)
{
	Super::OnPossess(NewPawn);

	if (!IsRunningDedicatedServer())
	{
		OnPossessedPawn(NewPawn);
	}
}

void AMyPlayerController::OnRep_Pawn()
{
	Super::OnRep_Pawn();

	OnPossessedPawn(GetPawn());
}

void AMyPlayerController::OnPossessedPawn(APawn* NewPawn)
{
	if (AFPMovementPlayerCameraManager* CameraManager = Cast<AFPMovementPlayerCameraManager>(PlayerCameraManager))
	{
		CameraManager->OnPossess(NewPawn);
	}
}
```

<a name="fpmovementsystem-fpmovementsystemeditor"></a>
### FPMovementSystemEditor

用来存放动画修改器

<a name="fponlinesystem"></a>
## FPOnlineSystem

管理服务器和会话，处理服务器玩家存档并生成玩家

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPOnlineManagerSubsystem|联机管理子系统：专用服务器创建会话和地图，客户端读取玩家ID和名称|
|FPOnlineServerSubsystem|服务器子系统：管理服务器数据，读取服务器玩家存档|
|FPOnlineSessionsSubsystem|会话子系统：管理会话|
|FPOnlinePlayerState|玩家状态：处理服务器玩家存档并生成玩家|
|FPOnlineDedicatedServerSettings|专用服务器设置，保存在ServerSettings.ini中|

* **存档和配置文件的读取保存时机**

|名称|功能|执行端|读取时机|保存时机|
|:-:|:-:|:-:|:-:|:-:|
|UserSettings.sav|保存玩家信息，主要包括玩家ID和名称，玩家ID不支持修改|客户端|游戏启动时|修改时|
|ServerSettings.sav|保存服务器设置，主要包括玩家信息列表和服务器时间|服务器|创建服务器|玩家退出游戏、退出服务器、周期性保存|
|Player_"玩家ID".sav|保存玩家游戏数据(类如：位置、属性等)，每个玩家对应一个存档|服务器|玩家加入游戏|更换`Pawn`(除了加入游戏更换`Pawn`)、玩家退出游戏、周期性保存|
|ServerSettings.ini|专用服务器配置文件|服务器|创建专用服务器|手动修改|

专用服务器配置文件：ServerSettings.ini
```ini
[ServerSettings]
bHasServerSettings=True
bServerCreateSession=True
ServerSaveCycle=600

[SessionsSettings]
ServerName=None
MapName=None
Password=
bDedicatedServer=True
bIsLANMatch=False
NumPublicConnections=10
NumPrivateConnections=0
BuildUniqueId=1
bAllowJoinInProgress=True
bAllowJoinViaPresence=True
bShouldAdvertise=True
bUsesPresence=False
bUseLobbiesIfAvailable=False
bUsesStats=False
bUseLobbiesVoiceChatIfAvailable=False
bAllowJoinViaPresenceFriendsOnly=False
bAllowInvites=True
bAntiCheatProtected=False
bStartAfterCreate=False
```

* **流程图**

1、专用服务器创建会话和地图，客户端读取玩家ID和名称
![FPOnlineSystem_StartGame](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPOnlineSystem_StartGame.png)

2、读取保存服务器存档和服务器玩家存档，服务器玩家存档类需要在项目设置中添加
![FPOnlineSystem_SaveGame](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPOnlineSystem_SaveGame.png)

* **使用指南**

1、**FPOnlineSystem**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|MainMenu|`TSoftObjectPtr<UWorld>`|主菜单地图|
|Maps|`TArray<TSoftObjectPtr<UWorld>>`|可以切换的地图，专用服务器可以通过ServerSettings.ini设置要切换的地图名称|
|ServerSaveCycle|`float`|服务器保存周期，专用服务器在ServerSettings.ini中设置|
|SavePlayerInfoClass|`TSubclassOf<USaveGame>`|保存玩家信息的类，服务器用来保存玩家存档|
|DefaultChangePawnClass|`TSubclassOf<APawn>`|加入游戏要更换的APawn类；AGameModeBase::DefaultPawnClass会失效，建议把DefaultPawnClass改为APawn|

![FPOnlineSystemSettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPOnlineSystemSettings.png)

2、继承`AFPOnlinePlayerState`创建自己的玩家状态重写以下函数，并添加给`AGameModeBase::PlayerStateClass`
```c++
// .h

// 加载玩家状态
virtual void LoadPlayerState(USaveGame* NewSaveObject) override;

// 保存玩家状态
virtual void SavePlayerState(USaveGame* NewSaveObject) override;

// UI提示禁止玩家加入游戏，不重写此函数玩家会直接退出游戏，没有拉黑提示
virtual void BanPlayerJoinGameForUI() override;

// .cpp

void AMyPlayerState::LoadPlayerState(USaveGame* NewSaveObject)
{
	//这只是一个案例，根据自己要保存的数据添加
	UFPSaveGame_PlayerInfo* SavePlayerInfo = Cast<UFPSaveGame_PlayerInfo>(NewSaveObject);
	if (!SavePlayerInfo)
	{
		return;
	}

	CharacterTransform = SavePlayerInfo->CharacterTransform;
}

void AMyPlayerState::SavePlayerState(USaveGame* NewSaveObject)
{
	Super::SavePlayerState(NewSaveObject);

	//这只是一个案例，根据自己要保存的数据添加
	UFPSaveGame_PlayerInfo* SavePlayerInfo = Cast<UFPSaveGame_PlayerInfo>(NewSaveObject);
	if (!SavePlayerInfo)
	{
		return;
	}

	SavePlayerInfo->CharacterTransform = CharacterTransform;
}

void AMyPlayerState::BanPlayerJoinGameForUI()
{
	// 在UMG中提示玩家被拉黑，延迟几秒后调用UFPOnlineFunctionLibrary::OpenMainMenu退出游戏
	UFPUIFunctionLibrary::AddWidget(this, FPGameplayTags::UITag_ServerBanPlayer);
}
```

3、在`AGameModeBase`初始化服务器，在`APlayerController::OnUnPossess`中调用`AFPOnlinePlayerState::WriteSavePlayerData`。完成这些，存档功能就可以正常使用

```c++
void AMyGameModeBase::InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage)
{
	Super::InitGame(MapName, Options, ErrorMessage);
	UFPOnlineFunctionLibrary::InitServer(this);
}

void AMyGameModeBase::Logout(AController* Exiting)
{
	if (APlayerController* PlayerController = Cast<APlayerController>(Exiting))
	{
		UFPOnlineFunctionLibrary::PlayerExitGame(this, PlayerController);
	}

	Super::Logout(Exiting);
}

void AMyPlayerController::OnUnPossess()
{
	if (AMyPlayerState* OnlinePlayerState = GetPlayerState<AMyPlayerState>())
	{
		//这个函数中做了判断，只会在服务器执行
		OnlinePlayerState->WriteSavePlayerData();
	}

	Super::OnUnPossess();
}

```

4、调用UFPOnlineFunctionLibrary的函数切换地图或调用子系统功能，这些函数可以根据情况使用

```c++
//====================================切换地图====================================>>

// 获取地图URL
// @param InMapName 需要在UFPOnlineProjectSettings::Maps设置才可以获取URL
UFUNCTION(BlueprintPure, Category = "FPOnline")
static FString GetMapURL(const FString& InMapName);

// 获取所有地图名称，需要在UFPOnlineProjectSettings::Maps设置
UFUNCTION(BlueprintPure, Category = "FPOnline")
static TArray<FString> GetAllMapName();

// 获取地图名称
UFUNCTION(BlueprintPure, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static FString GetMapName(const UObject* WorldContextObject);

// 设置地图名称，保存在硬盘，重启游戏默认选择的地图
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void SetMapName(const UObject* WorldContextObject, const FString& NewMapName);

// 打开服务器地图
// @param InMapName 需要在UFPOnlineProjectSettings::Maps设置才可以打开地图
// @param bInOnline 多人联机游戏输入true
// @return 成功打开服务器地图返回true
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static bool OpenServerLevel(const UObject* WorldContextObject, const FString& InMapName, bool bInOnline = false);

// 打开地图
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void CallOpenLevel(const UObject* WorldContextObject, const FString& InAddress);

// 客户端旅行
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void CallClientTravel(const UObject* WorldContextObject, const FString& InAddress);

// 打开主菜单地图，联机游戏会先删除会话
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void OpenMainMenu(const UObject* WorldContextObject);

//====================================子系统功能====================================>>

// 初始化服务器：在AGameModeBase::InitGame()中调用
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void InitServer(const UObject* WorldContextObject);

// 玩家离开游戏，在AGameModeBase::Logout()中调用
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void PlayerExitGame(const UObject* WorldContextObject, APlayerController* InPlayer);

// 玩家升级时更新
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void UpdateWhenUpLevelPlayer(const UObject* WorldContextObject, int32 InPlayerId, int32 InLevel);

// 拉黑或取消拉黑玩家时更新
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void UpdateWhenBannedPlayer(const UObject* WorldContextObject, int32 InPlayerId, bool bInHasBan);

// 删除玩家存档时更新
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void UpdateWhenRemovePlayer(const UObject* WorldContextObject, int32 InPlayerId);

// 获取玩家信息列表
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static TArray<FFPOnlinePlayersInfo> GetPlayersInfoList(const UObject* WorldContextObject);

// 获取在线玩家控制器列表
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static TArray<APlayerController*> GetPlayerControllerList(const UObject* WorldContextObject);

// 设置服务器时间，仅用来作弊
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void SetServerTimeSeconds(const UObject* WorldContextObject, int32 InTime);

// 获取服务器时间
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static int32 GetServerTimeSeconds(const UObject* WorldContextObject);

// 设置玩家ID：本地调用，可以通过APlayerState获取玩家ID
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void SetPlayerId(const UObject* WorldContextObject, int32 NewPlayerId);

// 设置玩家名称：本地调用，可以通过APlayerState获取玩家名称
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void SetPlayerName(const UObject* WorldContextObject, const FString& NewPlayerName);

// 获取网络模式
UFUNCTION(BlueprintPure, meta = (WorldContext = "WorldContextObject", DisplayName = "GetNetMode"), Category = "FPOnline")
static EFPOnlineNetMode K2_GetNetMode(const UObject* WorldContextObject);
```

5、会话子系统可以创建会话、搜索会话、加入会话、删除会话、开始会话和更新会话。调用这些函数需要绑定委托

```c++
// 创建会话
// @param NewCreateSessionData 创建会话的数据
void CreateSession(const FFPOnlineCreateSessionData& NewCreateSessionData);

// 搜索会话
// @param InMaxSearchResults 最大搜索结果
// @param bInDedicatedServer 仅搜索专用服务器的时候输入true
void FindSessions(int32 InMaxSearchResults, bool bInDedicatedServer);

// 加入会话
// @param InSessionResult 搜索会话返回的结果
void JoinSession(const FOnlineSessionSearchResult& InSessionResult);

// 删除会话
void DestroySession();

// 开始会话
void StartSession();

// 更新会话
// @param NewCreateSessionData 更新会话的数据
void UpdateSession(const FFPOnlineCreateSessionData& NewCreateSessionData);

// 完成以上事件调用的委托
FFPOnlineOnCreateSessionComplete OnlineOnCreateSessionComplete;
FFPOnlineOnFindSessionsComplete OnlineOnFindSessionsComplete;
FFPOnlineOnJoinSessionComplete OnlineOnJoinSessionComplete;
FFPOnlineOnDestroySessionComplete OnlineOnDestroySessionComplete;
FFPOnlineOnStartSessionComplete OnlineOnStartSessionComplete;
FFPOnlineOnUpdateSessionComplete OnlineOnUpdateSessionComplete;
```

6、没联机的时候(开始游戏界面)，玩家状态不建议继承`AFPOnlinePlayerState`，这时候如果要获取ID和名称，可以通过`FPOnlineManagerSubsystem`直接获取。也可以设置给玩家状态后，再通过玩家状态获取

```c++
void AMyPlayerState::BeginPlay()
{
	Super::BeginPlay();
	if (UFPOnlineManagerSubsystem* OnlineManager = GetGameInstance()->GetSubsystem<UFPOnlineManagerSubsystem>())
	{
		SetPlayerId(OnlineManager->GetPlayerId());
		SetPlayerName(OnlineManager->GetPlayerName());
	}	
}
```

7、调用`AFPOnlinePlayerState`的函数添加聊天消息功能

```c++
// 发送聊天信息，本地调用
UFUNCTION(BlueprintCallable)
void SendChatMessage(const FFPOnlineMessageData& InMessageData);

// 接收聊天消息，重写后把消息添加到UI
UFUNCTION()
virtual void ReceiveChatMessage(const FFPOnlineMessageData& InMessageDat);
```

<a name="fpabilitysystem"></a>
## FPAbilitySystem

管理玩家属性、能力和技能组合

<a name="fpfeatures"></a>
## FPFeatures

包含简单的游戏功能模块

<a name="fpfeatures-fpinteraction"></a>
### FPInteraction

交互

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPInteractionComponent|添加此组件的`APawn`可使用互动功能|
|FPInteractionInterface|互动的目标必须继承此接口|
|FPInteractionWidget|显示互动提示的控件继承此类|
|FPInteractionActor|互动的目标Actor继承此类，或者添加`FPInteractionInterface`接口|

<a name="fpfeatures-fpinventory"></a>
### FPInventory

库存系统

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPInventoryComponent|库存组件：添加此组件的`AActor`可存放物品|
|FPInventoryBackpackComponent|背包组件：主要用来同步操作到服务器，可以操作自己的背包和场景中的物品箱，添加建议给`APlayerState`|
|FPInventoryItemBase|物品基类|
|FPInventoryItemChest|物品箱|

* **使用指南**

1、给`APlayerState`添加`FPInventoryBackpackComponent`组件

2、物品箱继承`FPInventoryItemChest`或者给`AActor`添加`FPInventoryComponent`组件

3、继承`FFPInventoryItemInfo`创建物品的数据表格
![FPInventory_DataTable](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPInventory_DataTable.png)

4、将创建好的数据表格添加到`FPInventory`项目设置
![FPInventorySettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPInventorySettings.png)

5、调用`FPInventoryBackpackComponent`的函数使用库存系统
```c++
// 移动物品，在本地调用
// @param InSourceInventoryComp 源库存组件
// @param InSourceIndex 源库存物品索引
// @param InTargetIndex 此库存物品索引，InTargetIndex=-1时，自动分配位置
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void LocalTransferSlots(UFPInventoryComponent* InSourceInventoryComp, int32 InSourceIndex, int32 InTargetIndex = -1, UFPInventoryComponent* InTargetInventoryComp = nullptr);

// 拆分物品，在本地调用
// @param InIndex 物品位置索引
// @param InQuantity 拆分数量
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void LocalSplitStack(int32 InIndex, int32 InQuantity, UFPInventoryComponent* InTargetInventoryComp = nullptr);

// 丢弃物品到世界，在本地调用
// @param InIndex 物品位置索引
// @param InQuantity 输入-1时，全部丢弃
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void LocalDropItemToWorld(int32 InIndex, int32 InQuantity = -1, UFPInventoryComponent* InTargetInventoryComp = nullptr);

// 整理物品
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void LocalSortItems(UFPInventoryComponent* InTargetInventoryComp = nullptr);
```

<a name="fpassets"></a>
## FPAssets

用于存储资产

* 字体：[Source Han Sans](https://www.github.com/adobe-fonts/source-han-sans)

<a name="non-self-made-assets"></a>
## Non self-made assets

非自制资产，除此之外的**美术**和所有**程序**均为自制

|资产类型|数量|资产来源|
|:-:|:-:|:-:|
|字体|1|[Source Han Sans](https://www.github.com/adobe-fonts/source-han-sans)|
|动画序列|93|[Advanced Locomotion System V4](https://www.unrealengine.com/marketplace/product/advanced-locomotion-system-v1)|
|音频_脚步|16|[Advanced Locomotion System V4](https://www.unrealengine.com/marketplace/product/advanced-locomotion-system-v1)|
|图标_鼠标|9|[Lyra Starter Game](https://www.unrealengine.com/marketplace/product/lyra)|

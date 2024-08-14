
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
`UFPInputComponent`是该模块的主要类，它继承于`UEnhancedInputComponent`，在`UFPInputComponent`中管理`FGameplayTag`和输入委托，重复绑定时(`UInputAction`和`ETriggerEvent`和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)。

* **使用指南**

1、在项目设置中把`DefaultInputComponentClass`的默认值设置为`UFPInputComponent`

2、**FPInput**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|DefaultInputConfig|`FFPInputConfig`|在`APlayerController`中绑定输入时，会自动调用`DefaultInputConfig`，生命周期和`APlayerController`相同|
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

// 绑定输入动作，重复绑定时(InputTag、TriggerEvent和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)
// @param InInputComponent 输入组件，APlayerController的输入组件会调用DefaultInputConfig，否则调用AbilityInputConfig
// @param InputTag 输入标签需要在项目设置DefaultInputConfig或AbilityInputConfig中设置
// @param TriggerEvent 触发状态
// @param Object 绑定函数的类
// @param Func 绑定的函数
template<class UserClass, typename FuncType>
static uint32 BindInputAction(UInputComponent* InInputComponent, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserClass* Object, FuncType Func);

// 绑定GAS输入，建议在ACS中调用，重复绑定时(InputTag、TriggerEvent和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)
// @param InInputComponent 输入组件，APlayerController的输入组件会调用DefaultInputConfig，否则调用AbilityInputConfig
// @param InputTag 输入标签需要在项目设置AbilityInputConfig中设置
// @param TriggerEvent 触发器事件
// @param Object 绑定函数的类，建议在ACS中调用
// @param Func 绑定的函数
// @param AbilitySpecHandle 能力规格句柄
// @param bActivate 为true是激活能力，为false时关闭能力
template<class UserClass, typename FuncType>
static uint32 BindGASInput(UInputComponent* InInputComponent, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserClass* Object, FuncType Func, const FGameplayAbilitySpecHandle AbilitySpecHandle, bool bActivate);

// 解除能力绑定
static void RemoveAbilityBind(UInputComponent* InInputComponent, uint32 InBindHandle);

// 解除所以能力绑定
static void RemoveAllAbilityBinds(UInputComponent* InInputComponent);

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

* **调试指令**

|指令|描述|
|:-:|:-:|
|FP.Movement.Debug.Camera (bool)|调试玩家摄像机|
|FP.Movement.Debug.ShowDebugShapes (bool)|调试视角方向、速度方向、按键方向、角色朝向|
|FP.Movement.Debug.CharacterStates (bool)|调试角色状态|
|FP.Movement.Debug.AnimCurves (bool)|调试动画曲线|
|FP.Movement.Debug.Mantle (bool)|调试玩家攀爬|
|FP.Movement.Debug.FootIK (bool)|调试脚部IK|
|FP.Movement.Debug.LandPrediction (bool)|调试坠落地面预测|

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

// 可以跳
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
bool CanJump() const;

// 跳，当bIsMantleCheck=true时可以检测攀爬
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void Jump();

// 停止跳，建议直接使用ACharacter::StopJumping
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
void StopJump();

// 可以冲刺
UFUNCTION(BlueprintCallable, Category = "FPMovement|InputEvent")
bool CanSprint() const;

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

3、在`AGameModeBase`初始化服务器，在`APlayerController::OnUnPossess()`中调用`AFPOnlinePlayerState::WriteSavePlayerData()`。完成这些，存档功能就可以正常使用

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

// 玩家或AI死亡时调用
// @param RespawnDelay 复活时间，等于0时立即复活
UFUNCTION(BlueprintCallable, meta = (WorldContext = "WorldContextObject"), Category = "FPOnline")
static void PawnDied(const UObject* WorldContextObject, AController* InController, float InRespawnDelay = 0.0f);

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

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPAbilitySystemComponent|能力系统组件，玩家添加给`APlayerState`，AI添加给`APawn`|
|FPAbilityManagerComponent|能力管理组件，添加给`APawn`。<br>服务器通过[能力属性配置](#fpabilitysystem-attributeconfig)和**玩家存档**初始化属性和能力，本地玩家通过[能力属性配置](#fpabilitysystem-attributeconfig)自动绑定按键输入，也可以手动[绑定能力输入](#fpabilitysystem-abilityinput)。<br>优先检测`APlayerState`的`UFPAbilitySystemComponent`(玩家)，<br>不存在时检测`APawn`的`UFPAbilitySystemComponent`(AI)|
|FPAbilityBase|能力基类，可以通过`FGameplayTag`读取[能力模型](#fpabilitysystem-abilitymodel)来构造能力|
|FPAttributeSetBase|属性集基类，请继承此类创建自己的属性。为了配合`FPAbilityManagerComponent`使用，玩家添加给`APlayerState`，AI添加给`Pawn`|
|FPAttributeSet|我使用的[属性集](#fpabilitysystem-attributeset)，仅供参考。请继承`FPAttributeSetBase`|
|FPAbilityCheatManager|作弊管理器。基于`FPAttributeSet`仅供参考，用来管理[调试指令](#fpabilitysystem-debug)，添加给`APlayerController`|

<a name="fpabilitysystem-abilitymodel"></a>
* **能力模型**

1、继承`FFPAbilityData`创建**能力模型**的数据表格，并把数据表格添加给`FPAbilitySystem`的项目设置

```c++
// 能力数据
USTRUCT(BlueprintType)
struct FFPAbilityData : public FTableRowBase
{
	GENERATED_BODY()

public:

#if WITH_EDITOR

	virtual void OnDataTableChanged(const UDataTable* InDataTable, const FName InRowName) override;

#endif

	// 能力标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPAbility.Ability"))
	FGameplayTag Tag = FGameplayTag::EmptyTag;

	// 能力类
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TSoftClassPtr<UFPAbilityBase> Class = nullptr;

	// 标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	FFPAbilityTags Tags;

	// 冷却
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	FFPAbilityCooldown Cooldown;

	// 消耗
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TMap<FGameplayAttribute, FScalableFloat> Costs;

	// 伤害
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	FFPAbilityDamage Damage;

	// 图标
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TSoftObjectPtr<UTexture2D> Icon = nullptr;

private:

	// 上次能力标签
	UPROPERTY()
	FGameplayTag LastTag = FGameplayTag::EmptyTag;
};
```

2、C++中使用**能力模型**必须在构造函数中调用函数`UFPAbilityBase::ConstructAbility()`；<br>
蓝图中使用可以通过修改变量`UFPAbilityBase::AbilityTag`读取**能力模型**，修改数据表格时也会同步修改蓝图，需要手动保存(我尝试通过UEditorAssetLibrary::SaveAsset()保存设置，但未打开蓝图时，会导致数据表格绑定委托未执行)

```c++
private:

#if WITH_EDITORONLY_DATA

	// 能力标签
	UPROPERTY(EditDefaultsOnly, meta = (Categories = "FPAbility.Ability"), Category = "FPAbility|Tags")
	FGameplayTag AbilityTag = FGameplayTag::EmptyTag;
#endif

protected:

#if WITH_EDITOR

	// 编辑更改属性后触发
	virtual void PostEditChangeProperty(struct FPropertyChangedEvent& PropertyChangedEvent) override;

	// 蓝图构造能力
	virtual void OnConstructAbility();

#endif

	// 构造能力，C++中使用能力模型必须在构造函数中调用此函数
	void ConstructAbility(const FGameplayTag& InAbilityTag);
```

FPAbilityDamageCalculation 能力[伤害计算](#fpabilitysystem-damagecalculation)

<a name="fpabilitysystem-attributeconfig"></a>
* **能力属性配置**

1、继承`FFPAttributeConfig`创建**能力属性配置**的数据表格，并把数据表格添加给`FPAbilitySystem`的项目设置

```c++
// 能力属性配置
USTRUCT(BlueprintType)
struct FFPAbilityAttributeConfig : public FTableRowBase
{
	GENERATED_BODY()

public:

	// 默认属性
	UPROPERTY(EditDefaultsOnly)
	TMap<FGameplayAttribute, FGameplayEffectModifierMagnitude> DefaultAttributes;

	// 默认应用的效果
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TArray<TSubclassOf<UGameplayEffect>> DefaultEffects;

	//  能力输入映射，<能力标签，能力输入>
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPAbility.Ability.Active"))
	TMap<FGameplayTag, FFPAbilityInputs> AbilityInputMap;
};
```

2、给`APawn`添加`UFPAbilityManagerComponent`组件，然后通过**能力属性配置**的**行命名**选择对应的配置，此`APawn`会根据选择的配置进行初始化

// 添加图片/////////////////////////

3、调用`UFPAbilityManagerComponent`的函数`ServerInitComp()`和`ClientInitComp()`初始化组件，绑定委托执行死亡逻辑，<br>
如果使用了[FPOnlineSystem](#fponlinesystem)插件，则调用UFPOnlineFunctionLibrary::PawnDied()重生。

```c++
// 服务器初始化组件，建议在APawn::PossessedBy中调用
// @param InSaveAttributes 属性存档数据，读取在服务器保存的属性数据，如果直接使用UFPAttributeSet，提供UFPAbilityFunctionLibrary::SaveDataToAttributeData()转换
void ServerInitComp(TMap<FGameplayAttribute, FScalableFloat> InSaveAttributes);

// 客户端初始化组件，建议在APawn::OnRep_PlayerState中调用
void ClientInitComp();

// 死亡委托，绑定委托后执行执行自己的逻辑，比如死亡动画或者布娃娃，如果使用FPOnlineSystem调用UFPOnlineFunctionLibrary::PawnDied()重生
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityDiedDelegate DiedDelegate;
```

4、服务器读取**能力属性配置**的`DefaultAttributes`和`DefaultEffects`初始化属性。死亡会添加死亡标签用来判断是否为复活，复活仅恢复属性(血量、耐力和能量)，不重复初始化

// 添加图片/////////////////////////

5、服务器读取**能力属性配置**`AbilityInputMap`的**能力标签**，**能力标签**读取[能力模型](#fpabilitysystem-abilitymodel)初始化能力，在死亡时会移除添加的能力

// 添加图片/////////////////////////

<a name="fpabilitysystem-abilityinput"></a>
6、本地调用`UFPAbilityManagerComponent`的函数`BindAbilityInput`，通过读取**能力属性配置**的`AbilityInputMap`自动绑定能力输入，死亡时移除所有能力输入；
`AbilityInputMap`添加`"FPAbility.Ability.Active.ConfirmCancel"`会自动绑定能力的确认和取消输入；
也可以通过调用`UFPAbilitySystemComponent`的函数`BindAbilityInput`绑定输入

```c++
// UFPAbilityManagerComponent的函数
// 绑定能力输入，建议在APawn::SetupPlayerInputComponent中调用
void BindAbilityInput();

// UFPAbilitySystemComponent的函数
// 绑定能力输入
void BindAbilityInput(TSubclassOf<UGameplayAbility> InAbilityClass, const FFPAbilityInputs& InAbilityInputs);
```

7、`UFPAbilityManagerComponent`还提供了一些小功能，比如：眩晕会取消主动能力、被攻击时根据攻击方向播放不同的动画、显示伤害数字的接口

<a name="fpabilitysystem-attributeset"></a>
* **属性集(仅供参考)**

|属性|描述|数据来源|网络复制|
|:-:|:-:|:-:|:-:|
|Level|等级，升级会提供属性点|存档数据|是|
|MaxLevel|最大等级|项目设置|否|
|XP|杀死敌人获取的经验值|存档数据|是|
|XPBaseValue|角色1级升级时所需的经验|项目设置|否|
|MaxXP|当前等级升级时所需的经验<br>$MaxXP=XPBaseValue \times Level^2$|计算|是|
|XPBounty|角色死亡时奖励给杀手的经验值 = $XPBounty \times Level$|数据表格|是|
|Points|加点剩余的属性点<br>可以加点的属性包括：血量、耐力、能量、速度和力量<br>已经使用的属性点 + $Points = LevelUpPoints \times Level$|存档数据|是|
|LevelUpPoints|升级提供的属性点|项目设置|否|
|MaxPoints|单个属性的最大点数|项目设置|否|
|Health|当前血量|存档数据|是|
|MaxHealth|血量最大值，当最大血量变化时，当前血量保持百分比不变<br>$MaxHealth=HealthInitial+HealthPerPoints\times HealthPoints$|计算|是|
|HealthInitial|血量初始值|数据表格|是|
|HealthRegenRate|血量恢复率，恢复血量会消耗能量|数据表格|是|
|HealthPoints|血量属性点|存档数据|是|
|HealthPerPoints|每个属性点添加的血量值|数据表格|是|
|Stamina|当前耐力|存档数据|是|
|MaxStamina|耐力最大值，当最大耐力变化时，当前耐力保持百分比不变<br>$MaxStamina=StaminaInitial+StaminaPerPoints\times StaminaPoints$|计算|是|
|StaminaInitial|耐力初始值|数据表格|是|
|StaminaRegenRate|耐力恢复率，恢复耐力会消耗能量|数据表格|是|
|StaminaPoints|耐力属性点|存档数据|是|
|StaminaPerPoints|每个属性点添加的耐力值|数据表格|是|
|Energy|当前能量，恢复血量和耐力会额外消耗能量|存档数据|是|
|MaxEnergy|能量最大值，当最大耐力变化时，当前能量保持不变(百分比改变)<br>$MaxEnergy=EnergyInitial+EnergyPerPoints\times EnergyPoints$|计算|是|
|EnergyInitial|能量初始值|数据表格|是|
|EnergyUseRate|能量消耗率，角色活着会持续消耗能量；没有能量时，持续消耗血量|数据表格|是|
|EnergyPoints|能量属性点|存档数据|是|
|EnergyPerPoints|每个属性点添加的能量值|数据表格|是|
|Speed|影响所有行为的速度，按百分比增速Speed%<br>$Speed=100+SpeedPerPoints\times SpeedPoints$|计算|是|
|SpeedPoints|速度属性点|存档数据|是|
|SpeedPerPoints|每个属性点添加的速度值|数据表格|是|
|Strength|力量，按百分比增伤Strength%<br>$Strength=100+StrengthPerPoints\times StrengthPoints$|计算|是|
|StrengthPoints|力量属性点|存档数据|是|
|StrengthPerPoints|每个属性点添加的力量值|数据表格|是|
|WeaponDamage|武器伤害|装备|是|
|Armor|护甲|装备|是|
|IncreaseDamage|百分比增伤，取值范围[-1, +∞]|Buff|是|
|ReduceDamage|百分比减伤，取值范围[-∞, 1]|Buff|是|
|Damage|受到的伤害，仅用于[伤害计算](#fpabilitysystem-damagecalculation)|计算|否|
|MaxWeight|最大负重|数据表格|是|

<a name="fpabilitysystem-damagecalculation"></a>
* **伤害计算**

未减伤前伤害(UnmitigatedDamage)；能力基础伤害(BaseDamage)；减伤后伤害/实际伤害(MitigatedDamage)。
$UnmitigatedDamage = (BaseDamage + WeaponDamage) \times \dfrac{Strength}{100} \times (1 + IncreaseDamage)$
$MitigatedDamage = UnmitigatedDamage\times \dfrac{100}{100 + Armor}\times (1 -ReduceDamage)$

<a name="fpabilitysystem-debug"></a>
* **调试指令**

|指令|描述|
|:-:|:-:|
|FP_God|状态全满|
|FP_FullState|状态一直全满|
|FP_NoCost|能力无消耗|
|FP_NoCooldown|能力无冷却|
|FP_AddXP (float)|增加经验|
|FP_AddHealth (float)|增加血量|
|FP_AddStamina (float)|增加耐力|
|FP_AddEnergy (float)|增加能量|
|FP_AddSpeed (float)|增加速度|
|FP_AddStrength (float)|增加力量|
|FP_AddWeaponDamage (float)|增加武器伤害|
|FP_AddArmor (float)|增加护甲|
|FP_AddIncreaseDamage (float)|增加百分比增伤|
|FP_AddReduceDamage (float)|增加百分比减伤|

* **使用指南**

1、给`APlayerState`或`APawn`添加`UFPAttributeSet`和`UFPAbilitySystemComponent`，并继承`IAbilitySystemInterface`接口，重写接口的函数`GetAbilitySystemComponent`

```c++
AMyPlayerState::AMyPlayerState()
{
	AttributeSet = CreateDefaultSubobject<UFPAttributeSet>(TEXT("AttributeSet"));
	AbilitySystemComponent = CreateDefaultSubobject<UFPAbilitySystemComponent>(TEXT("AbilitySystemComponent"));

	// AbilitySystemComponent的ReplicationMode默认为Mixed；AI使用时添加给APawn时，AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Minimal);
}

void AMyPlayerState::GetAbilitySystemComponent() const
{
	return AbilitySystemComponent;
}
```

2、使用[能力模型](#fpabilitysystem-abilitymodel)和[能力属性配置](#fpabilitysystem-attributeconfig)

3、如果不直接使用`FPAttributeSet`，而是使用它的子类，必须重写`FPAttributeSet`的函数`InitaAttributeSaveData`

```c++
// 初始化属性存档数据
UFUNCTION(BlueprintCallable, Category = "Attribute")
virtual void InitaAttributeSaveData(TMap<FGameplayAttribute, FScalableFloat> InSaveAttributes);

// 添加到属性的属性点，初始化调用
UFUNCTION(BlueprintCallable, Category = "Points")
virtual bool AddAttributePoints(const FGameplayAttribute& InAttribute, int32 NewValue);

// 恢复属性至满值，复活时调用
UFUNCTION(BlueprintCallable, Category = "Attribute")
virtual void RestoreFullAttribute();

// 应用受到到伤害
virtual void ApplyDamage(const FGameplayEffectModCallbackData& Data);
```

4、根据情况调用`UFPAbilityFunctionLibrary`的函数

```c++
// 查找能力数据，必须在项目设置添加AbilityModel
// @param InAbilityTag 能力标签
// @param OutAbilityData 返回能力数据
// @return 成功查找返回true
UFUNCTION(BlueprintPure, meta = (DisplayName = "FindAbilityData"), Category = "FPAbility")
static bool K2_FindAbilityData(UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag, FFPAbilityData& OutAbilityData);

// 查找能力数据，必须在项目设置添加AbilityModel
// @param InAbilityTag 能力标签
static FFPAbilityData* FindAbilityData(const FGameplayTag& InAbilityTag);

// 属性点添加到属性，加点时本地UI调用
// @param InPlayerController 本地的玩家控制器
// @param InAttribute 把对应的属性点加1点
UFUNCTION(BlueprintCallable, Category = "FPAbility|Points")
static void PointsAddToAttributePoints(APlayerController* InPlayerController, const FGameplayAttribute& InAttribute);

// 保存数据转属性数据，用来保存读取UFPAttributeSet的属性，如果创建了其他属性集不要使用这个函数
static TMap<FGameplayAttribute, FScalableFloat> SaveDataToAttributeData(FFPAttributeSaveData InSaveData);

// 属性数据转保存数据，用来保存读取UFPAttributeSet的属性，如果创建了其他属性集不要使用这个函数
static FFPAttributeSaveData AttributeDataToSaveData(TMap<FGameplayAttribute, FScalableFloat> InSaveAttributes);
```

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
|FPInteractionWidget|显示互动提示的控件继承此类，并添加给`FPInteraction`项目设置|
|FPInteractionActor|互动的目标Actor继承此类，或者添加`FPInteractionInterface`接口|

* **调试指令**

|指令|描述|
|:-:|:-:|
|FP.Interaction.Debug.Interact (bool)|启用交互组件的调试|

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

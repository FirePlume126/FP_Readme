
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
	- [FPAbilitySystem](#fpabilitysystem)
		- [FPAbilitySystem](#fpabilitysystem-fpabilitysystem)：管理角色属性和能力
		- [FPAbilityCombo](#fpabilitysystem-fpabilitycombo)：能力组合
		- [FPAbilityComboEditor](#fpabilitysystem-fpabilitycomboeditor)：能力组合编辑器

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

// 通过绑定数据解除能力绑定
static void RemoveAbilityInputForBindingInfo(UInputComponent* InInputComponent, const FFPInputBindingInfo& InAction);

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
|SavePlayerDataClass|`TSubclassOf<USaveGame>`|保存玩家数据的类，服务器用来保存玩家存档|
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
	if (UFPSavePlayerData* SavePlayerData = Cast<UFPSavePlayerData>(NewSaveObject))
	{
		CharacterTransform = SavePlayerData->CharacterTransform;
	}	
}

void AMyPlayerState::SavePlayerState(USaveGame* NewSaveObject)
{
	Super::SavePlayerState(NewSaveObject);

	//这只是一个案例，根据自己要保存的数据添加
	if (UFPSavePlayerData* SavePlayerData = Cast<UFPSavePlayerData>(NewSaveObject))
	{
		SavePlayerData->CharacterTransform = CharacterTransform;
	}	
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

<a name="fpabilitysystem-fpabilitysystem"></a>
### FPAbilitySystem

管理角色属性和能力

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPAbilitySystemComponent|能力系统组件，玩家添加给`APlayerState`，AI添加给`APawn`|
|FPAbilityManagerComponent|能力管理组件，添加给`APawn`。<br>服务器通过[能力属性配置](#fpabilitysystem-attributeconfig)和**玩家存档**初始化属性和能力，本地玩家通过[能力属性配置](#fpabilitysystem-attributeconfig)自动绑定按键输入，也可以手动[绑定能力输入](#fpabilitysystem-abilityinput)。<br>优先检测`APlayerState`的`UFPAbilitySystemComponent`(玩家)，<br>不存在时检测`APawn`的`UFPAbilitySystemComponent`(AI)|
|FPAbilityBase|能力基类，能力添加到[能力模型](#fpabilitysystem-abilitymodel)统一管理|
|FPAttributeSetBase|属性集基类，继承此类创建自己的[属性集](#fpabilitysystem-attributeset)。为了配合`FPAbilityManagerComponent`使用，玩家添加给`APlayerState`，AI添加给`Pawn`|

* **FPAbilitySystem**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|AttributeConfig|`TSoftObjectPtr<UDataTable>`|[能力属性配置](#fpabilitysystem-attributeconfig)|
|CommonAbilityModel|`TSoftObjectPtr<UDataTable>`|通用[能力模型](#fpabilitysystem-abilitymodel)，管理所有Pawn都可以使用的能力|
|AbilityMapTable|`TSoftObjectPtr<UFPAbilityMapTable>`|[能力映射表](#fpabilitysystem-maptable)，管理所有能力映射，插件已设置好，不建议修改|
|CooldownGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|冷却GE类，所有能力都使用此GE，插件已设置好，不建议修改|
|CostGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|消耗GE类，，所有能力都使用此GE，插件已设置好，不建议修改|
|DamageGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|伤害GE类，，所有能力都使用此GE，插件已设置好，不建议修改|
|FullStateGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|状态一直全满GE类，控制台输入"FP_FullState (bool)"启动GE，根据自己的需求添加|

![FPAbilitySystemSettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilitySystemSettings.png)

<a name="fpabilitysystem-attributeconfig"></a>
* **能力属性配置**

**使用建议**：[能力属性配置](#fpabilitysystem-attributeconfig)数据表格的每一行对应一个骨骼或者角色，并对应一种[能力模型](#fpabilitysystem-abilitymodel)

1、继承`FFPAbilityAttributeConfig`创建**能力属性配置**的数据表格

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

	// 能力模型，管理能力数据
	UPROPERTY(config, EditAnywhere, meta = (RequiredAssetDataTags = "RowStructure=/Script/FPAbilitySystem.FPAbilityData"), Category = "Ability")
	TSoftObjectPtr<UDataTable> AbilityModel;

	// 能力的确认取消输入
	UPROPERTY(EditDefaultsOnly, Category = "Ability")
	FFPAbilityInput ConfirmCancelInput;

	// 主动技能，FFPAbilityInput不为空时绑定输入，<能力标签，能力输入>
	UPROPERTY(EditDefaultsOnly, meta = (Categories = "FPAbility.Ability.Active"), Category = "Ability")
	TMap<FGameplayTag, FFPAbilityInput> ActiveAbilities;

	// 被动技能
	UPROPERTY(EditDefaultsOnly, meta = (Categories = "FPAbility.Ability.Passive"), Category = "Ability")
	TArray<FGameplayTag> PassiveAbilities;
};
```

2、给`APawn`添加`UFPAbilityManagerComponent`组件，然后通过**能力属性配置**的**行命名**选择对应的配置，此`APawn`会根据选择的配置进行初始化

![FPAbilitySystem_AbilityManager](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilitySystem_AbilityManager.png)

3、服务器和客户端分别调用`UFPAbilityManagerComponent::ServerInitComp()`和`UFPAbilityManagerComponent::ClientInitComp()`初始化组件，绑定委托执行死亡逻辑<br>
如果使用了[FPOnlineSystem](#fponlinesystem)插件，则调用UFPOnlineFunctionLibrary::PawnDied()重生。

```c++
// 服务器初始化组件，建议在APawn::PossessedBy中调用
// @param InSaveAttributes 属性存档数据，读取在服务器保存的属性数据，如果直接使用
void ServerInitComp(TMap<FGameplayAttribute, FScalableFloat> InSaveAttributes);

// 客户端初始化组件，建议在APawn::OnRep_PlayerState中调用
void ClientInitComp();

// 接收伤害命中反应，源和目标都可接收到委托。可以播放被击反应的动画和声音，也用来显示伤害数字
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityReceiveDamageHitReactDelegate ReceiveDamageHitReactDelegate;

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FFPAbilityDiedDelegate, APawn*, Pawn);

// 死亡委托，绑定委托后执行执行自己的逻辑，比如死亡动画或者布娃娃，如果使用FPOnlineSystem调用UFPOnlineFunctionLibrary::PawnDied()重生
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityDiedDelegate DiedDelegate;
```

4、服务器读取**能力属性配置**的`DefaultAttributes`和`DefaultEffects`初始化属性。死亡会添加死亡标签用来判断是否为复活，复活仅恢复属性(血量、耐力和能量)，不重复初始化

5、添加[能力模型](#fpabilitysystem-abilitymodel)，管理此`APawn`的所有能力

6、服务器读取**能力属性配置**`ActiveAbilities`和`PassiveAbility`的**能力标签**，**能力标签**读取[能力模型](#fpabilitysystem-abilitymodel)初始化能力，在死亡时会移除添加的能力

```c++
// UFPAbilityManagerComponent的函数↓↓↓

// 移除所有能力，服务器调用
void RemoveAbilities();

// 给予能力，服务器调用
FGameplayAbilitySpecHandle GiveAbility(const FGameplayTag& InAbilityTag);

// 清除能力，服务器调用
void ClearAbility(const FGameplayTag& InAbilityTag);

// 查找能力类，优先查找能力模型，其次查找通用能力模型
TSoftClassPtr<UFPAbilityBase> FindAbilityClass(const FGameplayTag& InAbilityTag) const;
```

<a name="fpabilitysystem-abilityinput"></a>
6、本地调用`UFPAbilityManagerComponent::InitAbilityInput()`，通过读取**能力属性配置**的`ActiveAbilities`和`ConfirmCancelInput`自动绑定能力输入，死亡时移除所有能力输入；
也可以通过函数`UFPAbilityManagerComponent`的函数手动绑定输入。

```c++
// 能力输入
USTRUCT(BlueprintType)
struct FFPAbilityInput
{
	GENERATED_BODY()

public:

	// 激活能力输入标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPInput.Ability"))
	FGameplayTag ActivateInputTag;

	// 激活能力触发事件
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	ETriggerEvent ActivateTriggerEvent = ETriggerEvent::Started;

	// 取消能力输入标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPInput.Ability"))
	FGameplayTag CancelInputTag;

	// 取消能力触发事件
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	ETriggerEvent CancelTriggerEvent = ETriggerEvent::Started;

	// 是否有效
	FORCEINLINE bool IsValid() const
	{
		return ActivateInputTag.IsValid() || CancelInputTag.IsValid();
	}
};

// UFPAbilityManagerComponent的函数↓↓↓

// 初始化能力输入，建议在APawn::SetupPlayerInputComponent中调用
void InitAbilityInput();

// 移除所有能力输入，本地调用
void RemoveAllAbilityInput();

// 绑定能力输入，按键被使用时会自动解绑之前的能力，本地调用
void BindAbilityInput(const FGameplayTag& InAbilityTag, const FFPAbilityInput& InAbilityInput);

// 解除此按键绑定的技能，本地调用
void RemoveAbilityInput(const FFPAbilityInput& InAbilityInput);
```

<a name="fpabilitysystem-abilitymodel"></a>
* **能力模型**

**能力模型**用于管理配置能力，修改数据表格时会同步修改蓝图，数据表格保存时也会同步保存蓝图，同时也会更新保存[能力映射表](#fpabilitysystem-maptable)，从而影响C++能力构造。<br>
将能力模型添加给[能力属性配置](#fpabilitysystem-attributeconfig)就可以通过[能力属性配置](#fpabilitysystem-attributeconfig)数据表格的Key和能力的FGameplayTag查询能力(详见：[能力函数库](#fpabilitysystem-functionlibrary))

1、继承`FFPAbilityData`创建**能力模型**的数据表格

```c++
// 能力数据
USTRUCT(BlueprintType)
struct FFPAbilityData : public FTableRowBase
{
	GENERATED_BODY()

public:

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

#if WITH_EDITORONLY_DATA
private:

	// 能力类的路径名称(仅用于蓝图类)。为空时，代表数据表格此行没有改动
	FString ClassPathName = "";

	// 上次能力标签
	UPROPERTY()
	FGameplayTag LastTag = FGameplayTag::EmptyTag;
#endif

#if WITH_EDITOR
public:

	// 保存蓝图能力类。保存数据表格时，仅当ClassPathName不为空时保存能力类，保存完成后会清空ClassPathName
	void SaveBlueprintAsset();

private:

	// 数据表格改变时调用
	virtual void OnDataTableChanged(const UDataTable* InDataTable, const FName InRowName) override;

	// 仅当能力标签被修改时才会执行
	void DataTableTagChanged();
#endif
};
```

2、在**能力模型**添加能力标签容器

```c++
// 能力标签
USTRUCT(BlueprintType)
struct FFPAbilityTags
{
	GENERATED_BODY()

public:

	// 当此能力执行时，带有这些标签的能力将被取消
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer CancelAbilitiesWithTag;

	// 当此能力处于活动状态时，带有这些标签的能力将被阻止
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer BlockAbilitiesWithTag;

	// 当此能力处于活动状态时，应用到激活所有者的标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer ActivationOwnedTags;

	// 仅当激活的演员/组件拥有所有这些标签时，此能力才能被激活
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer ActivationRequiredTags;

	// 如果激活的演员/组件拥有这些标签中的任何一个，此能力将被阻止
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer ActivationBlockedTags;

	// 仅当源演员/组件拥有所有这些标签时，此能力才能被激活
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer SourceRequiredTags;

	// 如果源演员/组件拥有这些标签中的任何一个，此能力将被阻止
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer SourceBlockedTags;

	// 仅当目标演员/组件拥有所有这些标签时，此能力才能被激活 
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer TargetRequiredTags;

	// 如果目标演员/组件拥有这些标签中的任何一个，此能力将被阻止
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FGameplayTagContainer TargetBlockedTags;
};
```

3、在**能力模型**添加能力冷却和消耗，可以根据自己的[属性集](#fpabilitysystem-attributeset)添加消耗的属性

```c++
// 能力冷却
USTRUCT(BlueprintType)
struct FFPAbilityCooldown
{
	GENERATED_BODY()

public:

	// 冷却时间
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	FScalableFloat Duration;

	// 冷却标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPAbility.Cooldown"))
	FGameplayTag Tag = FGameplayTag::EmptyTag;
};

// 消耗
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
TMap<FGameplayAttribute, FScalableFloat> Costs;
```

4、在**能力模型**添加能力伤害，调用函数`UFPAbilityBase::MakeDamageGameplayEffectSpecHandles()`返回的`TArray<FGameplayEffectSpecHandle>`，给目标应用伤害调用函数`UFPAbilityFunctionLibrary::ApplyDamage()`<br>
**伤害类型标签**：标签为空时，为普通[伤害计算](#fpabilitysystem-damagecalculation)。插件提供了`百分比伤害`、`真实伤害`和`伤害反馈`,
添加`伤害反馈`标签后会播放受击动画、声音等，绑定委托实现`UFPAbilityManagerComponent::ReceiveDamageHitReactDelegate`<br>
**伤害给与目标的效果**：如着火、眩晕等
```c++
// 能力伤害
USTRUCT(BlueprintType)
struct FFPAbilityDamage
{
	GENERATED_BODY()

public:

	// 伤害数值
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (ClampMin = 0))
	float Value = 0.0f;

	// 伤害类型标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPAbility.Damage.Type"))
	FGameplayTagContainer TypeTags;

	// 伤害给与目标的效果
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TArray<TSubclassOf<UGameplayEffect>> GiveEffects;
};

// UFPAbilityBase的函数↓↓↓
// 生成伤害GE规格句柄
UFUNCTION(BlueprintCallable, Category = "FPAbility.Damage")
TArray<FGameplayEffectSpecHandle> MakeDamageGameplayEffectSpecHandles() const;

// UFPAbilityFunctionLibrary的函数↓↓↓
// 应用伤害
// @param InAvatarActor 目标AvatarActor
// @param InSpecHandles 伤害GE规格委托，索引0是伤害数值，其他是状态GE
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static TArray<FActiveGameplayEffectHandle> ApplyDamage(AActor* InAvatarActor, TArray<FGameplayEffectSpecHandle> InSpecHandles);

// UFPAbilityManagerComponent的委托↓↓↓
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FFPAbilityReceiveDamageHitReactDelegate, UFPAbilityManagerComponent*, Source, UFPAbilityManagerComponent*, Target, FHitResult, HitResult, float, Damage);

// 接收伤害命中反应，源和目标都可接收到委托。可以播放被击反应的动画和声音，也用来显示伤害数字
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityReceiveDamageHitReactDelegate ReceiveDamageHitReactDelegate;
```

<a name="fpabilitysystem-maptable"></a>
* **能力映射表**

管理所有能力映射，[能力属性配置](#fpabilitysystem-attributeconfig)或任何[能力模型](#fpabilitysystem-abilitymodel)保存时自动更新(不支持手动修改)<br>

1、**功能**<br>
(1)、通过游戏标签查询能力(每个Pwan除了可以使用自己专用的[能力模型](#fpabilitysystem-abilitymodel)还可以使用通用[能力模型](#fpabilitysystem-abilitymodel))；
(2)、[能力模型](#fpabilitysystem-abilitymodel)保存时，检测保存的[能力模型](#fpabilitysystem-abilitymodel)和其他[能力模型](#fpabilitysystem-abilitymodel)的能力是否有重复，有重复会弹窗提示；
(3)、用于构造C++能力(蓝图会自己更新保存)

2、**能力映射表**的数据资产继承的类

```c++
// 能力映射
USTRUCT(BlueprintType)
struct FFPAbilityMap
{
	GENERATED_BODY()

public:

	// 能力映射
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
	TMap<FGameplayTag, TSoftClassPtr<UFPAbilityBase>> Abilities;
};

// 能力模型映射
USTRUCT(BlueprintType)
struct FFPAbilityModelMap
{
	GENERATED_USTRUCT_BODY()

public:

	// 行句柄
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	FDataTableRowHandle RowHandle;

	// 是否为蓝图类
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	bool bIsBlueprintClass = false;
};

// 能力映射表
UCLASS()
class FPABILITYSYSTEM_API UFPAbilityMapTable : public UDataAsset
{
	GENERATED_BODY()

protected:

	// 能力映射，通过游戏标签查询能力
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, meta = (EditCondition = "false"), Category = "Config")
	TMap<FName, FFPAbilityMap> AbilityMaps;

	// 通用能力映射，所有Pawn都可以使用的能力
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, meta = (EditCondition = "false"), Category = "Config")
	FFPAbilityMap CommonAbilityMap;

	// 能力模型映射；1、能力模型保存时，检测所有能力模型的能力是否有重复；2、用于构造C++能力(蓝图会自己更新保存)
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, meta = (EditCondition = "false"), Category = "Config")
	TMap<FSoftClassPath, FFPAbilityModelMap> AbilityModelMaps;

public:

	// 查找能力映射
	// @param InKey UFPAbilityProjectSettings::AttributeConfig数据表格的Key
	const FFPAbilityMap* FindAbilityMap(const FName& InKey) const;

	// 获取通用能力映射
	const FFPAbilityMap* GetCommonAbilityMap() const;

	// 查找能力类
	// @param InKey UFPAbilityProjectSettings::AttributeConfig数据表格的Key
	// @param InAbilityTag 能力标签
	TSoftClassPtr<UFPAbilityBase> FindAbilityClass(const FName& InKey, const FGameplayTag& InAbilityTag) const;

	// 查找能力数据
	// @param InAbilityClass 能力类
	// @param bCppOnly bCppOnly = true时，仅获取C++能力的数据
	const FFPAbilityData* FindAbilityData(UFPAbilityBase* InAbilityClass, bool bCppOnly = true) const;

#if WITH_EDITOR

	// 添加能力映射
	// @param InDataTable 数据表格需继承FFPAbilityAttributeConfig
	void AddAbilityMaps(UDataTable* InDataTable);

	// 添加通用能力映射
	// @param InAbilityModel 能力模型，数据表格需继承FFPAbilityData
	void AddCommonAbilityMap(UDataTable* InAbilityModel);

	// 添加能力模型映射
	// @param InAbilityModel 能力模型，数据表格需继承FFPAbilityData
	void AddAbilityModelMap(UDataTable* InAbilityModel);
#endif
};
```

* **使用指南**

1、给`APlayerState`(玩家)或`APawn`(AI)添加`UFPAttributeSet`和`UFPAbilitySystemComponent`，并继承`IAbilitySystemInterface`接口，重写接口的函数`GetAbilitySystemComponent`

```c++
AMyPlayerState::AMyPlayerState()
{
	AttributeSet = CreateDefaultSubobject<UFPAttributeSet>(TEXT("AttributeSet"));
	AbilitySystemComponent = CreateDefaultSubobject<UFPAbilitySystemComponent>(TEXT("AbilitySystemComponent"));

	// AbilitySystemComponent的ReplicationMode默认为Mixed，AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Minimal);
}

void AMyPlayerState::GetAbilitySystemComponent() const
{
	return AbilitySystemComponent;
}
```

2、使用[能力模型](#fpabilitysystem-abilitymodel)和[能力属性配置](#fpabilitysystem-attributeconfig)

3、继承`FPAttributeSetBase`创建自己的属性集，可以根据情况重写以下函数

```c++
// 初始化属性存档数据
UFUNCTION(BlueprintCallable, Category = "Attribute")
virtual void InitaAttributeSaveData(TMap<FGameplayAttribute, FScalableFloat> InSaveAttributes) {};

// 恢复属性至满值，复活时调用
UFUNCTION(BlueprintCallable, Category = "Attribute")
virtual void RestoreFullAttribute() {};

// 添加到属性的属性点，初始化调用
UFUNCTION(BlueprintCallable, Category = "Points")
virtual bool AddAttributePoints(const FGameplayAttribute& InAttribute, int32 NewValue) { return false; };

// 应用受到到伤害
virtual void ApplyDamage(const FGameplayEffectModCallbackData& Data);
```

<a name="fpabilitysystem-functionlibrary"></a>
4、根据情况调用`UFPAbilityFunctionLibrary`的函数

```c++
// 移除所有能力，服务器调用
// @param InAvatarActor 目标AvatarActor
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static void RemoveAbilities(AActor* InAvatarActor);

// 给予能力，服务器调用
// @param InAvatarActor 目标AvatarActor
// @param InAbilityTag 能力标签
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static FGameplayAbilitySpecHandle GiveAbility(AActor* InAvatarActor, UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 清除能力，服务器调用
// @param InAvatarActor 目标AvatarActor
// @param InAbilityTag 能力标签
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static void ClearAbility(AActor* InAvatarActor, UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 查找能力类，优先查找AvatarActor的能力模型，其次查找通用能力模型
// @param InAvatarActor 目标AvatarActor
// @param InAbilityTag 能力标签
// @return 返回能力类
UFUNCTION(BlueprintPure, Category = "FPAbility")
static TSoftClassPtr<UFPAbilityBase> FindAbilityClass(AActor* InAvatarActor, UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 移除所有能力输入，本地调用
// @param InAvatarActor 目标AvatarActor
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static void RemoveAllAbilityInput(AActor* InAvatarActor);

// 绑定能力输入，按键被使用时会自动解绑之前的能力，本地调用
// @param InAvatarActor 目标AvatarActor
// @param InAbilityTag 能力标签
// @param InAbilityInputs 能力输入
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static void BindAbilityInput(AActor* InAvatarActor, UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag, const FFPAbilityInput& InAbilityInput);

// 解除此按键绑定的技能，本地调用
// @param InAvatarActor 目标AvatarActor
// @param InAbilityInputs 能力输入
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static void RemoveAbilityInput(AActor* InAvatarActor, const FFPAbilityInput& InAbilityInput);

// 应用伤害
// @param InAvatarActor 目标AvatarActor
// @param InSpecHandles 伤害GE规格委托，索引0是伤害数值，其他是状态GE
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static TArray<FActiveGameplayEffectHandle> ApplyDamage(AActor* InAvatarActor, TArray<FGameplayEffectSpecHandle> InSpecHandles);

// 获取命中反应方向
// @param InTarget 目标能力管理组件
// @param InImpactPoint 被攻击的位置
UFUNCTION(BlueprintPure, Category = "FPAbility|Damage")
static const FGameplayTag GetHitReactDirectionTag(UFPAbilityManagerComponent* InTarget, const FVector& InImpactPoint);

// 属性点添加到属性，加点时本地UI调用
// @param InPlayerController 本地的玩家控制器
// @param InAttribute 把对应的属性点加1点
UFUNCTION(BlueprintCallable, Category = "FPAbility|Points")
static void PointsAddToAttributePoints(APlayerController* InPlayerController, const FGameplayAttribute& InAttribute);
```

<a name="fpabilitysystem-attributeset"></a>
* **属性集**

**这是我自己项目中使用的属性集，非插件内容，仅供参考**

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
|FP_God|无敌|
|FP_FullState|状态一直全满(需要在项目设置配置自己的GE)|
|FP_NoCost|能力无消耗|
|FP_NoCooldown|能力无冷却|
|FP_AddXP (float)|增加经验|
|FP_AddHealth (float)|增加血量|
|FP_AddStrength (float)|增加力量|
|FP_AddWeaponDamage (float)|增加武器伤害|
|FP_AddArmor (float)|增加护甲|
|FP_AddIncreaseDamage (float)|增加百分比增伤|
|FP_AddReduceDamage (float)|增加百分比减伤|

以下是我自己项目使用的调试指令，非插件内容，仅供参考
|指令|描述|
|:-:|:-:|
|FP_AddStamina (float)|增加耐力|
|FP_AddEnergy (float)|增加能量|
|FP_AddSpeed (float)|增加速度|
|FP_AddMaxWeight (float)|增加速度|

<a name="fpabilitysystem-fpabilitycombo"></a>
### FPAbilityCombo

能力组合

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPAbilityTask_StartAbilityCombo|启动能力组合任务|
|FPAbilityCombo|能力组合，将动画组合成连击动作|
|FPAbilityComboNodeBase|能力组合节点基类|
|FPAbilityComboNodeTrans|能力组合节点转换，处理节点转换|
|FPAbilityComboNodeConduit|能力组合节点导管，建立分支|
|FPAbilityComboNodeAnimBase|能力组合动画节点基类|
|FPAbilityComboNodeEntry|能力组合开始节点|
|FPAbilityComboNodeAnim|能力组合动画节点，用来添加动画序列或动画蒙太奇，网络环境仅支持动画蒙太奇|
|FPAbilityComboNodeAnimBlueprint|能力组合动画蓝图节点|

<a name="fpabilitysystem-fpabilitycomboeditor"></a>
### FPAbilityComboEditor

能力组合编辑器

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPAbilityComboAssetTypeActions|能力组合资产类型操作，注册内容浏览器右键菜单的能力组合资产|
|FPAbilityComboAnimNodeAssetTypeActions|能力组合动画节点资产类型操作，注册内容浏览器右键菜单的能力组合资产|
|FPAbilityComboFactory|能力组合工厂|
|FPAbilityComboAnimNodeFactory|能力组合动画节点工厂|
|FPAbilityComboPanelNodeFactory|能力组合面板节点工厂|
|FPAbilityComboAssetEditor|能力组合编译器设置|
|FPAbilityComboPersonaBlueprintEditor|能力组合角色蓝图编辑器：可重复使用的角色功能，适用骨骼相关资产的资产编辑器|
|FPAbilityComboAnimAssetBrowser|能力组合动画资产浏览器|
|FPAbilityComboEditorToolbar|能力组合编辑器工具栏|
|FPAbilityComboEdGraph|能力组合编辑器图形|
|FPAbilityComboSchema|能力组合编辑器图形架构：定义和管理能力组合图形节点|
|FPAbilityComboApplicationMode|能力组合应用程序模式，创建此模式的选项卡|
|FPAbilityComboAnimAssetTabFactory|能力组合动画资产选项卡工厂|
|FPAbilityComboGraphViewportTabFactory|能力组合图形视口选项卡工厂|
|FPAbilityComboPropertyDetailsTabFactory|能力组合属性细节选项卡工厂|
|FPAbilityComboDebugger|能力组合调试器，编辑器播放时，可以在能力组合编辑器选择调试的对象进行调试|
|FPAbilityComboEdNodeBase|能力组合编辑器节点基类|
|FPAbilityComboEdNodeEntry|能力组合编辑器开始节点|
|FPAbilityComboEdNodeConduit|能力组合编辑器节点导管，建立分支|
|FPAbilityComboEdNodeTrans|能力组合编辑器节点转换|
|FPAbilityComboEdNodeAnim|能力组合编辑器动画节点|

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

|资产类型|数量|资产来源|使用资产模块|
|:-:|:-:|:-:|:-:|
|字体|1|[Source Han Sans](https://www.github.com/adobe-fonts/source-han-sans)|[FPAssets](#fpassets)|
|动画序列|93|[Advanced Locomotion System V4](https://www.unrealengine.com/marketplace/product/advanced-locomotion-system-v1)|[FPMovementSystem](#fpmovementsystem)|
|音频_脚步|16|[Advanced Locomotion System V4](https://www.unrealengine.com/marketplace/product/advanced-locomotion-system-v1)|[FPMovementSystem](#fpmovementsystem)|
|图标_鼠标|9|[Lyra Starter Game](https://www.unrealengine.com/marketplace/product/lyra)|[FPUI](#fpcommon-fpui)|

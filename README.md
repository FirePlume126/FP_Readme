
# 项目框架

* **项目未开源**
* **所有模块都支持多人联机游戏**

## 作者信息
Copyright FirePlume, All Rights Reserved. Email: fireplume@126.com
 
作者网址：
[Bilibili](https://space.bilibili.com/395084718)、
[YouTube](https://www.youtube.com/@FirePlume126)、
[GitHub](https://www.github.com/FirePlume126)

<a name="Directory"></a>
## 框架目录

**框架目录定义**

- 类别
	- 插件名称
		- 模块名称(只有一个模块且与插件同名时，不显示此行)

* Common：此类插件包含通用基础功能
	- [FPCommon](#fpcommon)：包含通用基础功能的插件
		- [FPInput](#fpcommon-fpinput)：管理输入(仅用于C++)
		- [FPUI](#fpcommon-fpui)：管理用户界面
		- [FPLoadingScreen](#fpcommon-fploadingscreen)：游戏启动或地图转换期间的加载界面

- System：此类插件为独立系统，可根据游戏类型选择使用
	- [FPOnlineSystem](https://www.github.com/FirePlume126/FP_OnlineSystem#fponlinesystem)：管理服务器和会话，加载/保存服务器存档和玩家存档并生成玩家
	- [FPAbilitySystem](#fpabilitysystem)
		- [FPAbilitySystem](#fpabilitysystem-fpabilitysystem)：能力属性管理框架
		- [FPAbilityCombo](#fpabilitysystem-fpabilitycombo)：能力组合(技能连招系统)
		- [FPAbilityComboEditor](#fpabilitysystem-fpabilitycomboeditor)：能力组合编辑器
	
* Misc：此类插件包含简单游戏功能和用于存储资产
	- [FPFeatures](#fpfeatures)：包含简单的游戏功能模块
		- [FPInteraction](#fpfeatures-fpinteraction)：交互
		- [FPInventory](#fpfeatures-fpinventory)：库存
	- [FPAssets](#fpassets)：用于存储资产
	
- Utility：此类插件是存放工具的插件
	- [FPEditorTools](#fpeditortools)：编辑器工具

* [Non self-made assets](#non-self-made-assets)：非自制资产，除此之外的**美术**和所有**程序**均为自制

---

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
// @param InActor 输入APlayerController调用DefaultInputConfig，输入其他AActor调用AbilityInputConfig
static void AddInputMappings(AActor* InActor);

// 解除输入映射上下文
// @param InActor 输入APlayerController调用DefaultInputConfig，输入其他AActor调用AbilityInputConfig
static void RemoveInputMappings(AActor* InActor);

// 绑定输入动作，重复绑定时(InputTag、TriggerEvent和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)
// @param InInputComponent 输入组件，APlayerController的输入组件会调用DefaultInputConfig，否则调用AbilityInputConfig
// @param InputTag 输入标签需要在项目设置DefaultInputConfig或AbilityInputConfig中设置
// @param TriggerEvent 触发状态
// @param Object 绑定函数的类
// @param Func 绑定的函数
template<class UserClass, typename FuncType>
static uint32 BindInputAction(UInputComponent* InInputComponent, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserClass* Object, FuncType Func);

// 绑定GAS输入，重复绑定时(InputTag、TriggerEvent和绑定函数的对象相同)，会覆盖旧的绑定(旧的绑定会移除)
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

// 查找默认输入动作
static const UInputAction* FindDefaultInputActionForTag(const FGameplayTag& InInputTag);

// 查找能力输入动作
static const UInputAction* FindAbilityInputActionForTag(const FGameplayTag& InInputTag);

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

管理用户界面，此模块参考[Lyra Starter Game](https://www.fab.com/listings/93faede1-4434-47c0-85f1-bf27c0820ad0)，用四个堆栈管理控件，聚焦控件时会设置输入模式和显示鼠标光标

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPUIManagerSubsystem|UI管理子系统，管理控件，把`W_UILayoutManager`添加到视口，用来添加移除控件|
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

4、添加移除控件
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

6、设置渐变控件：开启`bShowGradientWidget`启动渐变控件，渐变控件会在加载结束后启动，先让屏幕变黑，然后缓慢让屏幕变亮。`bAutoExecuteGradient`为自动执行渐变，为true时会自动渐变并销毁控件，为false时需要调用`UFPLoadingScreenFunctionLibrary::ExecuteLoadingScreenGradient()`才会执行渐变并销毁控件；`BlackScreenTime`为黑屏时间，`GradientTime`为渐变时间。
![FPLoadingScreen_Gradient](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPLoadingScreen_Gradient.png)

<a name="fpabilitysystem"></a>
## FPAbilitySystem

<a name="fpabilitysystem-fpabilitysystem"></a>
### FPAbilitySystem

能力属性管理框架，此模块基于GAS，输入基于[FPInput](#fpcommon-fpinput)

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPAbilityManagerComponent|能力管理组件，添加给`APawn`。<br>服务器通过[能力属性配置](#fpabilitysystem-attributeconfig)和**服务器存档**初始化属性和能力，本地通过[能力属性配置](#fpabilitysystem-attributeconfig)自动绑定按键输入，也可以通过[蓝图API](#fpabilitysystem-blueprintapi)手动添加激活能力和绑定输入|
|FPAbilitySystemComponent|能力系统组件，根据自己的游戏需求添加，详见[使用指南](#fpabilitysystem-usageguide)|
|FPAbilityBase|能力基类，能力添加到[能力模型](#fpabilitysystem-abilitymodel)统一管理|
|FPAbilityComboBase|运行[能力组合](#fpabilitysystem-fpabilitycombo)的能力基类，继承此类只需要完成基础设置就可以使用[能力组合](#fpabilitysystem-fpabilitycombo)|
|FPAbilityBTTask_RunAbility|运行[能力模型](#fpabilitysystem-abilitymodel)中能力的行为树任务，详见[AI激活能力](#fpabilitysystem-aiactivateability)|
|FPAttributeSetBase|属性集基类，继承此类创建自己的[属性集](#fpabilitysystem-attributeset)|

![FPAbilitySystem_Framework](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilitySystem_Framework.png)

* **FPAbilitySystem**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|AttributeConfig|`TSoftObjectPtr<UDataTable>`|[能力属性配置](#fpabilitysystem-attributeconfig)|
|AbilityMapTable|`TSoftObjectPtr<UFPAbilityMapTable>`|[能力映射表](#fpabilitysystem-maptable)，管理所有能力映射，插件已设置好，不建议修改|
|CooldownGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|冷却GE类，所有能力都使用此GE，插件已设置好，不建议修改|
|CostGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|消耗GE类，所有能力都使用此GE，插件已设置好，不建议修改|
|DamageGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|伤害GE类，所有能力都使用此GE，根据自己的需求更换计算逻辑|
|FullStateGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|状态一直全满GE类，控制台输入"FP_FullState (bool)"启动GE，根据自己的需求添加|

![FPAbilitySystemSettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilitySystemSettings.png)

<a name="fpabilitysystem-attributeconfig"></a>
* **能力属性配置**

**使用建议**：[能力属性配置](#fpabilitysystem-attributeconfig)数据表格的每一行对应一个骨骼或者角色

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

#if WITH_EDITORONLY_DATA

	// 能力模型，仅用于映射UFPAbilityMapTable::AbilityMaps，索引越大优先级越高(能力标签相同时，仅映射优先级最高的能力模型能力)
	UPROPERTY(EditDefaultsOnly, meta = (RequiredAssetDataTags = "RowStructure=/Script/FPAbilitySystem.FPAbilityData"), Category = "Ability")
	TArray<TSoftObjectPtr<UDataTable>> AbilityModels;
#endif

	// 能力的确认取消输入
	UPROPERTY(EditDefaultsOnly, Category = "Ability")
	FFPAbilityInput ConfirmCancelInput;

	// 主动能力，FFPAbilityInput不为空时绑定输入，<能力标签，能力输入>
	UPROPERTY(EditDefaultsOnly, meta = (Categories = "FPAbility.Ability.Active"), Category = "Ability")
	TMap<FGameplayTag, FFPAbilityInput> ActiveAbilities;

	// 被动能力标签容器
	UPROPERTY(EditDefaultsOnly, meta = (Categories = "FPAbility.Ability.Passive"), Category = "Ability")
	FGameplayTagContainer PassiveAbilities;
};
```

2、给`APawn`添加`UFPAbilityManagerComponent`组件，然后通过**能力属性配置**的**行命名**选择对应的配置，此`APawn`会根据选择的配置进行初始化

![FPAbilitySystem_AbilityManager](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilitySystem_AbilityManager.png)

3、服务器和客户端分别调用`UFPAbilityManagerComponent::ServerInitComp()`和`UFPAbilityManagerComponent::ClientInitComp()`初始化组件。<br>
绑定UFPAbilityManagerComponent::ReceivedDamageDelegate可以播放受击的动画和声音，也可以显示伤害数字，<br>
绑定UFPAbilityManagerComponent::DiedDelegate执行死亡逻辑，如果使用了[FPOnlineSystem](#fponlinesystem)插件，则调用UFPOnlineFunctionLibrary::PawnDied()重生

```c++
// 服务器初始化组件，建议在APawn::PossessedBy()中调用
// @param InAbilitySystemComp 能力系统组件
// @param NewServerData 服务器保存的数据
UFUNCTION(BlueprintCallable, Category = "FPAbility")
virtual void ServerInitComp(UFPAbilitySystemComponent* InAbilitySystemComp, const FFPAbilityServerData& NewServerData);

// 客户端初始化组件，建议在APawn::OnRep_PlayerState中()调用
// @param InAbilitySystemComp 能力系统组件
UFUNCTION(BlueprintCallable, Category = "FPAbility")
void ClientInitComp(UFPAbilitySystemComponent* InAbilitySystemComp);

// 接收伤害委托，源和目标的服务器和客户端都可接收到委托。攻击拥有"FPAbility.Damage.Type.NotHitReact"标签将不执行此委托，可以播放受击的动画和声音，也可以显示伤害数字
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityReceivedDamageDelegate ReceivedDamageDelegate;

// 死亡委托，绑定委托后执行死亡动画或者布娃娃，如果使用FPOnlineSystem调用UFPOnlineFunctionLibrary::PawnDied()重生
UPROPERTY(BlueprintAuthorityOnly, BlueprintAssignable, Category = "FPAbility")
FFPAbilityDiedDelegate DiedDelegate;
```

```c++
// 能力服务器数据
USTRUCT(BlueprintType)
struct FFPAbilityServerData
{
	GENERATED_BODY()

public:

	// 属性
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TMap<FGameplayAttribute, FScalableFloat> Attributes;
};

// 接收伤害数据
USTRUCT(BlueprintType)
struct FFPAbilityReceivedDamageData
{
	GENERATED_BODY()

public:

	// 伤害来源AvatarActor
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TObjectPtr<AActor> SourceAvatarActor;

	// 伤害目标AvatarActor
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TObjectPtr<AActor> TargetAvatarActor;

	// 命中结果
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	FHitResult HitResult;

	// 伤害数值
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (ClampMin = 0))
	float DamageValue = 0.0f;

	// 伤害类型标签
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPAbility.Damage.Type"))
	FGameplayTagContainer DamageTypeTags;
};
```

4、服务器读取**能力属性配置**的`DefaultAttributes`和`DefaultEffects`初始化属性。死亡会添加死亡标签用来判断是否为复活，复活仅恢复属性(需重写`FPAttributeSetBase::RestoreFullAttribute()`)，不重复初始化

5、添加[能力模型](#fpabilitysystem-abilitymodel)数组，管理此角色的所有能力，数组索引越大映射[能力映射表](#fpabilitysystem-maptable)的优先级越高(能力标签相同时，仅映射优先级最高的能力模型能力)

6、服务器读取**能力属性配置**`ActiveAbilities`和`PassiveAbility`的**能力标签**，**能力标签**读取[能力映射表](#fpabilitysystem-maptable)初始化能力，在死亡时会移除添加的能力

```c++
// UFPAbilityManagerComponent的函数↓↓↓

// 移除所有能力(仅支持服务器调用)
void RemoveAbilities();

// 给予能力(仅支持服务器调用)
void GiveAbility(const FGameplayTag& InAbilityTag);

// 清除能力(仅支持服务器调用)
void ClearAbility(const FGameplayTag& InAbilityTag);

// 激活能力(仅支持服务器调用)
void ActivateAbility(const FGameplayTag& InAbilityTag);

// 取消能力(仅支持服务器调用)
void CancelAbility(const FGameplayTag& InAbilityTag);

// 激活能力(仅支持服务器调用)，目前Ai行为树任务UFPAbilityBTTask_RunAbility在使用
bool ServerActivateAbility(const FGameplayTag& InAbilityTag, FPredictionKey InPredictionKey = FPredictionKey(), UGameplayAbility** OutInstancedAbility = nullptr, FOnGameplayAbilityEnded::FDelegate* OnGameplayAbilityEndedDelegate = nullptr, const FGameplayEventData* TriggerEventData = nullptr);

// 查找能力类，优先查找能力模型，其次查找通用能力模型
TSoftClassPtr<UFPAbilityBase> FindAbilityClass(const FGameplayTag& InAbilityTag) const;
```

7、本地调用`UFPAbilityManagerComponent::InitAbilityInput()`，通过读取**能力属性配置**的`ConfirmCancelInput`和`ActiveAbilities`自动绑定能力输入，死亡时移除所有能力输入，
也可以通过函数`UFPAbilityManagerComponent`的函数手动绑定输入

```c++
// UFPAbilityManagerComponent的函数↓↓↓

// 初始化能力输入，通过AttributeConfigPtr绑定输入，建议在APawn::SetupPlayerInputComponent中调用
void InitAbilityInput();

// 移除所有能力输入(仅支持本地调用)
void RemoveAllAbilityInput();

// 绑定能力输入(仅支持本地调用)，按键被使用时会自动解绑之前的能力
bool BindAbilityInput(const FGameplayTag& InAbilityTag, const FFPAbilityInput& InAbilityInput);

// 解除此按键绑定的能力(仅支持本地调用)
void RemoveAbilityInput(const FFPAbilityInput& InAbilityInput);
```

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
```

<a name="fpabilitysystem-abilitymodel"></a>
* **能力模型**

**能力模型**用于管理配置能力，修改数据表格时会同步修改编译蓝图，数据表格保存时也会同步保存蓝图，同时也会更新保存[能力映射表](#fpabilitysystem-maptable)，从而影响C++能力构造。<br>
将能力模型添加给[能力属性配置](#fpabilitysystem-attributeconfig)，通过[能力属性配置](#fpabilitysystem-attributeconfig)数据表格的`Key`和**能力标签**通过[能力映射表](#fpabilitysystem-maptable)查询能力

<a name="fpabilitysystem-maptable"></a>
1、继承`FFPAbilityData`创建**能力模型**的数据表格

```c++
// 能力数据，继承此结构体创建能力模型
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
	FFPAbilityCooldownData Cooldown;

	// 消耗
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TMap<FGameplayAttribute, FScalableFloat> Costs;

	// 伤害
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	FFPAbilityDamageData Damage;

	// 组合
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (EditCondition = "bIsAbilityComboClass", EditConditionHides, HideEditConditionToggle))
	FFPAbilityComboData Combo;

	// 图标
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TSoftObjectPtr<UTexture2D> Icon = nullptr;

#if WITH_EDITORONLY_DATA
private:

	// 首次启动时跳过。启动引擎时，不执行OnDataTableChanged()
	bool bSkipOnFirstLaunch = true;

	// 能力类的路径名称(仅用于蓝图类)。为空时，代表数据表格此行没有改动
	FString ClassPathName = "";

	// 上次能力标签
	UPROPERTY()
	FGameplayTag LastTag = FGameplayTag::EmptyTag;

	// 判断是否为运行能力组合的能力类
	UPROPERTY()
	bool bIsAbilityComboClass = false;
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
// 能力冷却数据
USTRUCT(BlueprintType)
struct FFPAbilityCooldownData
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
**伤害类型标签**：标签为空时，为普通[伤害计算](#fpabilitysystem-damagecalculation)。插件提供了`百分比伤害`、`真实伤害`和`关闭命中反馈`，
添加`关闭命中反馈`标签后将不执行`UFPAbilityManagerComponent::ReceivedDamageDelegate`，会停止播放受击动画、声音等<br>
**伤害给与目标的效果**：如着火、眩晕等

```c++
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

// 接收伤害委托，源和目标的服务器和客户端都可接收到委托。攻击拥有"FPAbility.Damage.Type.NotHitReact"标签将不执行此委托，可以播放被击反应的动画和声音，也用来显示伤害数字
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityReceivedDamageDelegate ReceivedDamageDelegate;
```

```c++
// 能力伤害数据
USTRUCT(BlueprintType)
struct FFPAbilityDamageData
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

	// 是否有效
	FORCEINLINE bool IsValid() const
	{
		return Value > 0;
	}
};
```

5、仅当能力继承`UFPAbilityComboBase`时，才会显示此能力组合设置

```c++
// 能力组合数据
USTRUCT(BlueprintType)
struct FFPAbilityComboData
{
	GENERATED_BODY()

public:

	// 能力组合
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	TSoftObjectPtr<UFPAbilityCombo> AbilityCombo = nullptr;

	// 初始输入标签，转换到导管节点后的第一个动画节点，存在导管节点时才会生效
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (Categories = "FPInput.Ability"))
	FGameplayTag InitialInputTag = FGameplayTag::EmptyTag;

	// 是否使用EventReceived代理广播内部游戏事件，AI使用此能力时会自动开启
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
	bool bBroadcastInternalEvents = false;
};
```

<a name="fpabilitysystem-maptable"></a>
* **能力映射表**

管理所有能力映射，添加到`UFPAbilityProjectSettings::AbilityMapTable`通过[能力属性配置](#fpabilitysystem-attributeconfig)或任何[能力模型](#fpabilitysystem-abilitymodel)保存时自动更新(不支持手动修改)，也可以通过RefreshMapTable()刷新<br>

1、**功能**<br>
(1)、通过[能力属性配置](#fpabilitysystem-attributeconfig)的`Key`和[能力模型](#fpabilitysystem-abilitymodel)的技能标签查询能力<br>
(2)、[能力模型](#fpabilitysystem-abilitymodel)保存时，检测保存的[能力模型](#fpabilitysystem-abilitymodel)和其他[能力模型](#fpabilitysystem-abilitymodel)的能力是否有重复，有重复会弹窗提示<br>
(3)、用于构造C++能力(蓝图会自己更新保存)

2、**能力映射表**的数据资产继承的类

```c++
// 能力映射表，不支持手动修改，添加到UFPAbilityProjectSettings::AbilityMapTable会自动更新
UCLASS()
class FPABILITYSYSTEM_API UFPAbilityMapTable : public UDataAsset
{
	GENERATED_BODY()

protected:

	// 能力映射，通过能力属性配置的Key和能力模型的技能标签查询能力
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, meta = (EditCondition = "false"), Category = "Config")
	TMap<FName, FFPAbilityMap> AbilityMaps;

	// 能力模型映射；1、能力模型保存时，检测所有能力模型的能力是否有重复；2、用于构造C++能力(蓝图会自己更新保存)
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, meta = (EditCondition = "false"), Category = "Config")
	TMap<FSoftClassPath, FFPAbilityModelMap> AbilityModelMaps;

public:

	// 查找能力映射
	// @param InKey UFPAbilityProjectSettings::AttributeConfig数据表格的Key
	const FFPAbilityMap* FindAbilityMap(const FName& InKey) const;

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

	// 添加能力模型映射
	// @param InAbilityModel 能力模型，数据表格需继承FFPAbilityData
	void AddAbilityModelMap(UDataTable* InAbilityModel);

	// 刷新映射表，通过读取UFPAbilityProjectSettings::AttributeConfig和所有能力模型刷新映射表
	UFUNCTION(CallInEditor, Category = "Config")
	void RefreshMapTable();
#endif
};
```

```c++
// UFPAbilityMapTable使用的结构体↓↓↓

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
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
	FDataTableRowHandle RowHandle;

	// 是否为蓝图类
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
	bool bIsBlueprintClass = false;
};
```

<a name="fpabilitysystem-usageguide"></a>
* **使用指南**

1、根据自己的游戏需求添加`UFPAttributeSet`和`UFPAbilitySystemComponent`，以下为我的添加方法，仅供参考<br>
给`APlayerState`添加`UFPAttributeSet`和`UFPAbilitySystemComponent`，继承`IAbilitySystemInterface`接口，并实现接口的函数`GetAbilitySystemComponent`。
给`APawn`也继承`IAbilitySystemInterface`接口，并实现接口的函数`GetAbilitySystemComponent`

```c++
AMyPlayerState::AMyPlayerState()
{
	AttributeSet = CreateDefaultSubobject<UFPAttributeSet>(TEXT("AttributeSet"));
	AbilitySystemComponent = CreateDefaultSubobject<UFPAbilitySystemComponent>(TEXT("AbilitySystemComponent"));

	// AbilitySystemComponent的ReplicationMode默认为Mixed，AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Minimal);
}

UAbilitySystemComponent* AMyPlayerState::GetAbilitySystemComponent() const
{
	return AbilitySystemComponent;
}

UAbilitySystemComponent* AMyPawn::GetAbilitySystemComponent() const
{
	// 通过能力管理组件获取ASC
	return AbilityManagerComp->GetAbilitySystemComponent();
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

// 获取被杀死后给与伤害源的奖励
virtual UGameplayEffect* GetBountyGameplayEffect() const;
```

<a name="fpabilitysystem-aiactivateability"></a>
4、AI激活能力，激活普通技能只需要设置`AbilityTag`，能力结束时会结束AI任务，也可以通过设置`WaitTime`强制取消能力，`InputTags`只对继承`UFPAbilityComboBase`的能力有效

![FPAbilitySystem_AIActivateAbility](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilitySystem_AIActivateAbility.png)

```c++
// 能力标签
UPROPERTY(EditAnywhere, meta = (Categories = "FPAbility.Ability"), Category = "FPAbility")
FGameplayTag AbilityTag = FGameplayTag::EmptyTag;

// 等待强制取消能力时间(s)，时间为0时不强制取消能力
UPROPERTY(EditAnywhere, meta = (ClampMin = 0), Category = "FPAbility")
float WaitTime = 0.0f;

// 输入标签，能力UFPAbilityComboBase会使用此标签切换组合动画
UPROPERTY(EditAnywhere, meta = (Categories = "FPInput.Ability"), Category = "FPAbility")
TArray<FGameplayTag> InputTags;
```

<a name="fpabilitysystem-blueprintapi"></a>
5、根据情况调用`UFPAbilityManagerComponent`的蓝图API，C++使用去掉`K2_`的函数，可以减少内存消耗

```c++
// 移除所有能力(仅支持服务器调用)
UFUNCTION(BlueprintCallable, DisplayName = "RemoveAbilities", Category = "FPAbility")
void K2_RemoveAbilities();

// 给予能力(仅支持服务器调用)
UFUNCTION(BlueprintCallable, DisplayName = "GiveAbility", Category = "FPAbility")
void K2_GiveAbility(UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 清除能力(仅支持服务器调用)
UFUNCTION(BlueprintCallable, DisplayName = "ClearAbility", Category = "FPAbility")
void K2_ClearAbility(UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 激活能力(仅支持服务器调用)
UFUNCTION(BlueprintCallable, DisplayName = "ActivateAbility", Category = "FPAbility")
void K2_ActivateAbility(UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 取消能力(仅支持服务器调用)
UFUNCTION(BlueprintCallable, DisplayName = "CancelAbility", Category = "FPAbility")
void K2_CancelAbility(UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag);

// 初始化能力输入，通过AttributeConfigPtr绑定输入(仅支持本地调用)
UFUNCTION(BlueprintCallable, DisplayName = "InitAbilityInput", Category = "FPAbility")
void K2_InitAbilityInput();

// 移除所有能力输入(仅支持本地调用)
UFUNCTION(BlueprintCallable, DisplayName = "RemoveAllAbilityInput", Category = "FPAbility")
void K2_RemoveAllAbilityInput();

// 绑定能力输入(仅支持本地调用)，按键被使用时会自动解绑之前的能力
UFUNCTION(BlueprintCallable, DisplayName = "BindAbilityInput", Category = "FPAbility")
bool K2_BindAbilityInput(UPARAM(meta = (Categories = "FPAbility.Ability")) FGameplayTag InAbilityTag, const FFPAbilityInput& InAbilityInput);

// 解除此按键绑定的能力(仅支持本地调用)
UFUNCTION(BlueprintCallable, DisplayName = "RemoveAbilityInput", Category = "FPAbility")
void K2_RemoveAbilityInput(const FFPAbilityInput& InAbilityInput);
```

6、根据情况调用`UFPAbilityFunctionLibrary`的函数

```c++
// 获取能力映射表
static UFPAbilityMapTable* GetAbilityMapTable();

// 查找能力数据
// @param InKey UFPAbilityProjectSettings::AttributeConfig数据表格的Key
static const FFPAbilityMap* FindAbilityMap(const FName& InKey);

// 应用伤害
// @param InAvatarActor 目标AvatarActor
// @param InSpecHandles 伤害GE规格委托，索引0是伤害数值，其他是状态GE
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static TArray<FActiveGameplayEffectHandle> ApplyDamage(AActor* InAvatarActor, TArray<FGameplayEffectSpecHandle> InSpecHandles);

// 属性点添加到属性，加点时本地UI调用
// @param InPlayerController 本地的玩家控制器
// @param InAttribute 把对应的属性点加1点
UFUNCTION(BlueprintCallable, Category = "FPAbility")
static void PointsAddToAttributePoints(APlayerController* InPlayerController, const FGameplayAttribute& InAttribute);

// 获取命中反应方向
// @param InTarget 目标能力管理组件
// @param InImpactPoint 被攻击的位置
UFUNCTION(BlueprintPure, Category = "FPAbility")
static const FGameplayTag GetHitReactDirectionTag(UFPAbilityManagerComponent* InTarget, const FVector& InImpactPoint);
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
|XPBounty|角色死亡时奖励给杀手的经验值 = $XPBounty \times Level$|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|Points|加点剩余的属性点<br>可以加点的属性包括：血量、耐力、能量、速度和力量<br>已经使用的属性点 + $Points = LevelUpPoints \times Level$|存档数据|是|
|LevelUpPoints|升级提供的属性点|项目设置|否|
|MaxPoints|单个属性的最大点数|项目设置|否|
|Health|当前血量|存档数据|是|
|MaxHealth|血量最大值，当最大血量变化时，当前血量保持百分比不变<br>$MaxHealth=HealthInitial+HealthPerPoints\times HealthPoints$|计算|是|
|HealthInitial|血量初始值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|HealthRegenRate|血量恢复率，恢复血量会消耗能量|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|HealthPoints|血量属性点|存档数据|是|
|HealthPerPoints|每个属性点添加的血量值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|Stamina|当前耐力|存档数据|是|
|MaxStamina|耐力最大值，当最大耐力变化时，当前耐力保持百分比不变<br>$MaxStamina=StaminaInitial+StaminaPerPoints\times StaminaPoints$|计算|是|
|StaminaInitial|耐力初始值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|StaminaRegenRate|耐力恢复率，恢复耐力会消耗能量|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|StaminaPoints|耐力属性点|存档数据|是|
|StaminaPerPoints|每个属性点添加的耐力值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|Energy|当前能量，恢复血量和耐力会额外消耗能量|存档数据|是|
|MaxEnergy|能量最大值，当最大耐力变化时，当前能量保持不变(百分比改变)<br>$MaxEnergy=EnergyInitial+EnergyPerPoints\times EnergyPoints$|计算|是|
|EnergyInitial|能量初始值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|EnergyUseRate|能量消耗率，角色活着会持续消耗能量；没有能量时，持续消耗血量|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|EnergyPoints|能量属性点|存档数据|是|
|EnergyPerPoints|每个属性点添加的能量值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|Speed|影响所有行为的速度，按百分比增速Speed%<br>$Speed=100+SpeedPerPoints\times SpeedPoints$|计算|是|
|SpeedPoints|速度属性点|存档数据|是|
|SpeedPerPoints|每个属性点添加的速度值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|Strength|力量，按百分比增伤Strength%<br>$Strength=100+StrengthPerPoints\times StrengthPoints$|计算|是|
|StrengthPoints|力量属性点|存档数据|是|
|StrengthPerPoints|每个属性点添加的力量值|[能力属性配置](#fpabilitysystem-attributeconfig)|是|
|WeaponDamage|武器伤害|装备|是|
|Armor|护甲|装备|是|
|IncreaseDamage|百分比增伤，取值范围[-1, +∞]|Buff|是|
|ReduceDamage|百分比减伤，取值范围[-∞, 1]|Buff|是|
|Damage|受到的伤害，仅用于[伤害计算](#fpabilitysystem-damagecalculation)|计算|否|
|MaxWeight|最大负重|[能力属性配置](#fpabilitysystem-attributeconfig)|是|

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

能力组合(技能连招系统)。此模块参考[Combo Graph](https://www.fab.com/listings/b0828da5-26a2-400a-b52d-2b9fd77c2bc6)(插件已购买)，
复刻后移除了**工具栏的模式切换**、**节点自动排列**、**非GAS的逻辑**(现在插件仅对GAS有效)、**旧粒子系统**(仅留下Niagara)；<br>
**不同部分**：必须基于骨架设置动画组合，输入基于[FPInput](#fpcommon-fpinput)用游戏标签设置输入，将序列节点和蒙太奇节点组合成一个动画节点(可以通过枚举切换)，节点Slate参考了动画蓝图，
为配合[FPAbilitySystem](#fpabilitysystem-fpabilitysystem)模块做了很多修改，但不影响此模块独立使用。

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPAbilityTask_RunAbilityCombo|运行能力组合任务|
|FPAbilityTask_PlayMontage|播放蒙太奇任务|
|FPAbilityTask_NetworkSyncPoint|网络同步点任务：为客户端和服务器提供通用同步点的任务|
|FPAbilityComboTasksComponent|能力组合任务组件，用于发送网络复制的游戏事件(按键输入后切换动画节点)|
|FPAbilityComboCollisionComponent|能力组合碰撞组件：为网格提供基本的[攻击碰撞检测](#fpabilitysystem-fpabilitycombo-attacktrace)机制|
|FPAbilityComboAnimNotifyState|能力组合动画通知状态：在组合动画开始或结束时发送事件|
|FPAbilityCollisionAnimNotifyState|能力碰撞动画通知状态，用来控制`FPAbilityComboCollisionComponent`的碰撞检测|
|FPAbilityGameplayCueNotify_HitImpact|能力游戏提示通知_命中打击，用于生成打击目标的粒子和声音|
|FPAbilityCombo|能力组合，将动画组合成连击动作|
|FPAbilityComboNodeBase|能力组合节点基类|
|FPAbilityComboNodeTrans|能力组合节点转换，处理节点转换|
|FPAbilityComboNodeConduit|能力组合导管节点，建立分支|
|FPAbilityComboNodeEntry|能力组合开始节点|
|FPAbilityComboNodeAnim|能力组合动画节点，用来添加动画序列或动画蒙太奇，网络环境仅支持动画蒙太奇|

* **使用指南**

1、**FPAbilityCombo**项目设置

|属性|数据类型|描述|
|:-:|:-:|:-:|
|DynamicMontageSlotName|`FName`|动画序列创建动态蒙太奇的插槽名称|
|NotifyStates|`TMap<TSoftClassPtr<UAnimNotifyState>, FFPAbilityComboNotifyStateAutoSetup>`|自动设置动画通知状态，给所有播放的动画添加动画通知状态|
|bAutoAddAbilityCollisionNotify|`bool`|伤害有效时自动给动画节点添加`UFPAbilityCollisionAnimNotifyState`|
|CostGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|消耗GE类，所有动画都使用此GE，插件已设置好，不建议修改|
|DamageGameplayEffectClass|`TSoftClassPtr<UGameplayEffect>`|伤害GE类，所有动画都使用此GE，根据自己的需求更换计算逻辑|
|bSequencesNetworkedWarning|`bool`|网络环境中，使用动画序列发送消息警告(网络环境仅支持动画蒙太奇)|
|TraceColor|`FLinearColor`|绘制[攻击碰撞检测](#fpabilitysystem-fpabilitycombo-attacktrace)轨迹的检测颜色|
|TraceHitColor|`FLinearColor`|绘制[攻击碰撞检测](#fpabilitysystem-fpabilitycombo-attacktrace)轨迹的检测命中颜色|
|DebugDrawTime|`float`|绘制[攻击碰撞检测](#fpabilitysystem-fpabilitycombo-attacktrace)轨迹的调试绘制时间|

![FPAbilityComboSettings](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityComboSettings.png)

2、在UGameplayAbility中调用RunAbilityCombo运行能力组合。也可以直接使用[FPAbilitySystem](#fpabilitysystem-fpabilitysystem)的`FPAbilityComboBase`来运行能力组合

```c++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FFPAbilityComboTaskDelegate, FGameplayTag, EventTag, FGameplayEventData, EventData);

// 开始能力组合时调用的委托
UPROPERTY(BlueprintAssignable)
FFPAbilityComboTaskDelegate OnComboStart;

// 结束能力组合时调用的委托
UPROPERTY(BlueprintAssignable)
FFPAbilityComboTaskDelegate OnComboEnd;

//  接收能力组合节点事件时调用的委托
UPROPERTY(BlueprintAssignable)
FFPAbilityComboTaskDelegate EventReceived;

// 创建并运行能力组合，从GA中启动能力组合的资产
// @param InOwningAbility 拥有的能力
// @param InAbilityCombo 要运行的能力组合资源
// @param InInitialInputTag 初始输入标签，转换到导管节点后的第一个动画节点，存在导管节点时才会生效
// @param bInBroadcastInternalEvents 是否使用EventReceived代理广播内部游戏事件(包括组合开始和结束事件)
UFUNCTION(BlueprintCallable, Category = "FPAbility|Tasks", meta = (DisplayName = "RunAbilityCombo", AdvancedDisplay = "InInitialInputTag, bInBroadcastInternalEvents", HidePin = "InOwningAbility", DefaultToSelf = "InOwningAbility", BlueprintInternalUseOnly = "TRUE"))
static UFPAbilityTask_RunAbilityCombo* CreateRunAbilityCombo(UGameplayAbility* InOwningAbility, UFPAbilityCombo* InAbilityCombo, UPARAM(meta = (Categories = "FPInput.Ability")) FGameplayTag InInitialInputTag, bool bInBroadcastInternalEvents = false);
```

![FPAbilityCombo_Run](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_Run.png)

3、在内容浏览器中创建新的**能力组合**或**动画节点蓝图**

![FPAbilityCombo_AssetType](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_AssetType.png)

4、选择要组合动画的骨架创建**能力组合**，类似动画蓝图；选择父类**动画节点蓝图**创建**动画节点蓝图**(可选)

![FPAbilityCombo_Filter](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_Filter.png)
![FPAbilityCombo_Panel](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_Panel.png)
![FPAbilityCombo_AnimNodeBlueprint](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_AnimNodeBlueprint.png)

5、节点转换

![FPAbilityCombo_NodeTrans](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_NodeTrans.png)

```c++
// 能力组合转换行为
UENUM(BlueprintType)
enum class EFPAbilityComboTransitionBehavior : uint8
{
	// 输入按键后，动画结束时执行(预输入)
	OnComboEnd,

	// 输入按键后，立即触发组合转换
	Immediately,

	// 输入按键后，动画通知触发组合转换，如果已动画通知，则立即触发
	OnAnimNotify,
};

// 转到下一个节点的输入标签
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (Categories = "FPInput.Ability"), Category = "FPAbility|Transition")
FGameplayTag TransitionInputTag;

// 输入事件类型
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "bIsValidInputTag", EditConditionHides, HideEditConditionToggle), Category = "FPAbility|Transition")
ETriggerEvent TriggerEvent = ETriggerEvent::Triggered;

// 能力组合转换行为，转换到下一个节点
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "bIsValidInputTag", EditConditionHides, HideEditConditionToggle), Category = "FPAbility|Transition")
EFPAbilityComboTransitionBehavior TransitionBehavior = EFPAbilityComboTransitionBehavior::OnComboEnd;

// 根据动画通知名称触发转换，接受到通知后转换
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "bShowAnimNotifyName", EditConditionHides, HideEditConditionToggle), Category = "FPAbility|Transition")
FName TransitionAnimNotifyName = NAME_None;

// 根据动画通知触发转换，接受到通知后转换
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "bShowAnimNotify", EditConditionHides, HideEditConditionToggle), Category = "FPAbility|Transition")
TSoftClassPtr<UAnimNotify> TransitionAnimNotify;
```

6、动画节点，可以通过拖拽动画资产创建动画节点，也可以右键添加创建设置好的**动画节点蓝图**

![FPAbilityCombo_AnimNode](https://github.com/FirePlume126/FP_Readme/blob/main/Images/FPAbilityCombo_AnimNode.png)

```c++
// 动画类型
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Anim")
EFPAbilityComboAnimType AnimType = EFPAbilityComboAnimType::Montage;

// 动画序列
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "AnimType == EFPAbilityComboAnimType::Sequence", EditConditionHides), Category = "FPAbility|Anim")
TSoftObjectPtr<UAnimSequence> Sequence;

// 蒙太奇
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "AnimType == EFPAbilityComboAnimType::Montage", EditConditionHides), Category = "FPAbility|Anim")
TSoftObjectPtr<UAnimMontage> Montage;

// 蒙太奇开始部分
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (EditCondition = "AnimType == EFPAbilityComboAnimType::Montage", EditConditionHides), Category = "FPAbility|Anim")
FName StartSection = NAME_None;

// 动画蒙太奇播放率
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (ClampMin = "0.0"), Category = "FPAbility|Anim")
float MontagePlayRate = 1.0f;

// 根运动平移的比例
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (ClampMin = "0.0"), Category = "FPAbility|Anim")
float RootMotionScale = 1.0f;

// 能力结束时停止动画
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Anim")
bool bStopAnimationWhenAbilityEnds = true;

// 通知状态覆盖，播放动画的之前给动画添加此通知状态和UFPAbilityComboProjectSettings::NotifyStates
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Notify")
TMap<TSoftClassPtr<UAnimNotifyState>, FFPAbilityComboNotifyStateAutoSetup> NotifyStatesOverrides;

// 能力消耗
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Abilities")
TMap<FGameplayAttribute, FScalableFloat> AbilityCosts;

// 能力伤害
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Abilities")
FFPAbilityDamageData AbilityDamage;

// GE容器映射
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Abilities")
TMap<FGameplayTag, FFPAbilityGameplayEffectContainer> EffectsContainerMap;

// GC容器映射
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Abilities")
TMap<FGameplayTag, FFPAbilityGameplayCueContainer> CuesContainerMap;

// 事件标签容器，当有这些标签时触发事件
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "FPAbility|Abilities")
FGameplayTagContainer EventTags;
```

```c++
// 能力伤害数据
USTRUCT(BlueprintType)
struct FFPAbilityDamageData
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

	// 是否有效
	FORCEINLINE bool IsValid() const
	{
		return Value > 0;
	}
};

// 游戏效果容器
USTRUCT(BlueprintType)
struct FFPAbilityGameplayEffectContainer
{
	GENERATED_BODY()

public:

	FFPAbilityGameplayEffectContainer() {};

	// 应用于目标的GE列表
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "GameplayEffectContainer")
	TArray<TSubclassOf<UGameplayEffect>> TargetGameplayEffectClasses;

	// 是否使用设置调用者数量
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "GameplayEffectContainer")
	bool bUseSetByCallerMagnitude = false;

	// 设置调用者数据标签
	UPROPERTY(EditAnywhere, BlueprintReadOnly, meta = (EditCondition = "bUseSetByCallerMagnitude", EditConditionHides), Category = "GameplayEffectContainer")
	FGameplayTag SetByCallerDataTag;

	// 设置调用者数量
	UPROPERTY(EditAnywhere, BlueprintReadOnly, meta = (EditCondition = "bUseSetByCallerMagnitude", EditConditionHides), Category = "GameplayEffectContainer")
	float SetByCallerMagnitude = 1.0f;
};

// GC容器
USTRUCT(BlueprintType)
struct FFPAbilityGameplayCueContainer
{
	GENERATED_BODY()

public:

	// GC列表容器
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "GameplayCueContainer")
	TArray<FFPAbilityGameplayCueContainerDefinition> Definitions;

	// 获取GT对应的GC列表和原对象
	void GetAggregatedDefinitionsAndObjects(TMap<FGameplayTag, FFPAbilityGameplayCueContainerDefinition>& OutDefinitions, TArray<TWeakObjectPtr<UObject>>& OutSourceObjects);

	// 获取GT对应的GC列表和原对象的路径
	void GetAggregatedDefinitionsAndPaths(TMap<FGameplayTag, FFPAbilityGameplayCueContainerDefinition>& OutDefinitions, TArray<FSoftObjectPath>& OutSoftObjectPaths);
};
```

<a name="fpabilitysystem-fpabilitycombo-attacktrace"></a>
7、添加**攻击碰撞检测**，给`APawn`添加`UFPAbilityComboCollisionComponent`，
对`UFPAbilityComboCollisionComponent`做基础设置后，调用`RegisterCollisionMesh()`和`UnregisterCollisionMesh()`注册或注销网格碰撞。
只有添加了动画通知状态`UFPAbilityCollisionAnimNotifyState`的动画(技能伤害有效时，自动添加)，才能启动此组件的检测<br>
绘制攻击轨迹的调试指令：`FP.Ability.Debug.DrawAttackTrace (bool)`

```c++
// 跟踪类型
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "FPAbility|Traces")
EFPAbilityCollisionTraceType TraceType = EFPAbilityCollisionTraceType::Line;

// 检测半径
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (EditCondition = "TraceType == EFPAbilityCollisionTraceType::Sphere", EditConditionHides), Category = "FPAbility|Traces")
float SphereTraceRadius = 3.0f;

// 盒体检测尺寸(模型的长, 宽, 高)
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, meta = (EditCondition = "TraceType == EFPAbilityCollisionTraceType::Box", EditConditionHides), Category = "FPAbility|Traces")
FVector BoxTraceSize = FVector(10.0f);

// 检测复杂碰撞，为false时简化碰撞
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "FPAbility|Traces")
bool bTraceComplex = true;

// 碰撞跟踪通道
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "FPAbility|Traces")
TEnumAsByte<ETraceTypeQuery> CollisionTraceChannel = TraceTypeQuery2;

// 要忽略的Actor类型
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "FPAbility|Traces")
TArray<TSubclassOf<AActor>> ActorTypesToIgnore;

// 要忽略的碰撞配置文件
UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "FPAbility|Traces")
TArray<FName> CollisionProfilesToIgnore;

// 仅在服务器上成功命中时调用委托
UPROPERTY(BlueprintAuthorityOnly, BlueprintAssignable, Category = "FPAbility")
FFPAbilityComboHitRegisteredDelegate OnHitRegistered;

// 检查到网格时调用委托
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityComboSimpleDelegate OnTraceStart;

// 结束检查到网格时调用委托
UPROPERTY(BlueprintAssignable, Category = "FPAbility")
FFPAbilityComboSimpleDelegate OnTraceEnd;

// 注册网格碰撞
UFUNCTION(BlueprintCallable, Category = "FPAbility")
virtual void RegisterCollisionMesh(UPrimitiveComponent* InMesh);

// 注销网格碰撞
UFUNCTION(BlueprintCallable, Category = "FPAbility")
virtual void UnregisterCollisionMesh(UPrimitiveComponent* InMesh);
```

```C++
// 能力碰撞跟踪类型
UENUM(BlueprintType)
enum class EFPAbilityCollisionTraceType : uint8
{
	// 球体检测
	Sphere,

	// 盒体检测
	Box,
};

DECLARE_DYNAMIC_MULTICAST_DELEGATE(FFPAbilityComboSimpleDelegate);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FFPAbilityComboHitRegisteredDelegate, FHitResult, HitResult);
```

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
|FPAbilityComboEdGraph|能力组合编辑器图表|
|FPAbilityComboSchema|能力组合编辑器图表架构：定义和管理能力组合图表节点|
|FPAbilityComboApplicationMode|能力组合应用程序模式，创建此模式的选项卡|
|FPAbilityComboAnimAssetTabFactory|能力组合动画资产选项卡工厂|
|FPAbilityComboGraphViewportTabFactory|能力组合图表视口选项卡工厂|
|FPAbilityComboPropertyDetailsTabFactory|能力组合属性细节选项卡工厂|
|FPAbilityComboDebugger|能力组合调试器，编辑器播放时，可以在能力组合编辑器选择调试的对象进行调试|
|FPAbilityComboConnectionDrawingPolicy|能力组合连接绘制策略|
|FPAbilityComboDragDropAction|能力组合拖放操作|
|FPAbilityComboEdNodeBase|能力组合编辑器节点基类|
|FPAbilityComboEdNodeEntry|能力组合编辑器开始节点|
|FPAbilityComboEdNodeConduit|能力组合编辑器导管节点，建立分支|
|FPAbilityComboEdNodeTrans|能力组合编辑器节点转换|
|FPAbilityComboEdNodeAnim|能力组合编辑器动画节点|
|SFPAbilityComboNodeEntry|能力组合开始节点Slate|
|SFPAbilityComboNodeConduit|能力组合导管节点Slate，建立分支|
|SFPAbilityComboOutputPin|能力组合输出引脚Slate|
|SFPAbilityComboNodeTrans|能力组合节点转换Slate|
|SFPAbilityComboNodeAnim|能力组合动画节点Slate|

<a name="fpfeatures"></a>
## FPFeatures

包含简单的游戏功能模块

<a name="fpfeatures-fpinteraction"></a>
### FPInteraction

交互

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPInteractionComponent|互动组件，添加此组件的APawn或APlayerController可使用互动功能|
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

<a name="fpeditortools"></a>
## FPEditorTools

编辑器工具

* 动画修改器

<a name="non-self-made-assets"></a>
## Non self-made assets

非自制资产，除此之外的**美术**和所有**程序**均为自制

|资产类型|数量|资产来源|使用资产模块|
|:-:|:-:|:-:|:-:|
|字体|1|[Source Han Sans](https://www.github.com/adobe-fonts/source-han-sans)|[FPAssets](#fpassets)|
|图标_鼠标|9|[Lyra Starter Game](https://www.fab.com/listings/93faede1-4434-47c0-85f1-bf27c0820ad0)|[FPUI](#fpcommon-fpui)|
|动画序列|93|[Advanced Locomotion System V4](https://www.fab.com/listings/ef9651a4-fb55-4866-a2d9-1b38b028f9c7)|Game|
|音频_脚步|16|[Advanced Locomotion System V4](https://www.fab.com/listings/ef9651a4-fb55-4866-a2d9-1b38b028f9c7)|Game|

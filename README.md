
# 项目框架

* **项目未开源**
* **所有模块都支持多人联机游戏**

## 作者信息

Copyright FirePlume, All Rights Reserved.

Email: fireplume@126.com<br>
GitHub: [FirePlume126](https://www.github.com/FirePlume126)<br>
Bilibili: [火羽FP](https://space.bilibili.com/395084718)<br>
YouTube: [FirePlume126](https://www.youtube.com/@FirePlume126)

<a name="Directory"></a>
## 框架目录

**框架目录定义**

- 类别
	- 插件名称
		- 模块名称(只有一个模块且与插件同名时，不显示此行)

* Common：此类插件包含通用基础功能
	- [FPCommon](https://www.github.com/FirePlume126/FP_Common#fpcommon)：包含通用基础功能的插件
		- [FPInput](https://www.github.com/FirePlume126/FP_Common#fpcommon-fpinput)：管理输入(仅用于C++)
		- [FPUI](https://www.github.com/FirePlume126/FP_Common#fpcommon-fpui)：管理用户界面
	- [FPLoadingScreen](https://www.github.com/FirePlume126/FP_Common#fploadingscreen)：游戏启动或地图转换期间的加载界面

- System：此类插件为独立系统，可根据游戏类型选择使用
	- [FPOnlineSystem](https://www.github.com/FirePlume126/FP_OnlineSystem#fponlinesystem)：管理服务器和会话，加载/保存服务器存档和玩家存档并生成玩家
	- [FPAbilitySystem](https://www.github.com/FirePlume126/FP_AbilitySystem#fpabilitysystem)：能力系统，基于GAS主要包括能力属性管理框架和技能连招系统
		- [FPAbilitySystem](https://www.github.com/FirePlume126/FP_AbilitySystem#fpabilitysystem-fpabilitysystem)：能力属性管理框架
		- [FPAbilityCombo](https://www.github.com/FirePlume126/FP_AbilitySystem#fpabilitysystem-fpabilitycombo)：能力组合(技能连招系统)
		- [FPAbilityComboEditor](https://www.github.com/FirePlume126/FP_AbilitySystem#fpabilitysystem-fpabilitycomboeditor)：能力组合编辑器
	- [FPSpawnerSystem](https://www.github.com/FirePlume126/FP_SpawnerSystem#fpspawnersystem)：生成器系统
		- [FPSpawnerSystemEditor](https://www.github.com/FirePlume126/FP_SpawnerSystem#fpspawnersystem_fpspawnersystemeditor)：在编辑器中定义与生成实体数据
		- [FPSpawnerSystem](https://www.github.com/FirePlume126/FP_SpawnerSystem#fpspawnersystem_fpspawnersystem)：负责管理实体数据并动态生成实体

* Misc：此类包含杂项插件
	- [FPFeatures](https://www.github.com/FirePlume126/FP_Misc#fpfeatures)：包含简单的游戏功能模块
		- [FPInteraction](https://www.github.com/FirePlume126/FP_Misc#fpfeatures-fpinteraction)：交互
		- [FPInventory](https://www.github.com/FirePlume126/FP_Misc#fpfeatures-fpinventory)：库存
	- [FPEditorTools](https://www.github.com/FirePlume126/FP_Misc#fpeditortools)：编辑器工具

- [Non self-made assets](#non-self-made-assets)：非自制资产，除此之外的**美术**和所有**程序**均为自制

---

<a name="non-self-made-assets"></a>
## Non self-made assets

非自制资产，除此之外的**美术**和所有**程序**均为自制

|资产类型|数量|资产来源|使用资产模块|
|:-:|:-:|:-:|:-:|
|字体|1|[Source Han Sans](https://www.github.com/adobe-fonts/source-han-sans)|Game|
|动画序列|93|[Advanced Locomotion System V4](https://www.fab.com/listings/ef9651a4-fb55-4866-a2d9-1b38b028f9c7)|Game|
|音频_脚步|16|[Advanced Locomotion System V4](https://www.fab.com/listings/ef9651a4-fb55-4866-a2d9-1b38b028f9c7)|Game|
|图标_鼠标|9|[Lyra Starter Game](https://www.fab.com/listings/93faede1-4434-47c0-85f1-bf27c0820ad0)|[FPUI](https://www.github.com/FirePlume126/FP_Common#fpcommon-fpui)|

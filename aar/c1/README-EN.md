# Smart RFID Trolley JRI-ST-C1 SDK Instructions

### [📖中文文档](README.md) | 📖English Document

本文是智能RFID推车 JRI-ST-C1（以下简称：推车）SDK的标准的集成指南文档，用以说明推车SDK的使用方法。默认读者已经熟悉Android
Studio的基本使用方法，熟悉kotlin的基本语法，并且具有一定的Android编程基础。

耗材柜SDK包含两个`aar`包，分别是位于[aar/core](/aar/core/)中的`jri-manager-core-*.aar`
与位于[aar/c1](/aar/c1/)中的`jri-manager-st-c1-*.aar`。开发时请同时导入两个最新版`aar`包到你的项目中。

## 更新记录

#### 1.0.4 📅`2024.03.04`

* 简化初始化逻辑

[Click to view more update records](CHANGE-LOG.md)

## Import SDK

### Copy File

1. Download the latest `jri-manager-core-*.aar` file from the `core` folder.
2. Download the latest `jri-manager-st-c1-*.aar` file from the `c1` folder.
3. Copy the two files to the `libs` folder in your project.

### Add Dependency

```
dependencies {
    implementation files('libs/jri-manager-core-*-release.aar')
    implementation files('libs/jri-manager-st-c1-*-release.aar')
}
```

### Import Package

```
import com.jrfid.manager.trolley.c1.JRIDevicesManager
```

## Function Introduction

### Init Connect Device

Init connect device is a long-running task,recommend using kotlin coroutines.

```
lifecycleScope.launch(Dispatchers.IO) {
    JRIDevicesManager.instance.initDevices()
}
```

### Device Control

#### IC/ID Module

##### Listening IC/ID Data

```
JRIDevicesManager.instance.addOnReceivedICCardDataCallback(object : ReceivedICCardDataCallback {
    override fun onReceived(model: ICCardPacketData) {

            
        }

    })
```

`ICCardPacketData` class description

```
class ICCardPacketData{

    val icCardNo: ByteArray

    val icCardNoText: String

}
```

| field        | description                  |
|--------------|------------------------------|
| icCardNo     | IC/ID card number byte array |
| icCardNoText | IC/ID card number string     |

#### Drawer Control

##### Open The Drawer

The trolley has a total of 6 drawers from top to bottom, corresponding to numbers 1-6.

```
JRIDevicesManager.instance.openTheDoor(drawer:Int)
```

| param      | description        |
|------------|--------------------|
| drawer:Int | drawer number(1-6) |

##### Listening Drawer State

```
JRIDevicesManager.instance.addOnReceivedBasicDataCallback(object : ReceivedBasicDataCallback {

            override fun onDoorOpen(drawer:Int) {

            }


            override fun onDoorClose(drawer:Int) {

            }

        })
```

`ReceivedBasicDataCallback` interface description

| method                      | description                    |
|-----------------------------|--------------------------------|
| fun onDoorOpen(drawer:Int)  | callback when drawer is opened |
| fun onDoorClose(drawer:Int) | callback when drawer is closed |

#### UHF Control

##### Start Inventory

```
JRIDevicesManager.instance.startUhfInventory(drawer:Int)
```

| param      | description        |
|------------|--------------------|
| drawer:Int | drawer number(1-6) |

##### Listening Inventory Result

```
JRIDevicesManager.instance.addOnReceivedUhfInventoryDataCallback(object:ReceivedUhfInventoryDataCallback{

            override fun onReceivedTag(tag: InventoryTag) {
                
            }

            override fun onInventoryTagEnd() {

            }

            override fun onInventoryFailure(inventoryFailure: InventoryFailure) {
                
            }

    })
```

`ReceivedUhfInventoryDataCallback` interface description

| method                                            | description                   |
|---------------------------------------------------|-------------------------------|
| fun onReceivedTag(tag: InventoryTag)              | callback when inventory tag   |
| fun onInventoryEnd()                              | callback when inventory end.  |
| fun onInventoryFailure(failure: InventoryFailure) | callback when inventory fail. |

`InventoryTag` class description

```
class InventoryTag {

    val pc: String

    val epc: String

    val rssi: Int
    
}
```

| field | description    |
|-------|----------------|
| pc    | tag pc data    |
| epc   | tag epc data   |
| rssi  | tag rssi value |

#### Finger Vein Control

##### Recommended Usage Process

###### Input Finger Vein

分3次获取用户指静脉特征值 -> 合成指静脉特征值模版 -> 保存特征值模版

###### Check Finger Vein

导入特征值模版到算法库 -> 记录返回的ID并于用户绑定 -> 获取指静脉特征值 -> 验证特征值是否存在

##### 是否放置手指

调用后立即返回`Boolean`类型结果，`true`表示有手指放置到指静脉模块上，反之则没有。

```
val result:Boolean = JRIDevicesManager.instance.checkFingerIn()
```

##### 是否移开手指

调用后立即返回`Boolean`类型结果，`true`表示有没有手指放置到指静脉模块上，反之则有。

```
val result:Boolean = JRIDevicesManager.instance.checkFingerOut()
```

##### 等待手指放置

该操作为阻塞操作需放置到协程中调用，调用后会一直阻塞程序，检测到手指放置到指静脉模块上后会释放。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.waitingFingerIn()
 }
```

##### 等待手指移开

该操作为阻塞操作需放置到协程中调用，调用后会一直阻塞程序，检测到手指移开后会释放。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.waitingFingerOut()
}
```

##### 获取指静脉特征值

实时获取，只有当手指正确放置到指静脉模块上时调用后会立即返回`String`
类型的指静脉特征值，其他情况下返回`null`。

```
val result:String? = JRIDevicesManager.instance.getFingerVeinChara()
```

阻塞获取，需放置到协程中调用，调用后会一直阻塞程序，直到获取到指静脉特征值后返回，如果设置的超时时间内未获取到指静脉特征值则返回`null`。

```
lifecycleScope.launch {
    val result: String? = JRIDevicesManager.instance.getFingerVeinChara(timeoutMillis = 1000)
}
```

##### 获取指静脉特征值模板

将分3次获取的指静脉特征值融合成一个特征值模版，融合成功返回`String`
类型的特征值模版，融合失败则返回`null`。

```
val result: String? = JRIDevicesManager.instance.createFingerVeinTemp("", "", "")
```

##### 持续获取指静脉特征值:boom:

通过`kotlin`中`Flow`的形式持续获取指静脉特征值，可以根据返回的`MODEL_TYPE`反馈给用户做相应的操作。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.getFingerVeinCharaFlow()
        .flowOn(Dispatchers.IO)
        .collect {
            if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_IN) {
                LogUtils.d("请放置手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_OUT) {
                LogUtils.d("请移开手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_CHARA_DATA) {
                LogUtils.d("指静脉特征值为：${it.data}")
            }
        }
}
```

##### 持续获取指静脉特征值模版:boom:

通过`kotlin`中`Flow`的形式持续获取指静脉特征值，可以根据返回的`MODEL_TYPE`反馈给用户做相应的操作。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.getFingerVeinTempFlow()
        .flowOn(Dispatchers.IO)
        .collect {
            if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_IN) {
                 LogUtils.d("请放置手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_OUT) {
                 LogUtils.d("请移开手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_TEMP_DATA) {
                 LogUtils.d("指静脉特征值模版为：${it.data}")
            }
         }
}
```
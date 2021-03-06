[Japanese version](RTK1_Core.ja.md) | [Back to toppage](README.md)

# [RTK1_Core](RTK1_Core.js) Plugin

Core functions of RTK1 library for RPG Maker MV.

Download: [RTK1_Core.js](https://raw.githubusercontent.com/yamachan/jgss-hack/master/RTK1_Core.js)

## Overview

RTK1_Core Plugin is Core function library for RTK1　series plugins (RTK1_*.js). It includes some useful functions for development, so please feel free to use it.

![Screen shot - Pligin Manager](i/RTK1_Core-01.png)

The parameters are set by 0. Normally, you don't need to change it.

![Screen shot - Plugin](i/RTK1_Core-02.png)

If you need more information, please read following;

## language parameter

Language parameter shows your RPG Maker MV's language. The default value is "0:Auto detect", and normally you don't need to change it.

![Screen shot - Parameter](i/RTK1_Core-03.png)

In "0:Auto detect" mode, plugin checks some terms in database, then automatically set "1:English" or "2:Japanese".

For example, you develop a game in Japanese environment, but change some terms into English, then plugin can get it wrong to select "1:English" value. In this case, you need to set language parameter by "2:Japanese" demonstratively.

## debug parameter

You can use the debug mode, when debug parameter is set by 1.

![Screen shot - Parameter](i/RTK1_Core-04.png)

In the debug mode, you can see RTK1 series plugin's log messages in test console.

![Screen shot - Game and log](i/RTK1_Core-05.png)

RTK.log() function is useful. It's a simple console log command, but it works only in the debug mode. Please feel free to use it.

If it gets a String as the 1st argument, it will pass it to console.log() function. If it gets an Object as the 1st argument, it will pass it to console.dir() function. It can get a String as the 1st argument, then an Object as the 2nd argument.

RTK.trace() function is also useful. It outputs the stack trace to console. It don't need an argument, but you can set a String for the label output.

## json parameter

In PC enviroment, you can find some save files in 'save' folder. These files are compressed, so not easy to read for us.

![Screen shot - Parameter](i/RTK1_Core-06.png)

If you set json parameter by 1, plugin will automatically create uncompressed version of save files in same place. These files are useful for your development work.

## onReady service

You can implement your init function easily with using RTK.onReady() function, as follows;

```js
RTK.onReady(function(){
  // your init code here
});
```

onReady service will wait the initiation of game data (end of Scene_Boot), then sets up itself, finally calls all registered functions in a sequential order.

You don't worry about init timing and order with this service.

## onCall service

To implement your original plugin command, you need to replace (hook) Game_Interpreter.prototype.pluginCommand(command, args) function. But you can implement it easily with RTK.onCall() function, as follows;

```js
RTK.onCall(command, function(args){
  // your plugin command code here
});
```

With RTK.onCall() function, your code will be simple because you don't need to check command match. As the result, it will reduce the processing cost with skipping unncesessary functions.

If you have a function which processes more than 2 command String, you can refer the 2nd argument, as follwos;

```js
RTK.onCall(command, function(args, command){
  // your plugin command code here
});
```

## Persistent service

This service extends save data, to keep plugin original data in it. The function is simple as follows; (key is String, value is your original data)

```js
RTK.save(key, value);
var value = RTK.load(key);
```

For example, if you want to keep "myData" variable in save data, you should initiate it in your plugin;

```js
var myData = RTK.load("myData") || "default value";
```

At the game start, there is no save data, so you should set the default value with "||" operator. Then you can save the value as follows;

```js
myData = "saved value";
RTK.save("myData", myData);
```

Let's see the json data with using this plugin's json parameter. You will find your value in the end of the file;

```js
{
  "system":{
  //... (中略) ...
  },
  "RTK1_Core":{
    "myData":"saved value"
  }
}
```
If you don't need the value anymore, just delete it.

```js
RTK.del(key);
```
The following is a sample event to test these functions. After you choose 1-3 value, the value will be kept after save and restart the game.

![Screen shot - Event](i/RTK1_Core-07.png)


In addtion, please use pack/unpack function which convert game variables into an Array value. (dataArray is Array、startVariable and endVariable are numbers which shows game variables)

```js
var dataArray = RTK.pack(startVariable, endVariable);
RTK.unpack(startVariable, dataArray);
```

For example. with using these functions, you can save game variables 100-119 easily as follows;

```js
RTK.save("backup100-119", RTK.pack(100,119));
```

To recover game variables 100-119, your code should be;

```js
RTK.unpack(100, RTK.load("backup100-119"));
```

By the way, RTK.unpack is also useful when you want to set lots of default values in event script. For example, the following script sets value 8 to game valiables 1-5.

```js
RTK.unpack(1, [8,8,8,8,8]);
```

In addition, Persistent service includes onSave/onLoad service which will call registered functions during the end of save/load timing. It's good to convert the data transformation for save data.


```js
RTK.onSave(function(){
  // Update your original save data with RTK.save function
});
RTK.onLoad(function(){
  // Convert your original save data with RTK.load function
});
```

For example, your plugin has a list which contains item objects, it's not good for save. You should convert item object list to item id list. In this case, these onSave/onLoad service is useful.

In this library's common game functions, RTK.objects2ids function is useful for function called by onSave, and RTK.ids2objects function is useful for function called by onLoad. Please feel free to use them.

## onStart service

This onStart service looks similar with onReady service above. But this service will be called later than onReady service. This service will be called just before Scene_Map start.

```js
RTK.onStart(function(mode){
  // your start code here
});
```

The registered function will be called with one argument (mode) which will be 1 in new game, will be 0 in loaded game. The value 2 means battle test, 3 means event test.

The important difference from onReady service is - this onStart timing is after loading save data. So if you want to use Persistent service (RTK.load function) to initiate your plugin setting, you　should use this onStart service. onReady is too early to refer the saved information.

## Simple text control service

We can set both English and Japanese comments in plugin file, but we feel a little bit difficulty about text resources which depend on the language in the code.

RTK.text provides a simple mechanism to control text resources which depend on the language in the code. For example, you can relate English text to Japanese text as follows;

```js
RTK.text("Yes", "はい");
RTK.text("No", "いいえ");
```

After this code, you can use RTK.text("Yes") in spite of "Yes". The rule of text selection in RTK.text function is as follows;

1. If the English argument doesn't have a related Japanese text, just return the English argument.
2. If RTK1_Option_EnJa plugin is found, the text selection depends on its language setting.
3. The text selection depends on the environment's language setting (RTK.\_lang).

After you register the necessary relationship among English text and Japanese text, you don't need to care about the language with using RTK.text function.

By the way, the text selection in RTK.text is not case sensitive. So RTK.text("Yes") and RTK.text("yes") have a same reault.

## Common JS functions

| function | argument | description |
| :---- | :---- | :---- |
| RTK.cloneObject | o : Object | Copy a object (shallow copy, not deep). |
| RTK.isTrue | v : Object or value | Return true, if v is true.<br>Useful with Array.filter function. |

## Common Game functions

| function | argument | description |
| :---- | :---- | :---- |
| RTK.object2id | o : Object | Convert Item/Weapon/Armor/Skill object to prefix and id String.<br>e.g. $dataItems[1] => "i1"<br>e.g. $dataWeapons[10] => "w10"|
| RTK.id2object | id : String | Backward convert of the above function. |
| RTK.objects2ids | list : Object Array | List version of RTK.object2id function.<br>It removes error "" Strings automatically.<br>e.g. [$dataArmors[2],$dataSkills[3]] => ["a2","s3"] |
| RTK.ids2objects | list : String Array | Backward convert of the above function. |

## Update history

| version | date | require | update |
| --- | --- | --- | --- |
| ver1.16 | 2016/08/16 | N/A | RPG Maker MV ver1.3.1 Support |
| [ver1.15](archive/RTK1_Core_v1.15.js) | 2016/07/17 | N/A | Support [RTK1_MapLocalVariables](RTK1_MapLocalVariables.md) plugin |
| [ver1.14](archive/RTK1_Core_v1.14.js) | 2016/07/16 | N/A | Support [RTK1_Shop](RTK1_Shop.md) plugin |
| [ver1.13](archive/RTK1_Core_v1.13.js) | 2016/07/12 | N/A | Enhance RTK.objectClone function |
| [ver1.12](archive/RTK1_Core_v1.12.js) | 2016/07/10 | N/A | Support Battle\Event Test |
| [ver1.11](archive/RTK1_Core_v1.11.js) | 2016/07/06 | N/A | Add RTK.id2object() and other id support functions |
| [ver1.09](archive/RTK1_Core_v1.09.js) | 2016/07/01 | N/A | Stable open |

## License

[The MIT License (MIT)](https://opensource.org/licenses/mit-license.php)

You don't need to display my copyright, if you keep my comments in .js files. Of course, I'll be happy, when you will display it. :-)

[Back to toppage](README.md)

---
layout:     post
title:     MacOS右shift键切换输入法
subtitle:   
date:       2022-02-10
author:     laosuan
header-img: 
catalog: true
tags:
    - 



---



## 背景

键盘升级到HHKB后，由于缺失caps键，切换输入法改为了用左shift键完成。但这样存在一个问题，经常会在输入大写字母时不小心切换了输入法，相当恼人。而系统设置中切换输入法的快捷键设置不能配置单个按键，于是找到一个更改按键的神器karabiner，能将右shift键绑定为按下多个按键，从而实现单键切换输入法，实际用过一段时间下来感觉很方便。



官网链接：

https://karabiner-elements.pqrs.org/



安装后打开Karabiner-Elements应用，根据提示打开相关系统权限

新建文件 ~/.config/karabiner/assets/complex_modifications/mine.json 

内容如下：

```json
{
   "description":"单击右Shift切换输入法!",
   "manipulators":[
      {
         "from":{
            "key_code":"right_shift"
         },
         "to":[
            {
               "key_code":"right_shift"
            }
         ],
         "to_if_alone":[
            {
               "key_code":"0",
               "modifiers":[
                  "left_control",
                  "left_option",
                  "left_command"
               ]
            }
         ],
         "type":"basic"
      }
   ]
}
```



打开karabiner-Elements的Prefrences，点击Complex modifications选项，点击左下角【add rule】，启用"单击右Shift切换输入法!"。此时我们就将右shift键绑定为"left_control" + "left_option" + "left_command" + 0 的快捷键。



打开系统偏好设置-键盘-快捷键-输入法-勾选“选择上一个输入法”， 这里默认是 ^ + 空格键，双击它， 录入 "left_control" + "left_option" + "left_command" + 0， 至此配置完成。






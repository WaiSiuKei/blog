---
title: 键盘输入事件
date: 2019-05-27 14:35:01
tags: 技术
---
# 事件类型
接触过的键盘事件（KeyboardEvent）有keydown、keyup、keypress：
- keydown：任意按键按下后。
- *keypress：当一个键按下，且这个键产生一个字符值时，浏览器才会发送keypress事件。比如字母键、数字键，标点符号；Alt, Shift, Ctrl这些键敲击后不会有keypress事件。
- keyup： 任意按键释放后。

keydown/keyup适合用来用来处理键盘敲击事件。keypress适合用来处理内容输入事件。查一下最新的Draft, keypress已经算是legacy了，要替换为beforeinput和input。
# 输入识别
先来看看KeyboardEvent的接口定义（源自Typescript）：
```ts
interface KeyboardEvent extends UIEvent {
    readonly altKey: boolean;
    /** @deprecated */
    char: string;
    /** @deprecated */
    readonly charCode: number;
    readonly code: string;
    readonly ctrlKey: boolean;
    readonly key: string;
    /** @deprecated */
    readonly keyCode: number;
    readonly location: number;
    readonly metaKey: boolean;
    readonly repeat: boolean;
    readonly shiftKey: boolean;
    /** @deprecated */
    readonly which: number;
    getModifierState(keyArg: string): boolean;
    /** @deprecated */
    initKeyboardEvent(typeArg: string, canBubbleArg: boolean, cancelableArg: boolean, viewArg: Window, keyArg: string, locationArg: number, modifiersListArg: string, repeat: boolean, locale: string): void;
    readonly DOM_KEY_LOCATION_JOYSTICK: number;
    readonly DOM_KEY_LOCATION_LEFT: number;
    readonly DOM_KEY_LOCATION_MOBILE: number;
    readonly DOM_KEY_LOCATION_NUMPAD: number;
    readonly DOM_KEY_LOCATION_RIGHT: number;
    readonly DOM_KEY_LOCATION_STANDARD: number;
}
```
其中和输入键识别有关的是char、charCode、code、key、keycode、which。keyCode, charCode和which已经废弃，原因是：
> In practice, keyCode and charCode are inconsistent across platforms and even the same implementation on different operating systems or using different localizations.

[https://www.w3.org/TR/uievents/#interface-keyboardevent](https://www.w3.org/TR/uievents/#interface-keyboardevent)

现在提倡使用key和code。code属性是代码物理按键的标识。而key属性的值有两种情况：
 - A key string that corresponds to the character typed by the user, taking into account the user’s current locale setting, modifier state, and any system-level keyboard mapping overrides that are in effect.
 - A named key attribute value，比如`Alt`, `ArrowDown`这些，参考[https://www.w3.org/TR/uievents-key/#named-key-attribute-values](https://www.w3.org/TR/uievents-key/#named-key-attribute-values)。

 比如单纯按下a键, keydown会是`{ key: 'a', code: 'keyA' }`。

 比较麻烦的是和modifier一起按下是，hold住Alt再按下a键时，keydown会是`{ key: 'å', code: 'keyA' }`。
 
 更加麻烦的是，切换到中文输入法下，输入`啊`这个字，一般的顺序是a然后空格，事件顺序是:
|    | event   | keycode | key | code  |
|----|---------|---------|-----|-------|
| 1  | keydown | 229     | a   | KeyA  |
| 2  | keyup   | 65      | a   | KeyA  |
| 3  | keydown | 229     | 空  | Space |
| 4  | keyup   | 32      | 空  | Space |
keydown出现一个keycode=229的神秘代码，这是一个经典问题。这时keypress事件是不会发送的，想要获得中文输入值，需要处理InputEvent。




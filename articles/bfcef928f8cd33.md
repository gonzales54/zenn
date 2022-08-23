---
title: "typescriptについて勉強したメモ"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript"]
published: false
---

# typescript
基本的な方はstring,number,boolean,array,null,undefined,any,関数型,オブジェクト型がある。

## string
```js
const msg: string = 'Hello World';
const msg: string = 123456789; //エラーになる
const msg: string = 'Hello World' + 123456789 //文字列の結合に+を利用して片方に数値を指定した場合、エラーにならない
```
## number
```js
const num: number = 25;
const num: number = -25.2; //小数やマイナスの値も指定可能
```
## boolean
```js
const isLogin: boolean = true; //trueかfalseしか入らない
```
## array
型名[]、もしくはArray<型名>(Generics)で指定する。
```js
const user: string[] = ['jason', 'gonzales'];
const user: Array<string> = ['json', 'gonzales'];
const user: string[] = ['jason', 10]; //違う方があるのでエラー
const user: (string | number)[] = ['jason', 10];//unionを利用して両方の型を指定
```
## null
```js
```
## undefined
```js
```
## any
```js
```
## 関数型
```js
```
## オブジェクト型
```js
```
# reaction.md

## 概述

reaction对应mobx的概念为副作用，是一种特殊的derivation。shouldCompute(reaction)判断观察的reaction.observing=[observable]数据更新时，执行reaction副作用。

## 接口

* IReactionPublic接口，须实现dispose方法。
* IReactionDisposer接口，用于装饰函数，须实现$mobx属性、onError方法。

## api

* new Reaction(name,onInvalidate)，创建reaction实例
## 结合elementUI源码讲解BEM的使用

### 前言：

​		早就听说过*BEM*命名管理样式，平时在项目中也有用到，但总感觉写起来太麻烦，最近刚好在看Elementplus源码，发现Elementplus使用BEM的方案非常的棒，本文就来结合ElementPlus的源码分析一下如何在项目中优雅的使用BEM命名

### BEM的介绍

BEM是一种针对css的前端命名规范，是块(Block)，元素（Element），修饰符（Modifier）的简写。

Block可以理解为模块，比如：article，dialog，sidebar，form，tab

Element可以理解为块里面的元素，比如form里面的input，submit

modifier可以理解为对block或者element的修饰，比如修饰form__submit--disable，form--theme-dark

### BEM的命名规范

```scss
.block { }
.block__element { }
.block--modifier { }
.block__element--modifier { }
```

> 使用__来连接block和element，使用--来连接block和modifier

清楚了概念和命名规范，下面我们就来看ElementPlus是怎么实现BEM的

### BEM的实践

![image-20220917171022400](https://raw.githubusercontent.com/PoliWen/myBlog/master/example/src/assets/images/dialog.png)

以实现一个简单dialog组件为例，使用普通的BEM方法实现如下

```vue
<template>
    <div class="dialog-overlay" v-if="visible">
        <section class="dialog">
            <header class="dialog__header">
                <div class="dialog-title">{{ title }}</div>
                <span class="dialog-header__close" @click="visible=false">X</span>
            </header>
            <div class="dialog__body">
                {{ message }}
            </div>
            <footer class="dialog__footer">
                <button class="dialog__btn">{{ cancelBtnTxt }}</button>
                <button class="dialog__btn dialog__btn--primary">{{ comfirmBtnTxt }}</button>
            </footer>
        </section>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const props = defineProps({
    title:{
        type:String,
        default:'Title'
    },
    message:String,
    cancelBtnTxt:{
        type:String,
        default:'Cancel'
    },
    comfirmBtnTxt:{
        type:String,
        default:'Comfirm'
    }
})
const visible = ref(true)
const close = ()=>{
    visible.value = false
}
</script>

<style scoped lang="scss">
:global(body){
    font-family: PingFang SC,Microsoft YaHei,Arial,sans-serif;
}
.dialog-overlay{
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 2000;
    height: 100%;
    background-color:rgba(0, 0, 0, .5);
    overflow: auto;
}

.dialog {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    border-radius:2px;
    background: #fff;
    width: 559px;
    min-height: 200px;
    &__header{
        padding: 20px 20px 10px;
    }
    &__title{
        color: #333;
        font-size: 18px;
    }
    &__body{
        padding: 30px 20px;
        font-size: 14px;
        color: #333;
    }
    &-close{
        position: absolute;
        right: 15px;
        top: 15px;
        color: #999;
        font-size: 18px;
    }
    &__btn{
        min-width: 33px;
        padding: 8px 15px;
        background: none;
        border: 1px solid #ddd;
        border-radius: 2px;
        color: #666;
        margin-left: 10px;
        cursor: pointer;
        &--primary{
            background: #409eff;
            color: #fff;
            border-color:#409eff;
        }
    }
    &__footer{
        padding: 10px 20px 20px;
        display: flex;
        justify-content: flex-end;
    }
}
</style>
```

代码很简单，一看就懂，不做过多解释，下面看用ElementPlus的方法如何进行改造

### useNameSpace

首先定义一个工具方法**useNameSpace**，此方法返回符合**BEM**命名规则的方法

```typescript
// useNameSpace.ts
import { computed, unref } from 'vue'

// 定义一个组件的命名前缀
const defaultNamespace = 'poliwen'

// 定义一个描述组件状态的变量
const statePrefix = 'is-'

// 定义个_bem方法，此方法返回符合BEM规范的命名
const _bem = (
  namespace: string,
  block: string,
  blockSuffix: string,
  element: string,
  modifier: string,
) => {
  let cls = `${namespace}-${block}`
  if (blockSuffix)
    cls += `-${blockSuffix}`

  if (element)
    cls += `__${element}`

  if (modifier)
    cls += `--${modifier}`

  return cls
}
```



```typescript
// useNameSpace.ts
// 定义一个命名空间，并且返回BEM方法
export const  useNamespace = (block: string) => {
  const namespace = computed(() => defaultNamespace)
  const b = (blockSuffix = '') =>
    _bem(unref(namespace), block, blockSuffix, '', '')
  const e = (element?: string) =>
    element ? _bem(unref(namespace), block, '', element, '') : ''
  const m = (modifier?: string) =>
    modifier ? _bem(unref(namespace), block, '', '', modifier) : ''
  const be = (blockSuffix?: string, element?: string) =>
    blockSuffix && element
      ? _bem(unref(namespace), block, blockSuffix, element, '')
      : ''
  const em = (element?: string, modifier?: string) =>
    element && modifier
      ? _bem(unref(namespace), block, '', element, modifier)
      : ''
  const bm = (blockSuffix?: string, modifier?: string) =>
    blockSuffix && modifier
      ? _bem(unref(namespace), block, blockSuffix, '', modifier)
      : ''
  const bem = (blockSuffix?: string, element?: string, modifier?: string) =>
    blockSuffix && element && modifier
      ? _bem(unref(namespace), block, blockSuffix, element, modifier)
      : ''
  const is: {
    (name: string, state: boolean | undefined): string
    (name: string): string
  } = (name: string, ...args: [boolean | undefined] | []) => {
    const state = args.length >= 1 ? args[0]! : true
    return name && state ? `${statePrefix}${name}` : ''
  }
  return {
    namespace,
    b,
    e,
    m,
    be,
    em,
    bm,
    bem,
    is,
  }
}

export type UseNamespaceReturn = ReturnType<typeof useNamespace>

```

useNameSpace库的使用

```typescript
import { useNameSpace } from './compo/useNamespace'
const bs = useNamespace('dialog')
import { useNamespace } from '../composables/useNamespace'
const ns = useNamespace('dialog')
ns.b() // el-dialog
ns.b('overlay') // el-dialog-overlay
ns.e('header') // el-dialog__header
ns.m('theme-dark') // el-dialog--theme-dark
ns.be('header','close') // el-dialog-header__close
ns.em('footer','small') // el-dialog__footer--small
ns.bm('footer','small') // el-dialog-footer--small
ns.bem('footer','btn','primary') // el-dialog-footer__btn--primary
ns.is('closeable') // is-closeable
```

ns.b() 返回结果为 "el-dialog "

ms.b('overlay') 

ns.e('header') 返回结果为 "el-dialog__header"

ns.m('theme-dark') 返回结果为 "el-dialog--theme-dark"

ns.be('header','close') 返回结果为 "el-dialog-header__close"  意思为返回一个block + element

ns.em('foter,'small') 返回结果为 "el-dialog__footer--small" 意思为返回一个element + modifier 

ns.bm('footer','small') 返回结果为 "el-dialog-footer--small" 意思为返回一个block + modifier

ns.bem('footer','btn','primary') 返回结果为" el-dialog-footer__btn--primary" 意思为返回一个block+element+modifer

ns.is('closeable') 返回is-closeable 通常用来描述组件的状态，如is-closeable  表示：是否显示关闭按钮



知道了useNameSpace的用法下面我们来改造dialog.vue的html模板

**改造前的代码**

```html
<template>
    <div class="dialog-overlay" v-if="visible">
        <section class="dialog">
            <header class="dialog__header">
                <div class="dialog__title">{{ title }}</div>
                <span class="dialog-header__close" @click="visible=false">X</span>
            </header>
            <div class="dialog__body">
                {{ message ?? 'BEM是一种针对css的前端命名规范，是块(Block)，元素（Element），修饰符（Modifier）的简写。' }}
            </div>
            <footer class="dialog__footer">
                <button class="dialog__btn">{{ cancelBtnTxt }}</button>
                <button class="dialog__btn dialog__btn--primary">{{ comfirmBtnTxt }}</button>
            </footer>
        </section>
    </div>
</template>
```

**改造后的代码**

```vue
<template>
    <div :class="ns.b('overlay')" v-if="visible">
        <section :class="ns.b()">
            <header :class="ns.e('header')">
                <div :class="ns.e('title')">{{ title }}</div>
                <span :class="ns.be('header','close')" @click="visible=false">X</span>
            </header>
            <div :class="ns.e('body')">
                {{ message ?? 'BEM是一种针对css的前端命名规范，是块(Block)，元素（Element），修饰符（Modifier）的简写。' }}
            </div>
            <footer :class="ns.e('footer')">
                <button :class="ns.e('btn')">{{ cancelBtnTxt }}</button>
                <button :class="[ns.e('btn'), ns.em('btn','primary')]">{{ comfirmBtnTxt }}</button>
            </footer>
        </section>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useNamespace } from '../composables/useNamespace'
const ns = useNamespace('dialog')
const visible = ref(true)
const close = ()=>{
    visible.value = false
}
</script>
```

scss也需要改造，新增了一个$nameSpace命名空间

```scss
<style scoped lang="scss">
:global(body){
    font-family: PingFang SC,Microsoft YaHei,Arial,sans-serif;
}
$namespace: 'poliwen';
.#{$namespace}-dialog-overlay{
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 2000;
    height: 100%;
    background-color:rgba(0, 0, 0, .5);
    overflow: auto;
}

.#{$namespace}-dialog {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    border-radius:2px;
    background: #fff;
    width: 559px;
    min-height: 200px;
    &__header{
        padding: 20px 20px 10px;
    }
    &__title{
        color: #333;
        font-size: 18px;
    }
    &__body{
        padding: 30px 20px;
        font-size: 14px;
        color: #333;
    }
    &-header{
        &__close{
            position: absolute;
            right: 15px;
            top: 15px;
            color: #999;
            font-size: 18px;
        }
    }
    &__btn{
        min-width: 33px;
        padding: 8px 15px;
        background: none;
        border: 1px solid #ddd;
        border-radius: 2px;
        color: #666;
        margin-left: 10px;
        cursor: pointer;
        &--primary{
            background: #409eff;
            color: #fff;
            border-color:#409eff;
        }
    }
    &__footer{
        padding: 10px 20px 20px;
        display: flex;
        justify-content: flex-end;
    }
}
</style>
```

我们还可以将BEM的命名封装成scss的mixin方法

```scss
$namespace: 'poliwen';
$common-separator: '-';
$element-separator: '__';
$modifier-separator: '--';
$state-prefix: 'is-';
$namespace: 'el';
// BEM support Func
@function selectorToString($selector) {
  $selector: inspect($selector);
  $selector: str-slice($selector, 2, -2);
  @return $selector;
}

@function containsModifier($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, $modifier-separator) {
    @return true;
  } @else {
    @return false;
  }
}

@function containWhenFlag($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, '.' + $state-prefix) {
    @return true;
  } @else {
    @return false;
  }
}

@function containPseudoClass($selector) {
  $selector: selectorToString($selector);

  @if str-index($selector, ':') {
    @return true;
  } @else {
    @return false;
  }
}

@function hitAllSpecialNestRule($selector) {
  @return containsModifier($selector) or containWhenFlag($selector) or
    containPseudoClass($selector);
}


// bem('block', 'element', 'modifier') => 't5-block__element--modifier'
@function bem($block, $element: '', $modifier: '') {
  $name: $namespace + $common-separator + $block;

  @if $element != '' {
    $name: $name + $element-separator + $element;
  }

  @if $modifier != '' {
    $name: $name + $modifier-separator + $modifier;
  }

  // @debug $name;
  @return $name;
}

// BEM
@mixin b($block) {
  $B: $namespace + "-" + $block !global;

  .#{$B} {
    @content;
  }
}

@mixin e($element) {
  $E: $element !global;
  $selector: &;
  $currentSelector: "";
  @each $unit in $element {
    $currentSelector: #{$currentSelector + "." + $B + $element-separator + $unit + ","};
  }

  @if hitAllSpecialNestRule($selector) {
    @at-root {
      #{$selector} {
        #{$currentSelector} {
          @content;
        }
      }
    }
  } @else {
    @at-root {
      #{$currentSelector} {
        @content;
      }
    }
  }
}

@mixin m($modifier) {
  $selector: &;
  $currentSelector: "";
  @each $unit in $modifier {
    $currentSelector: #{$currentSelector + $selector + $modifier-separator + $unit + ","};
  }

  @at-root {
    #{$currentSelector} {
      @content;
    }
  }
}
```

以上代码稍稍有点复杂，感兴趣的可以去学习下scss的高级用法

定义好mixins方法之后，我们最终的样式代码如下

```scss
<style scoped lang="scss">
@import "../assets/mixins.scss";
:global(body){
    font-family: PingFang SC,Microsoft YaHei,Arial,sans-serif;
}
@include b(dialog) {
    &-overlay{
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        z-index: 2000;
        height: 100%;
        background-color:rgba(0, 0, 0, .5);
        overflow: auto;
    }
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    border-radius:2px;
    background: #fff;
    width: 559px;
    min-height: 200px;
    @include e(header){
        padding: 20px 20px 10px;
    }
    @include e(title){
        color: #333;
        font-size: 18px;
    }
    @include e(body){
        padding: 30px 20px;
        font-size: 14px;
        color: #333;
    }
    &-header{
        &__close{
            position: absolute;
            right: 15px;
            top: 15px;
            color: #999;
            font-size: 18px;
        }
    }
    @include e(btn){
        min-width: 33px;
        padding: 8px 15px;
        background: none;
        border: 1px solid #ddd;
        border-radius: 2px;
        color: #666;
        margin-left: 10px;
        cursor: pointer;
        &--primary{
            background: #409eff;
            color: #fff;
            border-color:#409eff;
        }
    }
    @include e(footer){
        padding: 10px 20px 20px;
        display: flex;
        justify-content: flex-end;
    }
}
</style>
```

### 总结：

BEM样式命名是各大主流组件库的常用命名方法，因此掌握它的使用很有必要，本文主要结合elementPlus的源码分析了BEM的妙用，通过定义一个useNameSpace工具库，来生成符合BEM命名规范的方法，使用scss编写符合BEM命名的mixins，在组件中结合useNameSpace和minxins就能很方便的使用BEM命名大法。

本文参考资料：https://github.com/element-plus/element-plus

文中案例代码请访问：https://github.com/PoliWen/myBlog

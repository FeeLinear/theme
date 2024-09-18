# 主题切换

## 一、主题切换方案
## 二、KOCA框架的主题切换实现原理
## 三、基于KOCA项目主题切换开发

### --------- 主题切换方案 ---------

方案|优点|缺点
-|:-:|:-
动态引入/删除link标签|实现简单|工作量大，需要预写好，各种主题样式文件，用户无法自定义主题|
类名切换|实现简单|工作量大，维护困难，需要预写好，各种主题类名对应不同的样式，用户无法自定义主题，性能差|
CSS变量|易维护，扩展性强，可动态自定义|可读性较差，页面个性化颜色过多时需要使用大量的颜色变量
SASS变量|灵活，维护成本低|需要预编译，编译后代码量大，无法动态自定义

<br /><br />

#### --------- CSS变量简介 ---------

CSS变量也叫自定义的属性，定义CSS变量需要使用 <font color="red">“--”</font> 开头，且只能使用<font color="green"> 英文、数字、破折号以及下划线</font> 组成

2、CSS变量的作用域跟JS 变量(var)作用域一样，即在定义CSS变量的元素内部有效，在定义CSS变量的元素外部无效。

3、改变CSS变量：dom.style.setProperty(prop, val);

```html
<html>
  <head>
    <style>
      :root {
        --primary-color: #3079FF;
      }
      .box {
        width: 100px;
        height: 100px;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <div class="box" style="background-color: var(--primary-color);"></div>
    </div>
  </body>
  <script>
    document.querySelector('#app').addEventListener('click', function () {
      document.querySelector('.box').style.setProperty('--primary-color', 'red');
    });
  </script>
</html>
```

<br /><br />

### --------- KOCA框架的主题切换实现原理 ---------

主要分为<mark>三个模块</mark>、<mark>一个主题色</mark>及<mark>所有组件的黑白文字颜色</mark>

KOCA框架主题控制工具方法：
```js
import { baseHandler, handler } from "szkingdom.yf.koca-template/lib/components/Settings/handler.ts";
```
模块|组件名|颜色变量|改变颜色方法
-|:-:|:-:|:-
顶部导航栏|FrameTop|--topbar-bgcolor|handler(HandlerEnum.HEADER_THEME, color)
侧边菜单栏|SideMenu|--sidebar-bgcolor|handler(HandlerEnum.MENU_THEME, color)
页面内容区域|子页面路由|--bgcolor-primary|当用户切换主题时，如果主题的key存在，则框架会自动加载{key}-theme-style.css文件，如果需要设置该变量的值已达到更改内容背景色的效果：见public/assets/dark-theme-style.css文件
主题色替换|--|--color-primary|koca框架是使用了webpack插件实现:webpack-theme-color-replacer 项目启动/打包时，会执行new ThemeColorReplacer，事先创建了 一个css样式文件存储在NG;该样式文件包含element-ui所有跟主题颜色相关的样式；切换主题时，对应的主题颜色会相应发生改变，框架会通过ajax请求事先创建好的css样式文件，然后根据正则表达式动态替换掉主题颜色相关的颜色，然后动态创建style标签追加至body；
黑白文字颜色|所有组件|--|同第三行：页面内容区域
```scss
// public/assets/blue-theme-style.css
[data-theme="dark"]:root {
    --color-primary-light-1: #3079FF1A;
    --color-primary-light-2: #3079FF33;
    --color-primary-light-3: #3079FF4D;
    --color-primary-light-4: #3079FF66;
    --color-primary-light-5: #3079FF80;
    --color-primary-light-6: #3079FF99;
    --color-primary-light-7: #3079FFB3;
    --color-primary-light-8: #3079FFCC;
    --color-primary-light-9: #3079FFE6;
    --bgcolor-primary: #1c1c1c;
    --bgcolor-primary-light: var(--color-primary-light-4);
}
[data-theme="dark"] input {
  background-color: #000000;
  border-color: #4a4d57;
  color: #f1f1f1;
}
```


因此，主题切换的开发工作主要是以上五项，再加上二级页面写死颜色样式；



### --------- 基于KOCA主题切换项目开发 ---------

##### 1、新增主题：
在src/config/settings.ts文件skins配置项中新增主题
```js
skins: {
    // 亮色模式
    light: [
      { title: "经典蓝", color: "#3261ff", key: null }, // key: null, // null 无需加载皮肤文件
      { title: "商务红", color: "#E13C39", key: null },
      { title: "雅致青", color: "#5ABFC1", key: null },
    ],
    // 暗色模式
    dark: [
      { title: "深邃蓝", color: "#3079FF", key: "dark" },
      { title: "亮彩蓝", color: "#3365FF", key: "blue" },
      { title: "亮彩红", color: "#FF0000", key: null },
      { title: "活力橙", color: "#ff6600", key: "orange" },
    ],
  }
```

<font style="color:red">settings.ts文件的frameUrlList.queryPanelConfig 配置需清空，否则本地配置好又被接口配置覆盖；导致不生效</font>

<br />

##### 2、新增皮肤样式文件设置内容区域背景颜色变量的值(--bgcolor-primary)：
在public/assets目录下新增orange-theme-style.css皮肤文件（黑夜模式）:
黑夜模式，对应elment-ui/koca-ui组件的黑色/灰色文字颜色，需要修改为白色/白灰色（直接copy dark-theme-style.css文件即可修改主题名，具体见文件内容）；

<br />

##### 3、安装并配置webpack-theme-color-replacer插件：

```js
npm i --save-dev webpack-theme-color-replacer@1.3.26
```

```js
// vue.config.js
const ThemeColorReplacer = require("webpack-theme-color-replacer");
const forElementUI = require('webpack-theme-color-replacer/forElementUI');
const changeSelector = require("./build/webpack-theme-color-replacer/changeSelector");
...
plugins: [
  new ThemeColorReplacer({
    fileName: 'css/theme-colors.[contenthash:8].css',
    matchColors: [
      ...forElementUI.getElementUISeries(themeColor)
    ],
    // 修改原来的changeSelector方法，fix el-tabs,el-select下的el-tags，el-radio等组件的样式优先级出现问题
    // changeSelector: forElementUI.changeSelector,
    changeSelector,
    isJsUgly: isPro
  }),
]

```

<br />

##### 4、框架顶栏及侧边菜单栏（若为自写组件，可跳过此步骤）


切换主题时，KOCA框架写死了三种主题对应的顶栏及侧边菜单栏的颜色变量的自动更新：
```js
export function updateTopbarBgColor(color?: string, darkMode: ThemeEnum | string = settings$.darkMode) {
  if (!color) {
    if (darkMode === ThemeEnum.DARK) {
      color = variables.bgColorDark;
    } else if (darkMode === ThemeEnum.BLUE) {
      color = variables.bgColorBlue;
    } else if (darkMode === ThemeEnum.LIGHT) {
      color = settings$.layout === "sidebar" ? variables.bgColorLight : variables.bgColorDark;
    }
  }
  if (color) {
    const hoverColor = lighten(color!, 6);
    // bg color
    setCssVar(CssVarEnum.TOPBAR_BGCOLOR_VAR, color);
    setCssVar(CssVarEnum.TOPBAR_MENU_ACTIVE_BGCOLOR_VAR, hoverColor);
    setCssVar(CssVarEnum.TOPBAR_ACTIVE_BGCOLOR_VAR, hoverColor);


    const isDark = colorIsDark(color!);
    settings$.setTopbarSetting({ theme: isDark ? ThemeEnum.DARK : ThemeEnum.LIGHT });

  }
}

export function updateSidebarBgColor(color?: string, darkMode: ThemeEnum | string = settings$.darkMode) {
  if (!color) {
    if (darkMode === ThemeEnum.DARK) {
      color = variables.bgColorDark;
    } else if (darkMode === ThemeEnum.BLUE) {
      color = variables.bgColorBlue;
    } else if (darkMode === ThemeEnum.LIGHT) {
      color = settings$.layout === "sidebar" ? variables.bgColorDark : variables.bgColorLight;
    }
  }

  if (color) {
    // bg color
    setCssVar(CssVarEnum.SIDEBAR_BGCOLOR_VAR, color);

    const isDark = colorIsDark(color!);

    settings$.setMenuSetting({ theme: isDark ? ThemeEnum.DARK : ThemeEnum.LIGHT });
  }
}

```

因此，切换自定义主题时， 需要手动调用以下两个更新顶栏及侧边菜单栏的颜色变量的方法：
在src/assets/scss/layout-variables.scss 文件新增对应主题的顶部/左侧菜单的背景色变量：$bg-color-orange: #3b1901;然后在export-var.scss文件导出
```scss
// layout-variables.scss
$bg-color-orange: #3b1901

// export-var.scss

:export {
  ...
  bgColorBlue: $bg-color-blue;
  bgColorOrange: $bg-color-orange;
}
```
```js
// "szkingdom.yf.koca-template/lib/components/Settings/handler.ts";
case HandlerEnum.HEADER_THEME:
      updateTopbarBgColor(value);
      settings$.setTopbarSetting({ bgColor: value }); break;
case HandlerEnum.MENU_THEME:
  updateSidebarBgColor(value);
  settings$.setMenuSetting({ bgColor: value }); break;


// 调用方法：
import { handler } from 'szkingdom.yf.koca-template/lib/components/Settings/handler.ts';
import { HandlerEnum } from 'szkingdom.yf.koca-template/lib/components/Settings/enum.ts';
import variables from "scss/mixin/layout-variables.scss";
handler(HandlerEnum.HEADER_THEME, variables.bgColorOrange);
handler(HandlerEnum.MENU_THEME, variables.bgColorOrange);
```
<font style="color:red">调用更新顶栏侧边菜单栏方法时，方法内部会根据颜色计算是黑夜还是白天模式，从而设置不同的class类名，改变文字颜色</font>

<br />

##### 5、二级页面的修改

###### 5.1 文字颜色
创建变量基本样式文件base-theme-style.css文件，在index.scss文件引入。
```scss
/ *文件内容 */
:root {
  --text-primary-color: #333;
  --gray-color1: #333;
  --gray-color2: #666;
  --gray-color3: #999;
  --gray-color4: #ddd;
  --gray-color5: #f2f2f2;
  --bg-gray-color1: #f3f5f7;
  ...
}
```
将二级页面内的颜色替换成对应的变量；如 color: #333; 替换成： color: var(--text-primary-color);
###### 5.2 背景颜色
背景颜色需要替换成透明度颜色，这样才能适配黑夜模式的主题；
5.2.1 在theme-variables.scss 文件新建常用颜色：
```scss
// src/assets/scss/mixin/theme-variables.scss
$--blue-color: #0066ff; // 蓝
$--red-color: #F04730; // 红
$--green-color: #50BC19; // 绿
$--orange-color: #F09A03; // 橙
```
5.2.2 将二级页面的颜色以常用颜色+透明度的方式替换：
```scss
.item-1 {
  // background: #fef6f4;
  background: rgba($--red-color, 0.05);
}
```
###### 5.3 图片（svg, 其它）
5.3.1 在src/assets/mixin/文件夹下创建theme.scss文件
```scss
// theme.scss
$themes: (
  dark: (
    testColor: red
  ),
  light: (
    testColor: blue
  ),
  orange: (
    testColor: orange
  )
);
$curTheme: light;
@mixin useTheme() {
  @each $key, $value in $themes {
    $curTheme: $key !global;
    [data-theme='#{$key}'] & {
      @content;
    }
  }
}

@mixin themeImage($name, $preSrc: '/assets/images/themes/') {
  @each $key, $value in $themes {
    $curTheme: $key !global;
    [data-theme='#{$key}'] & {
      background-image: url(#{$preSrc}#{$key}/#{$name});
      @content;
    }
  }
}
@mixin themeBgImage($name, $preSrc: '/assets/images/themes/') {
  @each $key, $value in $themes {
    $curTheme: $key !global;
    [data-theme='#{$key}'] & {
      background: url(#{$preSrc}#{$key}/#{$name}) no-repeat center /100% 100%;
      @content;
    }
  }
}
@function getVar($key) {
  $themeMap: mag-get($themes, $curTheme);
  @return map-get($themeMap, $key);
}
```
5.3.2 在vue.config.js文件添加配置：
```js
css: {
    loaderOptions: {
      scss: {
        sourceMap: false,
        prependData: `
          @import "scss/mixin/layout-variables.scss";
          @import "scss/mixin/_utils.scss";
          @import "scss/mixin/theme.scss";
          `
      }
    },
  },
```
5.3.3 将各主题对应的图片文件放至到public/assets/images/themes/{theme}/目录下：
```
public/assets/images/themes/light/basic-img-none2.svg
public/assets/images/themes/dark/basic-img-none2.svg
public/assets/images/themes/orange/basic-img-none2.svg
```
5.3.4 使用:
```scss
.item {
  @include themeImage('basic-img-none2.svg');
}
// 以orange主题为例，编译后⬇⬇
.item {
  background-image: url('/assets/images/themes/orange/basic-img-none2.svg');
}

// =======================

.item {
  @include themeBgImage('basic-img-none2.svg');
}
// 以orange主题为例，编译后⬇⬇
.item {
  background: url('/assets/images/themes/orange/basic-img-none2.svg') no-repeat center /100% 100%;
}
```

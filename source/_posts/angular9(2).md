---
title: 使用Angular Library开发组件库
---
## Angular9 组件开发（二）
——使用Angular Library开发组件库

### 创建项目
###### 创建一个angular项目，这里不多做赘述
```
ng new uos-ui
```
###### 创建Library项目
```
ng g library sf-lib --prefix=dp
```
--prefix参数用于指定项目中各组件的名称前缀。
此时生成的项目结构如下：
```
projects
├── deepin-ui
│   └── src
│       ├── lib
│       |   ├── deepin-ui.component.html
│       |   ├── deepin-ui.component.scss
│       |   ├── deepin-ui.component.ts
│       |   ├── deepin-ui.module.ts
│       |   └── deepin-ui.service.ts
│       ├── public-api.ts
│       └── test.ts
```
同时还会在根项目的angular.json以及tsconfig.json中看到新增的项目信息配置。
### 添加组件
添加组件的方法和angular中基本一致，只是需要指定要添加到的位置：
```
ng g component checkbox --project=deepin-ui
```
执行成功后，目录结构如下：
```
projects
├── deepin-ui
│   └── src
│       ├── lib
│       |   ├── checkbox
│       |   |   ├── checkbox.component.html
│       |   |   ├── checkbox.component.scss
│       |   |   └── checkbox.component.ts
│       |   ├── deepin-ui.component.html
│       |   ├── deepin-ui.component.scss
│       |   ├── deepin-ui.component.ts
│       |   ├── deepin-ui.module.ts
│       |   └── deepin-ui.service.ts
│       ├── public-api.ts
│       └── test.ts
```
checkbox的源码请参考[上一期](https://docsin.uniontech.com/?p=5881 "angular9 组件开发（一）")
此时组件已经添加完成，但要在外部使用还需要执行两步：
###### 在module中导出：
```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

import {CheckboxComponent} from './checkbox/checkbox.component';

const components = [
  CheckboxComponent
]
const modules = [
  FormsModule,ReactiveFormsModule,CommonModule
]

@NgModule({
  declarations: [...components],
  imports: [
    ...modules
  ],
  exports: [...components,...modules]
})
export class DeepinUiModule { }

```
###### 在public-api中导出
```
/*
 * Public API Surface of deepin-ui
 */

export * from './lib/deepin-ui.service';
export * from './lib/deepin-ui.module';
export * from './lib/checkbox/checkbox.component';
```
到此，仓库可以正常的打包编译，新加的组件也能使用了。

### 编译运行
###### 编译
```
ng build --prod deepin-ui
```
若此处报错私有组件无法导出，请检查上一步public-api中，是否有未添加的组件或模块。
编译成功后会在根目录生成一个dist文件夹，这时已经可以在根项目中导入这个模块并使用了。
找到app.module.ts，代码如下：
```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import {DeepinUiModule} from 'deepin-ui'

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,DeepinUiModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
###### 在app.component.html中使用包中的模块
```
<dp-checkbox></dp-checkbox>
```
需要注意的是：如果本地library中有改动，必须要先build之后才能生效。
### 推送到npm仓库
开发包的目的是为了让更多的人或项目能够使用到，以节省开发时间，提高工作效率。
npm为当前最主流的javascript包管理工具，把开发好的包发布上去，就能让全世界的开发者都拿到并使用。
###### 注册账号：
在[npm.js](https://www.npmjs.com/package/ng-clone-deep "npm.js")上注册账号。
###### 登录：
```
npm adduser
```
验证登录：
```
npm whoami
```
如果输出你的用户名，则代表登录成功！
###### 发布：
在你编译后的目录下（dist/deepin-ui)
```
npm publish
```
显示如下：
```
npm notice 
npm notice 📦  deepin-ui@0.0.1
npm notice === Tarball Contents === 
npm notice 8.0kB  esm2015/lib/checkbox/checkbox.component.js
npm notice 8.3kB  esm5/lib/checkbox/checkbox.component.js   
npm notice 468B   esm2015/deepin-ui.js                      
npm notice 468B   esm5/deepin-ui.js                         
npm notice 5.6kB  fesm2015/deepin-ui.js                     
npm notice 6.1kB  fesm5/deepin-ui.js                        
npm notice 2.3kB  esm2015/lib/deepin-ui.module.js           
npm notice 2.4kB  esm5/lib/deepin-ui.module.js              
npm notice 1.2kB  esm2015/lib/deepin-ui.service.js          
npm notice 1.3kB  esm5/lib/deepin-ui.service.js             
npm notice 17.6kB bundles/deepin-ui.umd.js                  
npm notice 6.1kB  bundles/deepin-ui.umd.min.js              
npm notice 764B   esm2015/public-api.js                     
npm notice 764B   esm5/public-api.js                        
npm notice 5.7kB  deepin-ui.metadata.json                   
npm notice 684B   package.json                              
npm notice 3.8kB  fesm2015/deepin-ui.js.map                 
npm notice 3.8kB  fesm5/deepin-ui.js.map                    
npm notice 29.4kB bundles/deepin-ui.umd.js.map              
npm notice 15.6kB bundles/deepin-ui.umd.min.js.map          
npm notice 1.0kB  README.md                                 
npm notice 720B   lib/checkbox/checkbox.component.d.ts      
npm notice 78B    deepin-ui.d.ts                            
npm notice 40B    lib/deepin-ui.module.d.ts                 
npm notice 60B    lib/deepin-ui.service.d.ts                
npm notice 132B   public-api.d.ts                           
npm notice === Tarball Details === 
npm notice name:          deepin-ui                               
npm notice version:       0.0.1                                   
npm notice package size:  26.0 kB                                 
npm notice unpacked size: 122.3 kB                                
npm notice shasum:        7bcb4f7bea9adfbd25bba0c4bf37a2a76edacad0
npm notice integrity:     sha512-650EchBqsEeTc[...]/k68l/vhaK/Mg==
npm notice total files:   26                                      
npm notice 
+ deepin-ui@0.0.1
```
代表发布成功！
这里需要注意的是，如果使用非官方源或者私有源，需要有管理员权限才可以添加，否则会报403，这里建议使用官方源。
[源的切换](https://blog.csdn.net/z277094641/article/details/90214214 "源的切换")
### 引用云端依赖
```
npm install deepin-ui
```
与平常无异
### 总结
npm的发布我们已经很熟悉了，但要开发专属angular的包，打包编译会变得特别困难。好在angular官方提供了解决方案（[enjoy](https://github.com/aaaa9990aaaa/uos-ui "gayhub")）

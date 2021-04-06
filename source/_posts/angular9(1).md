---
title: angular9封装CheckBoxGroup组件
---
# angular9组件开发（一） 
## 组件开发入门之实现CheckBoxGroup

### 一、双向绑定
&emsp;1.双向绑定是MVVM框架的核心组成部分(
```
view->model model->view
```
)
在表单中，我们常常能看到诸如：
```
<input [(ngModel)]="data" />
```
这种写法在三大框架里面都很常见~~比如v-model等~~  
&emsp;2.尽管原生表单控件与自定义控件的实现方式不同，但表达的思想确是一样的。angular中双向绑定有至少三种实现方式，这里暂时只讲表单组件。
&emsp;了解双向绑定，等于完成了组件开发的第一步。
### 二、angular中表单组件双向绑定的实现
&emsp;1.引入FormsModule模块
```
import { FormsModule } from '@angular/forms';
...
@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
    FormsModule
  ]
})
```
2.注册NG_VALUE_ACCESSOR服务
```
@Component({
  selector: 'app-checkbox',
  templateUrl: './checkbox.component.html',
  styleUrls: ['./checkbox.component.scss'],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CheckboxComponent),
      multi: true
    }
  ]
})
```
3.实现ControlValueAccessor接口
```
export interface ControlValueAccessor {
  writeValue(obj: any): void;
  registerOnChange(fn: any): void;
  registerOnTouched(fn: any): void;
  setDisabledState?(isDisabled: boolean): void;
}
```
writeValue(obj: any)：该方法用于将模型中的新值写入视图或 DOM 属性中。

registerOnChange(fn: any)：设置当控件接收到 change 事件后，调用的函数

registerOnTouched(fn: any)：设置当控件接收到 touched 事件后，调用的函数

setDisabledState?(isDisabled: boolean)：当控件状态变成 DISABLED 或从 DISABLED 状态变化成 ENABLE 状态时，会调用该函数。该函数会根据参数值，启用或禁用指定的 DOM 元素。

### 三、简单实例（Checkbox)
1.checkbox.component.html
```
<div class="checkbox" [ngClass]="{disabled: config.disabled,circle:config.circle}">
  <input type="checkbox" [disabled]="config.disabled" [(ngModel)]="checked" (ngModelChange)="change(checked)" />
  <label [class]="config.type||'primary'">
    <i [class.dstore-checked]="checked"></i>
    <span *ngIf="config.label != ''" >{{config.label}}</span>
  </label>
</div>
```
2.checkbox.component.scss
```
:host {
  display: inline-block;
}

.checkbox {
  line-height: 1;
  position: relative;
  >input {
    position: absolute;
    z-index: 99999999;
    top: 0;
    left: 0;
    display: block;
    width: 100%;
    height: 100%;
    margin: 0;
    cursor: pointer;
    opacity: 0;
  }
  i {
    border: 1px solid #888;
    background-color: rgba(255, 255, 255, 0.3);
    display: inline-block;
    vertical-align: top;
    //border-radius: 0.4rem;
    font-size: 1rem;
    width: 1.3rem;
    height: 1.3rem;
    position: relative;
    box-sizing: border-box;
    transition: background-color .15s ease-in .05s;
    &::after {
      box-sizing: content-box;
      content: "";
      border: 1px solid #fff;
      border-left: 0;
      border-top: 0;
      height: 0.7rem;
      left: 0.3rem;
      position: absolute;
      top: 1px;
      transform: rotate(45deg) scaleY(0);
      width: 0.4rem;
      transition: transform .15s ease-in .05s;
      transform-origin: center;
    }
  }
  label {
    line-height: 1.3rem;
    vertical-align: top;
    font-weight: normal;
    font-size: 0.92rem;
    span {
      margin-left: 0.5rem;
    }
  }
  //各种checkbox的样式
  .primary {
    .dstore-checked {
      background-color: #0081ff;
      border-color: #0081ff;
      &::after {
        transform: rotate(45deg) scaleY(1);
      }
    }
  }
  .success {
    .dstore-checked {
      background-color: #5cb85c;
      border-color: #5cb85c;
      &::after {
        transform: rotate(45deg) scaleY(1);
      }
    }
  }
  .warning {
    .dstore-checked {
      background-color: #f0ad4e;
      border-color: #f0ad4e;
      &::after {
        transform: rotate(45deg) scaleY(1);
      }
    }
  }
  .info {
    .dstore-checked {
      background-color: #5bc0de;
      border-color: #5bc0de;
      &::after {
        transform: rotate(45deg) scaleY(1);
      }
    }
  }
  .danger {
    .dstore-checked {
      background-color: #d9534f;
      border-color: #d9534f;
      &::after {
        transform: rotate(45deg) scaleY(1);
      }
    }
  }
}

.disabled {
  input {
    cursor: not-allowed !important;
  }
  i {
    background-color: #ccc;
  }
}

.circle {
  i {
    border-radius: 50%;
  }
}
```
3.checkebox.coponent.ts
```
import { Component, forwardRef, Input } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-checkbox',
  templateUrl: './checkbox.component.html',
  styleUrls: ['./checkbox.component.scss'],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CheckboxComponent),
      multi: true
    }
  ]
})
export class CheckboxComponent implements ControlValueAccessor {

  constructor() { }

  @Input()
  config: CheckboxConfig;
  
  checked:boolean

  //初始化一个空方法，不需要任何实现
  change = (value: any) => {
  };

  writeValue(obj: any): void {
    if(obj != null && obj != undefined) {
      this.checked = obj
      //this.change(this.checked);
    }
  }
  registerOnChange(fn: any): void {
    //将接口传过来的方法赋予change，用于实现 model=>view
    this.change = fn;
  }
  registerOnTouched(fn: any): void {
  }
  setDisabledState?(isDisabled: boolean): void {
    
  }
}

export enum CheckboxType {
  primary = 'primary',
  success = 'success',
  warning = 'warning',
  info = 'info',
  danger = 'danger'
}

export interface CheckboxConfig {
  label:string,
  value:any,
  type?:CheckboxType,
  circle?: boolean,
  disabled?: boolean,
  data1?:any //预留字段
}
```
4.使用：
```
config:CheckboxConfig = {
      type: CheckboxType.info,
      circle:true,
      label: 'info',
      value: 2
    }
...
<app-checkbox [config]="config" [(ngModel)]="checkeds" (ngModelChange)="checkBoxChange()"></app-checkbox>
```
### 四、CheckboxGroup
再稍微引申下，写个简单的CheckboxGroup
1.checkbox-group.component.html
```
<ng-container *ngFor="let item of config.list;let i = index">
  <div [class]="config.align||'row'">
    <app-checkbox [config]="item" [(ngModel)]="checkedList[i]" (ngModelChange)="change(checkedList)"></app-checkbox>
  </div>
</ng-container>
```
2.checkbox-group.component.scss
```
:host {
  .row {
    margin-right: 10px;
    display: inline-block;
  }
  .column {
    display: block;
    margin: 1rem 0;
  }
}
```
3.checkbox-group.component.ts
```
import { Component, forwardRef, Input } from '@angular/core';
import { CheckboxType, CheckboxConfig } from '../../checkbox/checkbox/checkbox.component';
import { NG_VALUE_ACCESSOR, ControlValueAccessor } from '@angular/forms';

@Component({
  selector: 'app-checkbox-group',
  templateUrl: './checkbox-group.component.html',
  styleUrls: ['./checkbox-group.component.scss'],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CheckboxGroupComponent),
      multi: true
    }
  ]
})
export class CheckboxGroupComponent implements ControlValueAccessor {
 
  constructor() { }

  @Input()
  config:CheckboxGroupConf;

  checkedList:boolean[] = []

  change = (val:any)=>{}

  writeValue(obj: any): void {
    if(obj) {
      this.checkedList = obj
    }
  }
  registerOnChange(fn: any): void {
    this.change = fn;
  }
  registerOnTouched(fn: any): void {
    //throw new Error("Method not implemented.");
  }
  setDisabledState?(isDisabled: boolean): void {
    //throw new Error("Method not implemented.");
  }

}

export interface CheckboxGroupConf {
  align?:CheckboxAlignType,
  list:CheckboxConfig[]
}

export enum CheckboxAlignType {
  column = 'column',
  row = 'row'
}
```
4.使用：
```
<div>
  <app-checkbox-group [config]="config" [(ngModel)]="checkeds" (ngModelChange)="checkBoxChange()"></app-checkbox-group>
</div>
...
config: CheckboxGroupConf = {
    //align: CheckboxAlignType.row,
    list:[{
      type: CheckboxType.primary,
      label: 'primary',
      value: 1
    },{
      type: CheckboxType.info,
      circle:true,
      label: 'info',
      value: 2
    },{
      type: CheckboxType.warning,
      label: 'warning',
      value: 3
    },{
      circle:true,
      type: CheckboxType.danger,
      label: 'danger',
      value: 3
    }]
  };
```
5.预览：
[![演示](演示 "演示")](https://129.226.49.225/images/01.png "演示")
### 五、总结
上手angular一个月了，因为想要延续以前通用组件化开发的思路，于是就有了这一篇入门笔记。不对的地方欢迎大佬们指正。
源码地址：
[gayhub](https://github.com/aaaa9990aaaa/ng-checkbox-group "gayhub")
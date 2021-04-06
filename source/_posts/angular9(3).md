---
title: 使用Angular开发下拉滚动条组件（仿macos）
---
### 前言
&emsp;写这篇文章是因为在工作中，使用第三方插件遇到性能问题，在剔除插件后发现浏览器自带的滚动条，加上自定义的样式能实现的效果非常有限…… 于是想到自己实现一个简易的滚动条组件，以满足基本的功能需求。

### 需求
&emsp;1.滚动时显示，不滚动时延时一秒隐藏。
&emsp;2.鼠标移动到右侧时显示，加宽。
&emsp;3.支持拖动。
&emsp;4.支持点击快速跳转。

### 实现
&emsp;基本的思路有两点：利用浏览器的scroll事件，来移动滚动条；监听组件内元素的变化来更新滚动条的状态。

#### 1.布局
```
<div #scrollBoxRef class="scroll_box" (scroll)="scroll($event)" [ngClass]="isMac()?'mac_scroll':'default_scroll'">
  <div class="scroll_content" [class.full]="full">
      <ng-content></ng-content>
  </div>
  <div (mousedown)="clickY($event)" (mousemove)="moveY($event)" class="scroll_y">
    <div (dragstart)="disableEvent()" (mousedown)="clickBar($event)" class="scroll_bar"></div>
  </div>
</div>
```
```
 $scroll_box //滚动元素
 $scroll_content //内容容器
 $scroll_Y //滚动条容器（Y轴）
 $scroll_bar //滚动条
```
```
:host {
  height: 100%;
  width: 100%;
  position: relative;//保证滚动条浮动基准
  display: block;
  overflow: hidden;
  scrollbar-width: none;
  .scroll_box{
    height: 100%;
    overflow: auto;
	//隐藏chrome、safari滚动条
    &::-webkit-scrollbar {
      display: none;
    }
	//隐藏firefox滚动条
    scrollbar-width: none;
	//隐藏ie滚动条
    -ms-overflow-style:none;
    .scroll_content {
      width: 100%;
      min-height: 98%;
      overflow:hidden;
    }
    .scroll_y {
      position: absolute;
      right: 2px;
      top: 2px;
      bottom: 2px;
      width: 6px;
      border-radius: 3px;
      opacity: 0;
      transition: opacity 120ms ease-out;
      &:hover {
        border-radius: 5px;
        width: 10px;
        opacity: 1 !important;
      }
      .scroll_bar {
        width: 100%;
        position: absolute;
        right:0;
        display: block;
        cursor: pointer;
        border-radius: inherit;
        background-color: rgb(171, 171, 171, 0.8);
        transition: .05s all linear;
        &:hover {
          width:10px;
          background-color: rgb(153, 153, 153, 0.8);
        }
        &:active {
          background-color: rgb(102, 102, 102, 0.8);
        }
      }
    }
  }
}

//兼容弹性布局
:host.flex {
  flex:1;
  height:auto;
  width: auto;
  display: flex;
  .scroll_box {
    height: auto;
  }
}
```
#### 2.初始化滚动条的长度
```
let height = (this.$scroll_box.clientHeight * 100 / this.$scroll_box.scrollHeight);
    this.$scroll_bar.style.height = height+"%";
    this.$scroll_Y.style.display = this.$scroll_box.scrollHeight>this.$scroll_box.clientHeight?"block":"none"
```

#### 3.滚动时计算滚动条的位置
scroll事件具体实现如下：
```
const moveY = (_this.$scroll_box.scrollTop*100/_this.$scroll_box.clientHeight)
  _this.$scroll_bar.style.transform = "translateY("+moveY+"%)";
```

#### 4.点击滚动条空白区域快速跳转
这里分为点击时绑定事件，鼠标松开时解绑事件两步
clickBar：
```
 this.click_y = e.currentTarget.offsetHeight - (e.clientY - e.currentTarget.getBoundingClientRect().top);
    this.$scroll_Y.style.width = "10px";
    this.$scroll_Y.style.borderRadius = "5px";
	startDrag(e);//事件绑定
```
绑定mousemove:
注意：1.这里为了能正确获取this对象，而使用了Renderer2
2.绑定的事件为mousemove事件，需要在滚动条上禁止拖动(dragstart)="disableEvent()"
```
import { Renderer2} from '@angular/core';
startDrag(e){
	e.stopImmediatePropagation();
	this.mousemoveFunc = this.renderer.listen(document,"mousemove",(event)=>{
      this.mouseMove(event);
    });
    this.mouseupFunc = this.renderer.listen(document,"mouseup",(event) => {
      this.mouseUp(event);
    })
	document.onselectstart = undefined;
}
//禁止拖动
disableEvent(){
	return false;
}
```
#### 5.拖动
```
mouseMove(event:MouseEvent){
  const prevPage = _this.click_y;
  if(!prevPage) return;
  const offset = (_this.$scroll_box.getBoundingClientRect().top - event.clientY) * -1
  const bar_position = (_this.$scroll_bar.offsetHeight - _this.click_y);
  const bar_position_Percentage = (offset - bar_position) * 100 / _this.$scroll_box.offsetHeight;
  _this.$scroll_box.scrollTop = bar_position_Percentage * _this.$scroll_box.scrollHeight / 100;
}
```

#### 6.dom元素的resize监听
当组件内容变化时，需要更新滚动条的位置和大小，这里同样分为两步：
1.使用window.onresize,监听窗口大小的变化来更新。
2.使用mutationObserver接口，该功能是DOM3 Events规范的一部分,且符合观察者模式。（[查看文档](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver))。
```
//window.onresize
 onWindowResize(){
    this.onResizeSubscription = fromEvent(window, 'resize')
    .subscribe((event) => {
      this.reloadBar$.next(true)
    });
  }
  //mutationObserver
  const mutationOption = {
	  childList:true,
	  attributes: false,
	  characterData: false,
	  subtree:true,
	  attributeOldValue: false,
	  characterDataOldValue: false
	}
  addResizeListener() {
    this.mutationObserver =  new MutationObserver((mutations,observer)=>{
      this.reloadBar$.next(true)
    })
    this.mutationObserver.observe(this.$scroll_box,mutationOption)
  }
  ngOnInit(){
  //更新滚动条
  this.reloadBarSubscription = this.reloadBar$.pipe(
        debounceTime(150)
      ).subscribe(res=>{
        this.updateBar()
      })
	}
	//取消订阅
	ngOnDestroy(): void {
    if(this.onResizeSubscription) {
      this.onResizeSubscription.unsubscribe()
    }
    if(this.mutationObserver) {
      this.mutationObserver.disconnect()
    }
  }
```

### 总结
至此，一个简易且美观的滚动条组件就制作完成了
查看效果：
[embed]http://129.226.49.225/videos/deepin-app-store_20200729110313.mp4[/embed]
由于时间关系，以及制作的目的仅是为了满足商店客户端的需要，目前还有许多缺陷：比如mac下没有回复默认滚动条、firefox兼容不好、仅支持纵向滚动、组件封装不够完善等。
本篇（系列）旨在分享学习，如果有好的意见或建议欢迎留言，后续会在我的[github](https://github.com/aaaa9990aaaa/uos-ui "gayhub")进行完善，做一个热爱造轮子的切图仔（enjoy)。

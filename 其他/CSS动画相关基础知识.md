# CSS动画相关基础知识

## transition

transition是为了让html元素的属性变化有一个过渡效果，transition只关注状态前后的切换动画，所以，transition只关注两个点，起始和结束，transition定义的就是元素属性变化在起始和结束之间的过渡效果。

```css
xxx {
  transition-property: height; # 定义应用此过渡效果的元素属性
  transition-duration: 1s; # 定义过渡效果的持续时间，也就是从初识到结束状态的变化时间
  transition-delay: 1s; # 定义过渡效果启动延迟
  transition-timing-function: ease; # 定义过渡动画效果
}

xxx {
  # 简写方式
  transition: <property> <duration> <timing-function> <delay>; 
}
```

其中 transition-timing-function有四种取值

* linear：匀速
* ease-in：加速
* ease-out：减速
* cubic-bezier：自定义速度模式

注意：transition只是定义了元素属性变化时候的过渡效果，元素属性什么时候变化，这个不归trasition管，需要有触发方式，来**触发transition指定元素的属性值**，因此，transition可以让js操作dom时，让dom元素属性变化平滑。

## transform

transform就是元素的一个属性和height,width没有本质区别，用来定义元素的一些变化，放大、旋转、位移等等。

```css
xxx {
  transform: rotate(xxdeg); # 旋转 xxx°
  transform: skew(xxxdeg); # 倾斜 xxx°
  transform: scale(1.5); # 缩放1.5倍
  transform: translate(120px,10px); # 向右位移120px，向下位移10px
}
```

注意：transform和动画一点关系也没有，它只是一个属性，和其他属性一样，这个属性是可以变化的，只不过和其他属性的变化效果有些不同。

## animation

ainimation是元素的属性，这个属性需要和 @keyframes 一起用，animation其中一个设定值就是@keyframes的名字，@keyframes可以**定义多个帧的样式**，在0%~100%进度之间可以加入任意多个动画帧，每个帧定义了元素样式，然后元素就从0%开始一点点过渡到100%的样式，@keyframes定义的样式之间的过渡效果也在animation当中定义

```css
xxx {
  animation: gif 1.4s infinite linear;
}

@keyframes gif {
  0% {
   	transform: rotate(20deg);
  }
  20% {
    top: 100px;
  }
  50% {
    top: 200px;
  }
  100% {
    transform: rotate(100deg);
  }
}
```

其中animation的设定值有多个：

```css
animation: 
```


## CSS3 Animations
```css
/*
    keyframes 
    具体描述一个动画
*/

@keyframes example {
    from {background-color: red;}
    to {background-color: yellow;}
}
@keyframes example {
    0%   {background-color: red;}
    25%  {background-color: yellow;}
    50%  {background-color: blue;}
    100% {background-color: green;}
}


/*
    animation 
    所有动画属性的简写
*/
div {
    animation: example 5s linear 2s infinite alternate;
}
-->
div {
    animation-name: example;
    animation-duration: 5s;
    animation-timing-function: linear;
    animation-delay: 2s;
    animation-iteration-count: infinite;
    animation-direction: alternate;
}
/*
    animation-name
    动画的名字
*/
div{
    animation-name: example;
}

/*
    animation-duration
    动画时长
*/

/*
    animation-timing-function
    动画曲线控制器
    
*/
div{
    animation-timing-function: linear|ease|ease-in|ease-out|ease-in-out|step-start|step-end|steps(int,start|end)|cubic-bezier(n,n,n,n)|initial|inherit;
}

::steps::
div{
    animation-timing-function:steps(2, start);
    /* 
        steps 函数指定了一个阶跃函数，第一个参数指定了时间函数中的间隔数量（必须是正整数）；第二个参数可选，接受 start 和 end 两个值，指定在每个间隔的起点或是终点发生阶跃变化，默认为 end 
        通常是和
        animation-iteration-count: 2;
        animation-duration: 3s;
        这两个属性一起
    */
}
::cubic-bezier::

div{
    animation-timing-function:cubic-bezier(x1, y1, x2, y2);
    /* x1 and x2 must be in the range [0, 1] or the value is invalid.*/
}

/* 
    animation-fill-mode: none | forwards | backwards | both; 检索或设置对象动画时间之外的状态。
    
   有四个值可选，并且允许由逗号分隔多个值。
    
    none 不改变默认行为。
    forwards 当动画完成后，保持最后一个属性值（在最后一个关键帧中定义）。
    backwards 在 animation-delay 所指定的一段时间内，在动画显示之前，应用开始属性值（在第一个关键帧中定义）。
    both 向前和向后填充模式都被应用。
*/
/*
animation-direction
    normal：正常方向。
    reverse：动画反向运行,方向始终与normal相反。（FF14.0.1以下不支持）
    alternate：动画会循环正反方向交替运动，奇数次（1、3、5……）会正常运动，偶数次（2、4、6……）会反向运动，即所有相关联的值都会反向。
    alternate-reverse：动画从反向开始，再正反方向交替运动，运动方向始终与alternate定义的相反。（FF14.0.1以下不支持）
*/

/*  之行
    animation-iteration-count: number|infinite|initial|inherit;
*/


```
[CSS3 timing-function: steps() 详解](https://idiotwu.me/understanding-css3-timing-function-steps/)

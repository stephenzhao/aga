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

```
[CSS3 timing-function: steps() 详解](https://idiotwu.me/understanding-css3-timing-function-steps/)

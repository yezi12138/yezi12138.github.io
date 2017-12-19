---
title: 搭建vue音乐播放器（1）
date: 2017-05-17 15:16:23
tags: -vue
---

# 搭建vue音乐播放器（1）

------

## 首页轮播组件解析

### 实现功能

> * 自动轮播
> * 滑动切换
> * 点击indicator切换


### html：

    <template>
    	<div class="swiper">
    		<div class="swiper-items-wrap" ref="wrap">
    			<slot></slot>
    		</div>
    		<div class="swiper-indicators-wrap">
    			<div class="swiper-indicator" :class="{'is-active':$index==index}" v-for="(page,$index) in pages" v-show="showIndicators" @click="nevigateTo($event,$index)"></div>
    		</div>
    	</div>
    </template>

### 实现原理

 假设有4个页面，分别标号为1，2，3，4
 
 **css样式初始化：** 1页面设置为block，其他为none
 
 **页面初始化：** 获取`<div class="swiper-items-wrap" ref="wrap"></div>`这个节点,然后获取它下面子节点的个数，根据这个子节点的长度初始化indicator

**鼠标按下：** 删除保存的移动方向<font color="red">（不然单击页面会自动移动）</font>，初始化页面位置（调用initPos方法）
根据当前的index，将index--，index++设置block，分别设置为前一个页面和后一个页面。这样就存在三个设置为block的页面

**鼠标移动：** 保存鼠标移动的方向（toward），保存移动的距离（offset），调用translate方法移动页面

**鼠标离开：** 
1. 判断移动的距离，如果小于页面的一半，则取消移动，重新初始化页面，此时页面依然是1，2，3显示为block
2. 如果距离大于页面一般，则先将当前的3个页面设置为none隐藏，令index++<font color="red">（index--，这个根据方向判断）</font>,再重置页面位置，这时候显示的是2，3，4（block），其他未none，因为2，3页面在移动时候设置了动画，所以即使由block->none->block，动画却不会断开。相当于从隐藏的位置到初始化位置的动画过渡。

------

## bug修复

1. 单个页面时候取消移动效果
2. 两个页面的时候取消动画效果
3. 三个页面时候把位置由前->后 或者 后->前变化的页面动画取消
<font color="red">也就是说这种算法至少需要4个页面</font>



放上轮播组件的github地址： https://github.com/yezi12138/swiper
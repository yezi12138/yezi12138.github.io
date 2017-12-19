---
title: 搭建vue音乐播放器（2）
date: 2017-06-02 17:34:44
tags:
     -vue
---

# 搭建vue音乐播放器（2）

------

最近在写这个SPA，在做完的时候才有时间认真回顾一下过程，同时写下每个模板结构和我写代码时候发现的一些bug
附上项目地址（https://github.com/yezi12138/vuemusic）

## 第一个说说搜索页面有一个小小bug的修复，代码如下

    
    //search.vue下的search方法
    search(value){  //点击查找
            /*在搜索空字符的时候，会把空字符添加到搜索历史这里，所以 这里我用一个判断return掉空字符*/
				value = value?value:this.searchValue;
				if(value === '') return;
				

## 第二个说说我的song.vue模块的设置

第一次我是song.vue通过路由方式导入，可这样就出现一个问题，首先是动画太僵硬了，第二个是因为有多处可以直接切换到song模块（比如底部模块和主页右上角图标和搜索模块），对于每一个切换的模块来说，需要传入歌曲参数，增加了每个模块之间的耦合度，对于song模块来说，每次都要重新渲染数据，加载数据，网络不好的时候用户体验不好。
第二次思考将song隐藏起来，通过store.js来给数据更新，这样大大减低的各个模块的耦合度，而这次我将song里面的播放列表童颜隐藏起来。首先我用的是better-scroll，这个组件需要在dom都渲染完毕才可以拉动。因为我将播放列表先隐藏后展示，这样BScroll无法在显示的时候获取到dom，只能在下一次刷新dom时候才可以拉动播放列表。
对于上两次的bug，最终我将song隐藏起来，将songlist移出手机界面达到伪隐藏效果。song的动画和songlist刷新都可以了。

    <div id="app">
		<router-view></router-view>
		<transition name="slide">
			<song v-show="songShow" ref="song"></song>
		</transition>
		<songlist ref="songlist"></songlist>
		<transition name="fade">
			<div class="bg-wrap" @click="toggleWrapShow" v-show="wrapShow"></div>
		</transition>
	</div>


## 对于歌曲的滚动

首先是需要给li设置高度，否则很难移动对应的距离（当然可以统计前几个移动li的高度再给ul设置，不过这样会增加代码，我反而觉得限定高度比较好）

    refleshLyric(){
				var audio = this.$refs.audio;
				var that = this;
				audio.addEventListener('timeupdate', function(){
					var ul = document.getElementById('lrc'),
					lis = ul.querySelectorAll('li'),
					lyric = that.lyric;
					for(let i =0;i<lyric.length;i++){
						var j = i+1;
			//这个是用来判断正要高亮的歌词
			if(lyric[i].time<audio.currentTime&&lyric[j].time>audio.currentTime&&i>4){
							var height = Math.floor(lis[i].offsetHeight);
							//（i-4）是为了让高亮歌曲在中间
							ul.style.top = `-${(i-4)*height}px`;
							lis[i].className = 'active';
							return;	
						}else{
						    //将高亮歌词前面的li样式重置
							lis[i].className = '';
						}
					}
				});
			}
			
如果你拖动进度条改变歌曲的进度，则需要下面代码清楚所有的li样式，否则会出现多处高亮的情况

    clearLyricCss(){
				var ul = document.getElementById('lrc'),
				lis = ul.querySelectorAll('li');
				lis.forEach((item) => {
					item.className = ''
				})
			}
	
	//监听进度条移动时候清楚样式	
	indicator.addEventListener('touchmove', (event) => {
					this.doOnTouchMove(event);
					this.clearLyricCss();
				});
				
## 最后讲一下一个小bug
每次切换新歌曲的时候，如果你之前正在播放音乐，那么播放按钮会显示正在播放的图标，解决方法是每次切换歌曲判断新的歌曲是不是和旧的歌曲一样

    //判断进入的是否是正在播放的歌曲，如果不是则重置播放按钮样式
				var currentSong = this.$store.state.currentSong.data;
				if(currentSong.songid != songid){
					this.$store.commit('initPlaying')
				}
				
				
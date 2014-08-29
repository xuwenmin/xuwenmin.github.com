---
layout: post
category : JavaScript
tagline: ""
tags : [算法]
---
{% include JB/setup %}

今天发现了一个挺好玩的算法题,下面是它的算法描述,源自`twitter`的一道面试题

### twitter puddles 算法描述

先看一副图

<img src="https://github-camo.global.ssl.fastly.net/72bb7f52be6ff68e0ea91012f9add44fc2a663e9/687474703a2f2f7777312e73696e61696d672e636e2f6d773639302f376363383239643367773165613536736e6e6e6b7a6a3230356d30337a3734342e6a7067" alt="">

上图里的数字是根据一个数组内容来描述的,最后会根据每个数字的大小来模拟一道墙的高度,最后生成一面墙,问你,当下雨的时候,这面墙可以装多少水,以1为计数单位

下面是装完水之后的一面墙的样子

<img src="https://github-camo.global.ssl.fastly.net/631535d701052f925a561116fa255cca67aa20e9/687474703a2f2f7777332e73696e61696d672e636e2f6d773639302f376363383239643367773165613536706a6e746b6f6a3230356d30337a6161322e6a7067" alt="">

看完上面上幅图,感觉是不是很好玩,确实,下面来简单的分析下它的算法实现

其实这个原理比较简单,总共有下面几个要点:

* 最左边和最右边肯定不能装水
* 装水的高度依赖自身左右两侧内两个最大值其中的最小值

下面我们用`js`来简单的实现它

```js
/**
*  计算以数组项为高度的墙能装多少水
*  数组例子 [2,5,1,2,3,4,7,7,6,9]
**/
function getWaterCounts(arg){
	var i = 0,
		j = 0,
		count = 0;
	// 第一项和最后一项都得排除
	for(i = 1; i < arg.length - 1; i++){
		var left = Math.max.apply(null, arg.slice(0, i + 1));
		var right = Math.max.apply(null, arg.slice(i, arg.length));
		var min = left >= right ? right : left;
		// 以左右两边最大值内小的为准
		// 假如当前值大于或者等于这个值什么都不做
		if(arg[i] < min){
			count += min - arg[i];
		}
	}
	console.log(count);
}
getWaterCounts([2,5,1,2,3,4,7,7,6,9]); // 11

```

### 总结

嘿嘿,实现是不是挺简单的,其实只要你愿意思考,用`js`可以实现很多好玩的东西.
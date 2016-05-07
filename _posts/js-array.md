---
layout: page
title:
category: blog
description:
---
# Preface

	new Array(1,2,3);
	arr = [1,2,'d'];
	brr[0]=11;
	arr;//[11,2,'d']; //因为数组赋值使用的是对象引用

	.length
	.concat(arr2)
	.join([separator]); //implode
	.pop() ;//返回item
	.push(item1, itme2, ...);//return length
	.reverse()
	.shift();//左移(移出) .unshift(item1,item2);//右移(移入)

	.slice(start, howmany); //支持负数
	.splice(start, howmany[, newValue]);//删除
        a=[1,2,3,4,5]
            [1, 2, 3, 4, 5]
        a.splice(2,2,'abc')
            [3, 4]
        a;
            [1, 2, "abc", 5]
	.sort([funcSort]);

	转换:

		arr.toString();arr.valueOf();
		arr.join(',');//

	栈方法:

		arr.push('new item');
		arr.pop();

	队列:

		arr.push('new');
		arr.shift();//LIFO 后进先出

	重排方法(改变arr本身):

		arr.reverse();//倒序
		arr.sort();//从小到大
		arr.sort(function(a,b){return a-b;});//自定义排序

	操作方法:

		[0,1].concat(2,[3,4]);//返回数组[0,1,2,3,4];
		arr.slice(start,[end]);//范围不含end本身,end可为负数,
		arr.splice(start,[length]);//返回删除范围
		arr.splice(start,length,val1,val2,....);//返回删除范围,并插入数据

	位置方法:

		['aa','bb','cc'].indexOf('aa');//0 找不到就返回-1
		['aa','bb','cc','aa'].lastIndexOf('aa');//3 找不到就返回-1

	迭代方法:

		.every(func);//每一项运行给定函数都返回true,结果才返回true. //func= function(item,index,array){}; array是对数组本身的引用
		.some(func);//只要其中一项运行指定函数时返回true,结果就返回true.
		.filter(func);//返回运行为true item组成的数组
		.map(func);//返回函数运行结果组成的数组
		.forEach(func);//只运行不返回

	归并方法:

		.reduce(func);//从第一项开始
		.reduceRight(func);//从最后一项开始
		arr.reduce(function(pre,cur,index,array_self){return pre+cur;})
		[1,2,3].reduce(function(pre,cur,index,array_self){return pre+cur;})
			array_self 作为数组是按引用传值的(数组元素length>=2)
			prev = prev+ curr = 1+2 = 3;
			prev = prev+ curr = 3+3 = 6;

	Array.prototype.inArray = function(needle) {
        var length = this.length;
        for(var i = 0; i < length; i++) {
            if(this[i] == needle) return true;
        }
        return false;
    }


---
layout: page
title:
category: blog
description:
---
# Preface

library:
http://www.chartjs.org/docs/#line-chart-example-usage

echarts:
http://echarts.baidu.com/doc/example/line1.html

	option = {
		title : {
			text: 'v6-v4 QPS',
			subtext: ''
		},
		tooltip : {
			trigger: 'axis'
		},
		legend: {
			data:['v6','v4']
		},
		toolbox: {
			show : true,
			feature : {
				mark : {show: true},
				dataView : {show: true, readOnly: false},
				magicType : {show: true, type: ['line', 'bar']},
				restore : {show: true},
				saveAsImage : {show: true}
			}
		},
		calculable : true,
		xAxis : [
			{
				type : 'category',
				boundaryGap : false,
				data : ['200','400','600','800','1000','1200','1400', '1600', '1800', '2000']
			}
		],
		yAxis : [
			{
				type : 'value',
				axisLabel : {
					formatter: '{value} req/sec'
				}
			}
		],
		series : [
			{
				name:'v6',
				type:'line',
				data:[36.73, 82.25, 95.90, 91.47, 157.92, 250.14, 229.49, 196.83, 303.23, 180.28],
				markPoint : {
					data : [
						{type : 'max', name: '最大值'},
						{type : 'min', name: '最小值'}
					]
				},
				markLine : {
					data : [
						{type : 'average', name: '平均值'}
					]
				}
			},
			{
				name:'v4',
				type:'line',
				data:[
				189.99, 231.95, 241.36, 258.06, 239.92, 245.35, 245.93, 200.72, 257.30, 262.59
				],
				markPoint : {
					data : [
						{name : '最低', value : -2, xAxis: 1, yAxis: -1.5}
					]
				},
				markLine : {
					data : [
						{type : 'average', name : '平均值'}
					]
				}
			}
		]
	};



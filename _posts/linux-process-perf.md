---
layout: page
title:
category: blog
description:
---
# Preface

- sar, iostat, top

# sar
了解CPU、内存和硬盘的使用情况

## cpu
表示每秒采样一次，总共采样2次

	$ sar -u 1 2
	09:03:59 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
	09:04:00 AM     all      0.00      0.00      0.50      0.00      0.00     99.50
	09:04:01 AM     all      0.00      0.00      0.00      0.00      0.00    100.00

## memory

	sar -r 1 2

不采样的话, 也可以用`free`

## other

	-d       Report disk activity.
	-g       Report page-out activity.
	-p       Report page-in and page fault activity

# perf book
http://linuxtools-rst.readthedocs.org/zh_CN/latest/advance/index.html


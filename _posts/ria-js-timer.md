---
layout: page
title:	
category: blog
description: 
---
# Preface

You need to give JavaScript a reference to the interval:

	var t = setTimeout(function() {
		timer(counter + 1);
	}, 3000);

	var t = setTimeout(function() {
		timer(counter + 1);
	}, undefined);//立即执行:0ms

Then you can clear it like so:

	$("#stop").click(function () {
	   clearTimeout(t);
	});

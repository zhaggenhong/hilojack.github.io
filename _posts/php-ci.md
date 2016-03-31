---
layout: page
title:
category: blog
description:
---
# Preface

CI Framework 改造计划


vim src/config/config.php

	/*
	|--------------------------------------------------------------------------
	| Autoload Custom Controllers
	|--------------------------------------------------------------------------
	*/
	function __autoload($class) {
		if (substr($class,0,3) !== 'CI_') {
			if (file_exists($file = APPPATH . 'core/' . $class . EXT)) {
				include $file;
			}
		}
	}

---
layout: page
title:
category: blog
description:
---
# Preface

# php jquery
Operation html string like jquery

https://code.google.com/p/phpquery/

# tidy

	$str = tidy_parse_string($str, ['indent'=>true, 'wrap'=>0], 'UTF8');
	$str->cleanRepair();
	$str = (string) $str;

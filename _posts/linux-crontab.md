---
layout: page
title:
category: blog
description:
---
# Preface

# PATH

	$ man 5 crontab | grep -C5 PATH | tail
	SHELL=/bin/sh
	PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

	# m h dom mon dow usercommand
	17 * * * *  root  cd / && run-parts --report /etc/cron.hourlyPATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

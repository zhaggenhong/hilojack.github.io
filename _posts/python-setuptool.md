---
layout: page
title:
category: blog
description:
---
# Preface
setuptools 与 disutils
1. distutils 用于包的创建与分发到Python Package Index（PyPI）,
2. setuptools 是distuils 增强，并解决包间依赖关系。

# Basic Use
我想到打包我写的一个python脚本: fileset

	$ ls your_proj/
	fileset
	$ cd your_proj/

	$ vim setup.py

	from setuptools import setup, find_packages
	setup(
		name = "fileset",
		version = "0.1",
		packages = find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"]),
		scripts = ['fileset'],

		install_requires = ['python>=3.0'],


		# metadata for upload to PyPI
		author = "hilojack",
		author_email = "a132811@gmail.com",
		description = "This is a filset Package",
		license = "GPL",
		keywords = "fileset",
		url = "http://github.com/hilojack/",   # project home page, if any
		# could also include long_description, download_url, classifiers, etc.
	)

# setup

	$ python setup.py sdist
	$ python setup.py sdist

## find_packages
For simple projects, it’s usually easy enough to manually add packages to the packages argument of setup(). However, for very large projects (Twisted, PEAK, Zope, Chandler, etc.), it can be a big burden to keep the package list updated. That’s what setuptools.find_packages() is for.

	find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"])


# upload

	p3 setup.py upload

register and upload:

	p3 setup.py register bdist_egg upload
	p3 setup.py register sdist bdist_egg upload

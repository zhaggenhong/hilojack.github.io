---
layout: page
title:
category: blog
description:
---
# Preface

# argparse
利用 argparse 可以很方便的解析参数、生成Usage:

    #!/usr/bin/env python
    import argparse as apa
    def main():
        parser = apa.ArgumentParser(prog="convert") 
        parser.add_argument("-c", "--config", required=False, default="config.ini")
        parser.add_argument("-t", "--theme", required=False, default="default.theme")
        parser.add_argument("-f", "--f", default="-") # Accept Jupyter runtime option
        args = parser.parse_args()
        print(args);

    if __name__ == "__main__":
        main()

结果：

    > test p3 a.py
    Namespace(config='config.ini', f='-', theme='default.theme')
    > test p3 a.py  -h
    usage: convert [-h] [-c CONFIG] [-t THEME] [-f F]

    optional arguments:
      -h, --help            show this help message and exit
      -c CONFIG, --config CONFIG
      -t THEME, --theme THEME
      -f F, --f F

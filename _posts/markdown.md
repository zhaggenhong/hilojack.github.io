---
layout: page
title:	markdown & ma
category: blog
description:
---
# Preface

# LaTeX
Mathematics Stack Exchange uses MathJax to render LaTeX. You can use single dollar signs to delimit inline equations, and double dollars for blocks:

	The *Gamma function* satisfying $\Gamma(n) = (n-1)!\quad\forall
	n\in\mathbb N$ is via through the Euler integral

	$$
	\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
	$$

## jekyll
http://gastonsanchez.com/blog/opinion/2014/02/16/Mathjax-with-jekyll.html

# Escape
Refer to:
http://meta.stackexchange.com/questions/82718/how-do-i-escape-a-backtick-in-markdown

    ``List`1``

when inline it will display List`1

Markdown provides backslash escapes for the following characters:

    \   backslash
    `   backtick
    *   asterisk
    _   underscore
    {}  curly braces
    []  square brackets
    ()  parentheses
#   hash mark
    +   plus sign
    -   minus sign (hyphen)
    .   dot
    !   exclamation mark

for example, this:

    ## \\ \` \* \_ \{ \} \[ \] \( \) \# \+ \- \. \!

returns:

    \ ` * _ { } [ ] ( ) # + - . !

Escaped codeblock, italics, bold, link, headings and list:

`not codeblock`, *not italics*, **not bold**, [not link](http://www.google.com)

    # not h1

    ## not h2

    ### not h3

    + not ul

    - not ul



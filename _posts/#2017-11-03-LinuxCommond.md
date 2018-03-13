---
layout: post
category: Linux commond
title: cat
tagline: by wencolani
tags: [linux commond, cat]
---

## main commond
`cat file1`: output the content in file1

`cat file1 > file2`: rewrite file1 to file2

`cat file1 | less`: output the content in file1 in less way

`cat > file1`: write to file1, if file1 exists, the content will be covered. Use `Ctrl + d` to end input

`cat >> file1`: append new written content to file1

`cat file1 file2 file3`: output file1, file2 and file3

`cat file1 file2 file3 > file4`: write file1, file2 and file to file4

`cat file1 file2 file3 | sort > file4`: sort all the content in file1, file2 and file3, then write them to file4

`cat - file1 > file2`: first copy file1 to file2, then add input something to file2 

## option
`-n` or `--number`: 从0开始对每一行编号

`-b` or `--number-nonblank`: 对空白行不编号

`-s` or `--squeeze-blank`: 当有连续两行空白行时换成一行

...
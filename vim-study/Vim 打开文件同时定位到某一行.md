Vim 打开文件同时定位到某一行
转载豆-Metcalf 最后发布于2016-12-26 21:52:43 阅读数 8126  收藏
展开
使用vim 打开文件时，默认情况下光标会停留在文件开头，有时候文件比较大，翻阅和查找都比较麻烦，怎样在打开的时候直接定位到某一行呢？

使用 "+" 

vim filename +n 可以在打开文件的同时定位到第n行。
1
2
3
eg: vim myfile +100 //打开的同时光标停在第100行行首

也可以在打开文件的同时找到第一个匹配的词。

vim filename  +/pattern 
1
2
3
http://www.xuebuyuan.com/1111974.html
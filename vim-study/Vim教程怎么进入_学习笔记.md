# Vim教程怎么进入_学习笔记

https://vimjc.com/how-to-open-vimtutor.html

##  第二讲第七节：撤消类命令

ps: 撤销(u)的反撤销是Ctrl-R, 相当于windows中的Ctrl-Y, 这个功能非常好, 有时候使用u撤销了, 又后悔了, 不知道怎么办, 这下好了, 可以使用Ctrl-R来恢复撤销.
多次输入 CTRL-R (先按下 CTRL 键不放开，接着按 R 键)，这样就可以重做(Redo)被撤消的命令，也就是撤消掉撤消命令。

## 第四讲小结

CTRL-O 带您跳转回较旧的位置，CTRL-I 则带您到较新的位置。

ps: CTRL-O 类似于VSCOD中的 Alt-向左
CTRL-I 类似于Alt-向右

进行全文替换时询问用户确认每个替换需添加 c 标志 :%s/old/new/gc

## 第五讲第四节：提取和合并文件

提示：您还可以读取外部命令的输出。例如， :r !ls 可以读取 ls 命令的输出，并
      把它放置在光标下面。

有了上面这个功能, 在vim中想要查看 ps -ef|grep uxdb的结果, 并且需要做一些搜索处理, 就可以这样做:
1) 在当前vim中, 输入 :e 1   新建一个名称为1的临时文件
2) 在1文件中, :r !ps -ef|grep uxdb
3) 那么 ps 的结果就会被输出到 1 文件中
4) 然后就可以在 1 文件中慢慢进行查看和查找了
5) 查看完了, 输入 :ls   查看之前的文件序号, 如: 5
6) 输入 :b5, 就可以切换回去了
7) 至于那个临时文件, 目前不知道如何关闭, 因为在 1 中输入 :q!  会导致整个vim退出, 可以放着不用管

## 第六讲第三节：另外一个置换类命令的版本

** 输入大写的 R 可连续替换多个字符。**

在命令模式下, 修改一个字符, 并且还不想回到编辑模式, 我知道是输入小写的r, 即可替换.
而如果要修改一个单词, 或者多个字符, 之前使用的是 s 或 cw, 但修改完了会进入到编辑模式, 感觉不好.
这下好了, 可以使用 R 来实现连续替换多个字符, 并且修改完了仍然在命令模式.

## 第六讲第五节：设置类命令的选项


		  ** 设置可使查找或者替换可忽略大小写的选项 **

  1. 要查找单词 ignore 可在正常模式下输入 /ignore <回车>。
     要重复查找该词，可以重复按 n 键。

  2. 然后设置 ic 选项(Ignore Case，忽略大小写)，请输入： :set ic

  3. 现在可以通过键入 n 键再次查找单词 ignore。注意到 Ignore 和 IGNORE 现在
     也被找到了。

  4. 然后设置 hlsearch 和 incsearch 这两个选项，请输入： :set hls is

  5. 现在可以再次输入查找命令，看看会有什么效果： /ignore <回车>

  6. 要禁用忽略大小写，请输入： :set noic

提示：要移除匹配项的高亮显示，请输入：  :nohlsearch
提示：如果您想要仅在一次查找时忽略字母大小写，您可以使用 \c：
      /ignore\c <回车>

忽略大小写的命令可以缩写为 ic, 真的很方便

## 第七讲第三节：补全功能


	      ** 使用 CTRL-D 和 <TAB> 可以进行命令行补全 **

  1. 请确保 Vim 不是在以兼容模式运行： :set nocp

  2. 查看一下当前目录下已经存在哪些文件，输入： :!ls   或者  :!dir

  3. 现在输入一个目录的起始部分，例如输入： :e

  4. 接着按 CTRL-D 键，Vim 会显示以 e 开始的命令的列表。

  5. 然后按 <TAB> 键，Vim 会补全命令为 :edit 。

  6. 现在添加一个空格，以及一个已有文件的文件名的起始部分，例如： :edit FIL

  7. 接着按 <TAB> 键，Vim 会补全文件名(如果它是惟一匹配的)。

提示：补全对于许多命令都有效。您只需尝试按 CTRL-D 和 <TAB>。
      它对于 :help 命令非常有用。


ps:

之前对:e 的认识就如下
1) :e xxx        新建一个xxx的文件
2) :e /home/uxdb/1.sql    打开一个已有的文件
3) :e!           刷新当前文件

今天又学习到了一个很实用的功能:
输入 :e 后, 在按下 CTRL-D, 可以在vim下查看当前目录文件列表
## <center> Linux下常用命令 </center>

#### find
通过`find . -name xxx`查找当前目录下文件名为xxx的文件。
通过`find . -iname xxx`进行正则匹配。

#### history
查看历史命令，2000条

#### awk
awk作为一个强大的文本处理工具，可以把文本处理成想要的格式。
`ls -lh | awk 'print{ $1}'`输出ls -lh的第一列。

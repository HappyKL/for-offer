# 帮助命令

```shell
# man
man cd
# 有几个章节，每个章节负责不同的命令，默认是1，1表示shell命令，可省略，有时候可能是其他，比如文档什么的
man 1 cd
# 如果无法确定所查是什么类型，可以直接采用-a，将所有均进行查看
man -a cd

# help
# 对于内部命令使用以下命令，以cd为例
help cd
# 对于外部命令使用以下命令，以ls为例
ls --help
# 采用以下命令查看是否为内部命令,以cd为例
type cd

# info比help更详细
info ls

```

## less命令

```shell
less abc.log
G # 移动到最后一行
g # 移动到第一行
j # 向前移动一行
k # 向后移动一行

ctrl+f #向前移动一屏
ctrl+b #向后移动一屏
ctrl+d #向前移动半屏
ctrl+u #向后移动半屏

```


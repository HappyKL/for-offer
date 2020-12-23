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



### nc传文件

```shell
端口8000~8100可用
# 方法1 先启动接收，后启动发送
nc -l 8002 > offline-mr-1.0-SNAPSHOT-jar-with-dependencies.jar
nc 100.69.199.12 8002 < offline-mr-1.0-SNAPSHOT-jar-with-dependencies.jar

# 方法2 先启动发送，后启动接收
nc -l 9992 <test.mv
nc 10.0.1.162 9992 >test2.mv

# 传目录 先接收，后发送 (原理就是把目录打包成-，然后接收后解压)
nc -l 9995 | tar xfvz -
tar cfz - * | nc 10.0.1.162 9995

```



查看文件有多少行



```shell
wc -help

wc -l filename 就是查看文件里有多少行

wc -w filename 看文件里有多少个word。

wc -c filename

```



scp传文件

```
scp my_local_file.zip root@192.168.1.104:/usr/local/nginx/html/webs
```



查看日志最后

```shell
less sigma.log
shift+g  //移动到最后一行
ctrl+b   //往前一页一页翻页查看
```


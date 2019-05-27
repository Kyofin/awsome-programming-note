# webmagic快速体验





### pipline输出mysql5.7时遇到问题



#### 1.中文写入乱码问题：

我输入的中文编码是urf8的，建的库是urf8的，但是插入mysql总是乱码，一堆"???????????????????????"

我用的是ibatis，终于找到原因了，我是这么解决的：

原url地址是：jdbc:mysql://localhost:3306/comment1

改为：jdbc:mysql://localhost:3306/comment1??useUnicode=true&amp;characterEncoding=UTF-8

就OK了。



#### 2.Incorrect string value: '\xF0\x9F...' for column 'XXX' at row 1

这个问题，原因是UTF-8编码有可能是两个、三个、四个字节。Emoji表情或者某些特殊字符是4个字节，而Mysql的utf8编码最多3个字节，所以数据插不进去。

我的解决方案是这样的

1.在mysql的安装目录下找到my.ini,作如下修改：

[mysqld]

character-set-server=utf8mb4

[mysql]

default-character-set=utf8mb4

修改后重启Mysql

2. 将已经建好的表也转换成utf8mb4

命令：alter table TABLE_NAME convert to character set utf8mb4 collate utf8mb4_bin; （将TABLE_NAME替换成你的表名）



#### 3.MysqlDataTruncation: Data truncation: Data too long for column 'repo_readme'

这是因为原来表设计readme字段类型为text太小了。查阅了下资料，发现text最多支持64kb。

**改成mediumtext就可以。**

MySQL 3种text类型的最大长度如下：

- TEXT 65,535 bytes ~64kb
- MEDIUMTEXT 16,777,215 bytes ~16Mb
- LONGTEXT 4,294,967,295 bytes ~4Gb
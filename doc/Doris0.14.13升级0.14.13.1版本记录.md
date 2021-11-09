# Doris0.14.13升级0.14.13.1版本记录

由于只是小版本升级，所以可以采取直接部署新版，fe配置指向之前的元数据，be配置指向之前存储的数据。

### 版本变更内容

参考：http://palo.baidu.com/docs/%E7%89%88%E6%9C%AC%E5%8F%91%E5%B8%83%E5%8E%86%E5%8F%B2/

该版本为 0.14.13 基础上的 Bug 修复版本。

1. 重要 Bug 修复
   - 修复部分情况下，执行 `show proc /bdbje` 后，会导致历史元数据日志无法删除的问题。
   - 修复部分情况下，union 常量查询结果错误的问题。
   - 修复部分情况下，开启 colocation plan 变量导致查询结果不一致的问题。
   - 修复部分情况下，colocation 表的数据分片处于 DECOMMISSION 状态无法修复的问题。
   - 修复 no-avx2 版本的二进制程序错误的调用 avx2 指令的问题。
2. 优化
   - 执行 Insert 命令导入 0 行数据的情况下，依然后记录 Insert 操作。可以通过 `show load` 命令查看。
   - 升级 librdkafka 到 1.6 版本，以支持 routine load 消费 kafka 时使用 read_committed 语义。
   - 增加若干 FE 元数据 checkpoint 相关监控指标，以方便监控元数据 checkpoint 问题。

### 

## 具体步骤

### 先换be

1、选择旧版的be节点停止

2、停止后修改新版be节点的配置，和旧版的保持一致。主要修改参考如下：

![image-20210929164049095](http://image-picgo.test.upcdn.net/img/20210929164049.png)

这里注意新版的be配置里的存储目录要之前之前的存储数据目录。

3、新版的be节点启动

```
bin/start_be.sh --daemon
```

4、观察新版的be节点日志，看看是否正常，这里最好用工具查询一下表，看看有新的执行计划打印到控制台。

5、按1-4步骤重复在各个服务器上，停止旧版be，启动新版be节点





### 先换fe

1、先停止旧版的fe节点

2、修改新版fe配置，参考如下：

![image-20210929164503418](http://image-picgo.test.upcdn.net/img/20210929164503.png)

这里主要要将新fe指向之前fe存储元数据的目录。

3、启动新版fe节点

```
bin/start_fe.sh --daemon
```

4、检查日志观察fe是否成功启动，通过mysql客户端查询数据观察是否正常即可。
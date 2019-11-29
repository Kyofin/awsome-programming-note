对 escheduler master为1.1.0开发需进行以下操作

# 系统配置

需要配置免密登录

# IDEA配置

## 使用maven编译proto文件
`escheduler-rpc/src/main/proto/scheduler.proto` 这里用到

使用idea的maven插件执行一下就可以了。
![](https://i.loli.net/2019/11/29/zCPGthaDwp6EyQd.png)

# 数据库配置

- 手动建表导入sql
    这里以本地docker mysql为例
    
    ```
    sudo docker cp /opt/programming/github/incubator-dolphinscheduler/sql mysql3306:/opt/incubator-dolphinscheduler
    
    sudo docker exec -it mysql3306 /bin/bash
    ```
    
    ```
    root@65eed6cc1e35:~# mysql -uroot -p
    
    
    msyql> CREATE DATABASE escheduler DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
    
    msyql> use escheduler
    
    mysql> source /opt/incubator-dolphinscheduler/create/release-1.0.0_schema/mysql/dolphinscheduler_ddl.sql
    
    mysql> source /opt/incubator-dolphinscheduler/create/release-1.0.0_schema/mysql/dolphinscheduler_dml.sql
    
    mysql> source /opt/incubator-dolphinscheduler/upgrade/1.0.1_schema/mysql/dolphinscheduler_ddl.sql
    
    mysql> source /opt/incubator-dolphinscheduler/upgrade/1.0.2_schema/mysql/dolphinscheduler_ddl.sql
    
    mysql> source /opt/incubator-dolphinscheduler/upgrade/1.0.2_schema/mysql/dolphinscheduler_dml.sql
    
    mysql> source /opt/incubator-dolphinscheduler/upgrade/1.1.0_schema/mysql/dolphinscheduler_ddl.sql
    ```
- 使用java类自动建表更新sql（推荐）
   -  首先配置数据库信息`EasyScheduler/escheduler-dao/src/main/resources/dao/data_source.properties`
   ![](https://i.loli.net/2019/11/29/U7xyZo8tnuckjwK.png)
   -  使用cn.escheduler.dao.upgrade.shell.CreateEscheduler的`main方法`即可。

## 项目中需要修改的数据库配置文件

需要修改的配置文件

* `escheduler-dao` 下的 data_source.properties 文件
* `escheduler-common` 下的 quartz.properties 文件

# 需要进行修改的配置文件

## 环境变量配置文件

escheduler-common 下的 `common/common.properties`文件

主要修改 `escheduler.env.path`改成本地项目中的，如

```
escheduler.env.path=/opt/programming/github/incubator-dolphinscheduler/script/env/.escheduler_env.sh
```

### zookeeper配置

escheduler-common 下的zookeeper.properties ，配置zookeeper.quorum属性


### 监控服务的邮箱配置
修改文件`alert.properties`
![](https://i.loli.net/2019/11/29/ByxkOJ1Sbj95RXq.png)

# idea启动项目
使用`cn.escheduler.api.CombinedApplicationServer`启动

会一次把所有服务都启动。


# 前端配置

前端页面在 `escheduler-ui` 目录

先装依赖

```
npm install
```

修改配置文件 `.env` （取消下面一行的注释即可）

```
# 后端接口地址
API_BASE = http://localhost:12345

# 本地开发如需ip访问项目把"#"号去掉
DEV_HOST =localhost
```

启动项目

```
npm run start
```

# 登录信息

## 页面访问

http://localhost:8888/

## 登录用户

默认 admin/escheduler123 

## 需要admin做的事情

### 租户与用户

这里租户对应 linux 系统中的用户

所以这里创建与当前用户同名的租户

这里创建普通用户 zhouhongfa/123

## 数据库修改时要做好sql的版本控制
![](https://i.loli.net/2019/11/29/PKgzvIbUpaeyWl7.png)
## 元数据框架Atlas2.2.0源码编译运行

## mac上编译

如果遇到错误:`Too many files with unapproved license`，可以加入参数`-Drat.skip=true`

## centos编译源码

 mvn clean -DskipTests package -Pdist

注意这里不要选内置的hbase和solr。

注意要在centos系统中用root用户编译。

### 启动依赖组件

外部启动zk

外部启动hbase

外部启动es



### 更改atlas配置

atlas-application.properties

```

```

atlas-env.sh

```

```





### 启动atlas

```
export HBASE_CONF_DIR=/home/huzekang/software/hbase-2.3.3/conf
export MANAGE_LOCAL_HBASE=false
export MANAGE_LOCAL_SOLR=false
bin/atlas_start.py
```



启动成功后，访问21000端口接口。

启动成功后，hbase会有两个表

```
atlas.graph.storage.hbase.table=apache_atlas_janus2
atlas.audit.hbase.tablename=apache_atlas_entity_audit
```





es会有三个索引

```
janusgraph_vertex_index
janusgraph_fulltext_index
janusgraph_edge_index
```


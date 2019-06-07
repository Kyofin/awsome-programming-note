# postgres数据库实践

## 常用操作

```sql
-- 创建用户
create role jsontest password 'jsontest.dba' createdb nosuperuser createrole login;
-- 创建数据库
create database jsontest owner jsontest encoding 'utf-8';
-- 创建全局序列
drop sequence if exists jsontest_uuid_seq cascade;
create sequence jsontest_uuid_seq start 1;
-- 创建表(谁创建的表就是谁的)
drop table if exists TUsers cascade;
create table TUsers(
 id bigint default nextval('jsontest_uuid_seq') primary key, --用户id
 realName character varying(64) -- 真实姓名
)WITH ( OIDS=FALSE );
-- 为表字段添加注释
comment on column tusers.id is '主键id';
comment on column tusers.realName is '真实姓名';
-- 为表添加注释
comment on table tusers is '用户表';
-- 为表添加索引
create index tusers_cellphone_idx on tusers(realName);

-- Table: TProject,用户创建的项目表
DROP TABLE IF EXISTS TProject CASCADE;
CREATE TABLE TProject (
 id bigint DEFAULT nextval('jsontest_uuid_seq') PRIMARY KEY,-- 活动id
 title character varying(256) NOT NULL UNIQUE, -- 项目名称,设置为UNIQUE，避免混淆
 creatorId integer DEFAULT NULL REFERENCES TUsers (id) match simple on delete SET NULL -- 活动创建的用户id  
 )WITH ( OIDS=FALSE );
CREATE INDEX TProject_creator_idx ON TProject( creatorId );

-- 新增测试数据 - 插入指定列
insert into TUsers(realname) values('test1');
-- 新增测试数据 - 插入所有列
insert into TUsers values(2,'test2');
insert into TUsers values(3,'test3');
insert into TProject values(1,'测试项目1',1);
insert into TProject values(2,'测试项目2',1);
insert into TProject values(3,'测试项目3',1);

-- 通过用户id获得用户JSON对象
WITH myInfo AS (select a.id,a.realName from TUsers a where a.id = 1 )
SELECT row_to_json(b.*) from myInfo b;

-- 数据库返回多行数据：如获取用户参与的项目
WITH myProjects AS (select a.id,a.title from TProject a where a.creatorId = 1)
SELECT row_to_json(b.*) from
(SELECT array_to_json(array(select row_to_json(myProjects.*)  from myProjects),false) as myProjects) b



```



## json字段放置数组对象

```sql

-- 创建演出活动表
DROP TABLE IF EXISTS TActivity CASCADE;
CREATE TABLE TActivity (
 id bigint   PRIMARY KEY,-- 活动id
 title character varying(128) NOT NULL,-- 活动名称  
 pricepackage jsonb NOT NULL -- 价格套餐,格式如：[{"packagename":"成人票","price":25,"stock":1000},{"packagename":"儿童票（12岁以下）","price":15,"stock":1000}]
)WITH (
 OIDS=FALSE
);




-- 新增-插入测试数据到演出活动表
insert into TActivity values(1,'演出活动标题1','[{"packagename":"成人票","price":189,"stock":100},{"packagename":"儿童票（12岁以下）","price":66,"stock":20},{"packagename":"成人+儿童套票","price":128,"stock":10}]');
insert into TActivity values(2,'演出活动标题2','[{"packagename":"成人票","price":189,"stock":100},{"packagename":"儿童票（5-15岁）","price":66,"stock":20},{"packagename":"成人+儿童套票","price":128,"stock":10}]');
insert into TActivity values(3,'演出活动标题3','[{"packagename":"成人票","price":99,"stock":100},{"packagename":"儿童票（3-5岁）","price":58,"stock":20},{"packagename":"成人+儿童套票","price":99,"stock":10}]');

-- 更新-插入对象到票价json数组中的最后
update tactivity set pricepackage = pricepackage || '{"packagename":"后补票","price":157,"stock":99}' where id =1;
-- 更新-插入对象到票价json数组中的最前
update tactivity set pricepackage ='{"packagename":"前补票","price":1237,"stock":99}' || pricepackage  where id =1;
-- 更新-插入对象到票价json数组中，-1表示从数组后面数第一个，true表示在该元素的后面插入。
update tactivity set pricepackage = jsonb_insert(pricepackage, '{-1}', '{"packagename":"最后成人票新增100","price":189,"stock":100}', true)    where id = 1;
-- 更新-更新json数组中指定对象，fasle代表从前面数起
update tactivity set pricepackage = jsonb_set(pricepackage,'{1}','{"price":78779,"packagename":"乘车票"}',false) where id =1;
-- 更新-更新json数组中指定对象的某一属性，fasle代表从前面数起
update tactivity set pricepackage = jsonb_set(pricepackage,'{1,packagename}','"奥特曼的票"',false) where id =1;
-- 更新-删除json数组中指定index的对象
update tactivity set pricepackage = pricepackage - 0 where id =1;
-- 更新-删除json数组中指定index的对象的某一属性
update tactivity set pricepackage = pricepackage #- '{0,price}' where id =1;

-- 查找-字符串匹配-找出条件等于“packagename = 儿童票（3-5岁）”的那些活动
select * from tactivity a where a.pricepackage @> '[{"packagename":"儿童票（3-5岁）"}]'::jsonb;
select * from tactivity a where a.pricepackage @> '[{"packagename":"成人票"}]';
-- 查找-数值比较-找出数组中第0个对象的price < 100的那些活动
select * from tactivity a where cast(a.pricepackage->0->>'price' as int)<100

-- 查找-数值比较-找出数组中所有对象的price < 100的那些活动
-- 1. 先把jsonb中的值取出来
CREATE type json_type_pricepackage AS (packagename text, price numeric, stock numeric);
SELECT id , (jsonb_populate_recordset(null::json_type_pricepackage, pricepackage)).* FROM tactivity;
-- 2. 再做过滤获得结果
WITH rows_filtered AS (
	SELECT DISTINCT id FROM (
		SELECT id ,(jsonb_populate_recordset(null::json_type_pricepackage, pricepackage)).* FROM tactivity 
	) a WHERE price > 150
)SELECT * FROM TActivity a WHERE a.id IN (SELECT * FROM rows_filtered)



select * from TActivity;
```



## 使用json字段做图片应用

![](https://raw.githubusercontent.com/huzekang/picbed/master/20190602224652.png)

![](http://blog.sciencenet.cn/data/attachment/album/201610/14/141149uunnqibz9ntir3qi.png)



```sql

-- 创建照片识别活动表（把照片识别看作一次活动，一次活动可能同时识别多张照片）
DROP TABLE IF EXISTS TPActivity CASCADE;
CREATE TABLE TPActivity (
 id bigint DEFAULT nextval('jsontest_uuid_seq') PRIMARY KEY,-- 活动id,全局唯一性，用于外键关联
 title character varying(128) NOT NULL,-- 活动名称，如“计算所喊你回家了”
 pics bigint ARRAY -- 照片id数组  
)WITH (
 OIDS=FALSE
);


-- 创建照片表
DROP TABLE IF EXISTS TPIC CASCADE;
CREATE TABLE TPIC (
 id bigint DEFAULT nextval('jsontest_uuid_seq') PRIMARY KEY,-- 活动id,全局唯一性，用于外键关联
 picUrl character varying(256) DEFAULT NULL,-- 照片
 persons jsonb NOT NULL -- 照片里的人名、电话等，格式为：[{"position":"前1排左1","name":"张晓陆","phone":"18910180100","city":"济南","organization":"中科院计算所","attendanceYear":"2012","graduationYear":"2016"},{"position":"前1排左2","name":"张晓陆","phone":"18910180100","organization":"中科院计算所","attendanceYear":"2012","graduationYear":"2016"}]
)WITH (
 OIDS=FALSE
);

-- 插入测试数据
-- Table: TPActivity
-- 照片识别活动表（把照片识别看作一次活动，一次活动可能同时识别多张照片）
insert into TPActivity values(18,'28级4班的你,还记得他/她吗？','{109, 110}');

-- Table: TPIC
-- 照片表
insert into TPIC values(109,'/img/userupload/XZZX-90-1.2.png',
'[{"position":"前排左1","name":"","phone":"","city":"","organization":"","attendanceYear":"","graduationYear":""},
{"position":"前排左2","name":"","phone":"","city":"","organization":"","attendanceYear":"","graduationYear":""}
]');
insert into TPIC values(110,'/img/userupload/XZZX-90-2.2.png',
'[{"position":"前排左1","name":"","phone":"","city":"","organization":"","attendanceYear":"","graduationYear":""},
{"position":"前排左2","name":"","phone":"","city":"","organization":"","attendanceYear":"","graduationYear":""},
{"position":"前排左3","name":"","phone":"","city":"","organization":"","attendanceYear":"","graduationYear":""}
]');

-- 查询-所有的和活动相关的图片
WITH allPicsInOneActivity AS (select a.title,b.id,b.picurl,b.persons from TPActivity a, TPIC b where a.id =18 and b.id = any ( a.pics) )
select jsonb_pretty(to_jsonb(row_to_json(b.*))) from
(SELECT array_to_json(array(select row_to_json(allPicsInOneActivity.*) from allPicsInOneActivity),false) as activityallpics) b

```

切换jupyter内的python版本
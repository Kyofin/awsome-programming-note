# 利用transform转换数据

## 运行模型

![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/datax/datax/b5652c0492c394684958272219ce327c/image.png)



## 参考配置

```


{
  "job": {
    "setting": {
      "speed": {
        "channel": 1
      },
      "errorLimit": {
        "record": 0
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "root",
            "password": "root",
            "column": [

              "FIRST_OPENED_DATE",
              "STORE_PHONE",
              "STORE_ID",
              "STORE_STATE",
              "STORE_STREET_ADDRESS"
            ],
            "splitPk": "STORE_ID",
            "connection": [
              {
                "table": [
                  "store"
                ],
                "jdbcUrl": [
                  "jdbc:mysql://192.168.1.150:3306/foodmart"
                ]
              }
            ]
          }
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": true,
            "encoding": "UTF-8"
          }
        },
        "transformer": [
          {
            "name": "dx_substr",
            "parameter":
            {
              "columnIndex":0,
              "paras":["0","4"]
            }
          },
          {
            "name": "dx_replace",
            "parameter":
            {
              "columnIndex":1,
              "paras":["3","4","****"]
            }
          },
          {
            "name": "dx_pad",
            "parameter":
            {
              "columnIndex":2,
              "paras":["r","10","A"]
            }
          },
          {
            "name": "dx_filter",
            "parameter":
            {
              "columnIndex":3,
              "paras":["=","CA"]
            }
          },
          {
            "name": "dx_filter",
            "parameter":
            {
              "columnIndex":3,
              "paras":["=","WA"]
            }
          },
          {
            "name": "dx_groovy",
            "parameter":
            {
              "code": "Column column = record.getColumn(4);     String oriValue = column.asString();  String newValue = \"********\" + oriValue.substring(7, oriValue.length()); record.setColumn(4, new StringColumn(newValue));  return record;",
              "extraPackage":[

              ]
            }
          }
        ]
      }
    ]
  }
}
```

## 样本数据

![](https://i.loli.net/2019/12/04/VwfOahxrjbQMm2P.png)

## 处理结果

![](https://i.loli.net/2019/12/04/7E5SGH3LBrisagR.png)

这里对`FIRST_OPENED_DATE`字段数据进行了截断操作，对`STORE_PHONE`进行了替换操作，实现了脱敏数据。

对`STORE_ID`进行了填充操作，可以用于对id之类值统一规范。

可以看到结果是少了的，因为在配置中加入两个`datax-filter`的转换器，它过滤掉了`STORE_STATE`等于WA和CA的数据。

其中转换器`dx_groovy`为我们提供了动态编译代码生成新转换规则的功能。它只能加一次，而它也能对所有列进行操作的，所以配置时不需要知道操作哪一列数据。

我们也可以通过`dx_groovy`转换器引入外部包（datax项目中要有这个jar才可以），来实现更丰富的转换功能。如我们现在把最后一列的数据用uuid代替。

**配置`dx_groovy`转换器:**

```
 {
            "name": "dx_groovy",
            "parameter":
            {
              "code": " Column column = record.getColumn(4); String newValue = IdUtil.fastUUID() ;record.setColumn(4, new StringColumn(newValue));  return record;",
              "extraPackage":[
                "import cn.hutool.core.util.IdUtil;"
              ]
            }
          }
```

![](http://image-picgo.test.upcdn.net/img/20191204150034.png)


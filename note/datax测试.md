同步oracle指标表到postgresql

```json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 16
      }
    },
    "content": [
      {
        "reader": {
          "name": "oraclereader",
          "parameter": {
            "splitPk": "id",
            "username": "yibo",
            "password": "yibo123",
            "column": [
              "*"

            ],
            "connection": [
              {
                "table": [
                  "TB_LIS_INDICATORS"
				  
                ],
                "jdbcUrl": [
                  "jdbc:oracle:thin:@10.91.1.5:1521:gzfy"
                ]
              }
            ]
          }
        },
        "writer": {
          "name": "postgresqlwriter",
          "parameter": {
           
            "username": "yibo",
            "password": "yibo123",
            "column": [
              "*"
            ],
            "batchSize": "4096",
            "connection": [
              {
                "jdbcUrl": "jdbc:postgresql://10.91.1.4:5432/postgres",
                "table": [
                  "tb_lis_indicators"
                ]
              }
            ]
          }
        }
      }
    ]
  }
}
```



- json配置文件中channel 设置16 
- json配置文件中oracle读配置设置了`"splitPk": "id"`

- 启动命令 `python datax.py  --jvm="-Xms8G -Xmx8G" 1.josn`

  防止多线程并发时缓存过大导致频繁full gc或者oom错误

**结论：每秒可以达到9w。**

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190413173035.png)
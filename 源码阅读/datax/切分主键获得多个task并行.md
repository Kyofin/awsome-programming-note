# 切分主键获得多个task并行

当我们设置jdbc job的channel大于1时，datax是会帮我们切分task。

![](https://i.loli.net/2019/12/04/2zECpQw7BTUD1x4.png)

datax是没有限制切分依据列的类型的，所以无论是varchar和int都可以作为切分依据。

## bigint作为切割主键

这里选择一个自增的数字列作为切分依据。我们一般对数字都是这样确定范围的。其中有一条是查null的，所以一共11个task。

![Snipaste_2019-12-04_12-15-54.png](https://i.loli.net/2019/12/04/4GtYAyQN37SV6a9.png)



## varchar作为切割主键

这里选择varchar列作为切分依据。

![](https://i.loli.net/2019/12/04/nrY9RLot8SUklXA.png)

它的值是这样的。

![](https://i.loli.net/2019/12/04/V2SQPHxgkcibm1s.png)

datax会这样处理。执行了`SELECT MIN(STORE_NAME),MAX(STORE_NAME) FROM store;`获取最大和最小值，然后根据值的编码值确定每个task的查询范围。

![](https://i.loli.net/2019/12/04/pmL7toUVeQNEnHk.png)

但这样切割是不均匀的。查询过每个query sql，只有其中两条是有数据的，而且数据倾斜很严重。

![](https://i.loli.net/2019/12/04/yrb3qF6AHNsZnpD.png)
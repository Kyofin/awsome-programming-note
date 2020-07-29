## [翻译] What are Window Functions?

参考文档：https://databricks.com/blog/2015/07/15/introducing-window-functions-in-spark-sql.html



Before 1.4, there were two kinds of functions supported by Spark SQL that could be used to calculate a single return value. *Built-in functions* or *UDFs*, such as `substr` or `round`, take values from a single row as input, and they generate a single return value for every input row. *Aggregate functions,* such as `SUM` or `MAX`*,* operate on a group of rows and calculate a single return value for every group.

###### 1.4之前，只有两种内置函数，一种一进一出，如substr。另一种是聚合函数，如sum，操作一组中多行数据输出一行数据为每组的数据。

While these are both very useful in practice, there is still a wide range of operations that cannot be expressed using these types of functions alone. Specifically, there was no way to both operate on a group of rows while still returning a single value for every input row. This limitation makes it hard to conduct various data processing tasks like calculating a moving average, calculating a cumulative sum, or accessing the values of a row appearing before the current row. Fortunately for users of Spark SQL, window functions fill this gap.

###### 虽然还蛮好用到，但是没有办法分组计算多行的同时依旧按输入多少行就输出多少行。这样的限制就使得计算如移动平均值，累计，访问出现在当前行之前的值的任务十分困难。

At its core, a window function calculates a return value for every input row of a table based on a group of rows, called the *Frame*. Every input row can have a unique frame associated with it. This characteristic of window functions makes them more powerful than other functions and allows users to express various data processing tasks that are hard (if not impossible) to be expressed without window functions in a concise way. Now, let’s take a look at two examples.

###### 核心来了，一个窗口函数根据每一行组内的数据计算并返回一个值，我们称为框。组内每一行输入数据都有与自己关联的特定框。

Suppose that we have a *productRevenue* table as shown below.

###### 假设我们有一张产品收益表。
![1-1](https://databricks.com/wp-content/uploads/2015/07/1-1.png)

We want to answer two questions:

###### 我们想知道两个问题的答案：

1. What are the best-selling and the second best-selling products in every category?

   ###### 找出每个分类中销售第一名和第二名的产品

2. What is the difference between the revenue of each product and the revenue of the best-selling product in the same category of that product?

   ###### 找出每个分类中销售第一名和第二名的产品

To answer the first question “*What are the best-selling and the second best-selling products in every category?*”, we need to rank products in a category based on their revenue, and to pick the best selling and the second best-selling products based the ranking. Below is the SQL query used to answer this question by using window function `dense_rank` (we will explain the syntax of using window functions in next section).

###### 为了回答第一个问题，我们需要对每个分类里的产品按收益进行排名，然后根据排名找出第一名和第二名。下面这条sql就是答案，用了dense_rank这个窗口函数。可不能用rank喔~！

```
SELECT
  product,
  category,
  revenue
FROM (
  SELECT
    product,
    category,
    revenue,
    dense_rank() OVER (PARTITION BY category ORDER BY revenue DESC) as rank
  FROM productRevenue) tmp
WHERE
  rank <= 2
```

The result of this query is shown below. Without using window functions, it is very hard to express the query in SQL, and even if a SQL query can be expressed, it is hard for the underlying engine to efficiently evaluate the query.

###### 下面就是查询的结果展示。没有窗口函数的话，它是很难用sql去表达这个查询的，即使有一个sql可以表达这个查询，它也是很难被计算引擎高效的执行的。

![1-2](https://databricks.com/wp-content/uploads/2015/07/1-2.png)

For the second question “*What is the difference between the revenue of each product and the revenue of the best selling product in the same category as that product?*”, to calculate the revenue difference for a product, we need to find the highest revenue value from products in the same category for each product. Below is a Python DataFrame program used to answer this question.

###### 为了回答第二个问题，我们需要计算出分类里每一个产品自身的收益和该类产品中最好收益的那个之间的差距。下面用pyspark来解答这个问题。

```
import sys
from pyspark.sql.window import Window
import pyspark.sql.functions as func
windowSpec = \
  Window 
    .partitionBy(df['category']) \
    .orderBy(df['revenue'].desc()) \
    .rangeBetween(-sys.maxsize, sys.maxsize)
dataFrame = sqlContext.table("productRevenue")
revenue_difference = \
  (func.max(dataFrame['revenue']).over(windowSpec) - dataFrame['revenue'])
dataFrame.select(
  dataFrame['product'],
  dataFrame['category'],
  dataFrame['revenue'],
  revenue_difference.alias("revenue_difference"))
```

The result of this program is shown below. Without using window functions, users have to find all highest revenue values of all categories and then join this derived data set with the original *productRevenue* table to calculate the revenue differences.

###### 程序的结果如下展示。如果没有使用窗口函数，用户只能通过找到分类中最高的收益，然后通过join的方式连接来计算差距。

![1-3](https://databricks.com/wp-content/uploads/2015/07/1-3.png)

## Using Window Functions

Spark SQL supports three kinds of window functions: ranking functions, analytic functions, and aggregate functions. The available ranking functions and analytic functions are summarized in the table below. For aggregate functions, users can use any existing aggregate function as a window function.

###### spark sql支持三种窗口函数：排名、分析、聚合。排名和分析的函数可以在下表中找到。而聚合函数，用户可以使用已存在的聚合函数。

|                        | **SQL**     | **DataFrame API** |
| ---------------------- | ----------- | ----------------- |
| **Ranking functions**  | rank        | rank              |
| dense_rank             | denseRank   |                   |
| percent_rank           | percentRank |                   |
| ntile                  | ntile       |                   |
| row_number             | rowNumber   |                   |
| **Analytic functions** | cume_dist   | cumeDist          |
| first_value            | firstValue  |                   |
| last_value             | lastValue   |                   |
| lag                    | lag         |                   |
| lead                   | lead        |                   |

To use window functions, users need to mark that a function is used as a window function by either

- Adding an *OVER* clause after a supported function in SQL, e.g. `avg(revenue) OVER (...)`; or
- Calling the *over* method on a supported function in the DataFrame API, e.g. `rank().over(...)`*.*

###### 我们可以通过sql和df的方式来使用窗口函数。

Once a function is marked as a window function, the next key step is to define the *Window Specification* associated with this function. A window specification defines which rows are included in the frame associated with a given input row. A window specification includes three parts:

###### 一旦使用窗口函数，下一步就是要声明与之关联的窗口定义。一个窗口的规范声明了哪些行与给定行的关联。一个窗口的定义分了三部分：

1. Partitioning Specification: controls which rows will be in the same partition with the given row. Also, the user might want to make sure all rows having the same value for  the category column are collected to the same machine before ordering and calculating the frame.  If no partitioning specification is given, then all data must be collected to a single machine.

   ###### 分区定义：控制哪些行被划分到与给定行同一个分区中。而且，用户可能希望在排序和计算帧之前，保证所有同分类的数据都收集到相同机器中。如果没有提供分区定义，所有的数据必须收集到相同的一台机器中。

2. Ordering Specification: controls the way that rows in a partition are ordered, determining the position of the given row in its partition.

   ###### 排序定义：控制在一个分区内的数据是有顺序的，指明指定行在分区中的位置

3. Frame Specification: states which rows will be included in the frame for the current input row, based on their relative position to the current row.  For example, “the three rows preceding the current row to the current row” describes a frame including the current input row and three rows appearing before the current row.

   ###### 框定义：指定哪些行是在当前输入行的框中的，根据他们与当前行的相对位置。例如the three rows preceding the current row to the current row 描述一个框：当前行和它的前三行

In SQL, the `PARTITION BY` and `ORDER BY` keywords are used to specify partitioning expressions for the partitioning specification, and ordering expressions for the ordering specification, respectively. The SQL syntax is shown below.

```
OVER (PARTITION BY ... ORDER BY ...)
```

In the DataFrame API, we provide utility functions to define a window specification. Taking Python as an example, users can specify partitioning expressions and ordering expressions as follows.

```
from pyspark.sql.window import Window
windowSpec = \
  Window \
    .partitionBy(...) \
    .orderBy(...)
```

In addition to the ordering and partitioning, users need to define the start boundary of the frame, the end boundary of the frame, and the type of the frame, which are three components of a frame specification.

There are five types of boundaries, which are` UNBOUNDED PRECEDING`, `UNBOUNDED FOLLOWING`, `CURRENT ROW`, ` PRECEDING`, and ` FOLLOWING`. `UNBOUNDED PRECEDING` and `UNBOUNDED FOLLOWING` represent the first row of the partition and the last row of the partition, respectively. For the other three types of boundaries, they specify the offset from the position of the current input row and their specific meanings are defined based on the type of the frame. There are two types of frames, *ROW* frame and *RANGE* frame.

###### 上面描述的是框的范围定义。

**ROW frame**

ROW frames are based on physical offsets from the position of the current input row, which means that `CURRENT ROW`, ` PRECEDING`, or ` FOLLOWING` specifies a physical offset. If `CURRENT ROW` is used as a boundary, it represents the current input row. ` PRECEDING` and ` FOLLOWING` describes the number of rows appear before and after the current input row, respectively. The following figure illustrates a ROW frame with a` 1 PRECEDING` as the start boundary and `1 FOLLOWING` as the end boundary (`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` in the SQL syntax).

###### 根据行定义框。

![2-1](https://databricks.com/wp-content/uploads/2015/07/2-1-1024x338.png)

**RANGE frame**

RANGE frames are based on logical offsets from the position of the current input row, and have similar syntax to the ROW frame. A logical offset is the difference between the value of the ordering expression of the current input row and the value of that same expression of the boundary row of the frame. Because of this definition, when a RANGE frame is used, only a single ordering expression is allowed. Also, for a RANGE frame, all rows having the same value of the ordering expression with the current input row are considered as same row as far as the boundary calculation is concerned.

Now, let’s take a look at an example. In this example, the ordering expressions is `revenue`; the start boundary is `2000 PRECEDING`; and the end boundary is `1000 FOLLOWING` (this frame is defined as `RANGE BETWEEN 2000 PRECEDING AND 1000 FOLLOWING` in the SQL syntax). The following five figures illustrate how the frame is updated with the update of the current input row. Basically, for every current input row, based on the value of revenue, we calculate the revenue range `[current revenue value - 2000, current revenue value + 1000]`. All rows whose revenue values fall in this range are in the frame of the current input row.

###### 根据值的范围定义框。

![2-2](https://databricks.com/wp-content/uploads/2015/07/2-2-1024x369.png)

![2-3](https://databricks.com/wp-content/uploads/2015/07/2-3-1024x263.png)

![2-4](https://databricks.com/wp-content/uploads/2015/07/2-4-1024x263.png)

![2-5](https://databricks.com/wp-content/uploads/2015/07/2-5-1024x263.png)

![2-6](https://databricks.com/wp-content/uploads/2015/07/2-6-1024x263.png)

In summary, to define a window specification, users can use the following syntax in SQL.

```
OVER (PARTITION BY ... ORDER BY ... frame_type BETWEEN start AND end)
```

Here, `frame_type` can be either ROWS (for ROW frame) or RANGE (for RANGE frame); `start` can be any of `UNBOUNDED PRECEDING`, `CURRENT ROW`, ` PRECEDING`, and ` FOLLOWING`; and `end` can be any of `UNBOUNDED FOLLOWING`, `CURRENT ROW`, ` PRECEDING`, and ` FOLLOWING.`

In the Python DataFrame API, users can define a window specification as follows.

```
from pyspark.sql.window import Window
# Defines partitioning specification and ordering specification.
windowSpec = \
  Window \
    .partitionBy(...) \
    .orderBy(...)
# Defines a Window Specification with a ROW frame.
windowSpec.rowsBetween(start, end)
# Defines a Window Specification with a RANGE frame.
windowSpec.rangeBetween(start, end)
```

## 
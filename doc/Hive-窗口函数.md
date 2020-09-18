
# 排名函数示例
## 样本数据
```
id		    math	classid	    departmentid
1		        69	    class1	department1
2		        80	    class1	department1
3		        74	    class1	department1
4		        94	    class1	department1
5		        93	    class1	department1
6		        74	    class2	department1
7		        86	    class2	department1
8		        78	    class2	department1
9		        70	    class2	department1
10		        93	    class1	department2
11		        83	    class1	department2
12		        94	    class1	department2
13		        94	    class1	department2
14		        82	    class1	department2
15		        74	    class1	department2
16		        99	    class2	department2
17		        78	    class2	department2
18		        74	    class2	department2
19		        80	    class2	department2
20		        85	    class2	department2
```

## 每个班级中数学第一名和第二名?

### 错误答案：
```
select id ,math ,classid ,rank_n from (
select id ,math ,classid ,row_number()  over (partition by classid order by math desc)  as rank_n
from student_scores 
) a  where rank_n <=2;

id	math	classid	rank_n
16	99	class2	1
7	86	class2	2
4	94	class1	1
12	94	class1	2
```

### 错误答案：
```
select id ,math ,classid ,rank_n from (
select id ,math ,classid ,rank(math)  over (partition by classid order by math desc)  as rank_n
from student_scores 
) a  where rank_n <=2;

id	math	classid	rank_n
16	99	class2	1
7	86	class2	2
4	94	class1	1
12	94	class1	1
13	94	class1	1

```

### 正确答案：
```
select id ,math ,classid ,rank_n from (
select id ,math ,classid ,dense_rank(math)  over (partition by classid order by math desc)  as rank_n
from student_scores 
) a  where rank_n <=2;


id	math	classid	rank_n
16	99	class2	1
7	86	class2	2
4	94	class1	1
12	94	class1	1
13	94	class1	1
5	93	class1	2
10	93	class1	2

```


## 每个班级中每个同学的数学成绩和数学第一名的分数差距？

### 错误答案：
```
select id  ,classid ,math,max(math)  over (partition by classid order by math )  as max_math
from student_scores ;

id	classid	math	max_math
9	class2	70	70
6	class2	74	74
18	class2	74	74
8	class2	78	78
17	class2	78	78
19	class2	80	80
20	class2	85	85
7	class2	86	86
16	class2	99	99
1	class1	69	69
3	class1	74	74
15	class1	74	74
2	class1	80	80
14	class1	82	82
11	class1	83	83
5	class1	93	93
10	class1	93	93
4	class1	94	94
12	class1	94	94
13	class1	94	94
```


### 正确答案：
```
select id  ,classid ,math ,(max_math -math) grap from
(
select id  ,classid ,math,max(math)  over (partition by classid )  as max_math
from student_scores 
)

id	classid	math	grap
6	class2	74	25
7	class2	86	13
8	class2	78	21
9	class2	70	29
16	class2	99	0
17	class2	78	21
18	class2	74	25
19	class2	80	19
20	class2	85	14
1	class1	69	25
2	class1	80	14
3	class1	74	20
4	class1	94	0
5	class1	93	1
10	class1	93	1
11	class1	83	11
12	class1	94	0
13	class1	94	0
14	class1	82	12
15	class1	74	20
```



# python编码问题



## 字符串编码常用模式

![](http://image-picgo.test.upcdn.net/img/20200130021057.png)



## python2

### **重点**：

调用encode方法时，**前面**的对象一定要是**unicode**编码格式，否则会报错。



### 例子：

字符串直接encode会出错

```python
>>> sys.getdefaultencoding()
'ascii'
>>> s='我爱python'
>>> s.encode('utf-8')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 0: ordinal not in range(128)
```



字符串需要先decode再encode才不会报错。因为decode会转成**unicode**编码。

```python
>>> s.decode('utf-8').encode('utf-8')
'\xe6\x88\x91\xe7\x88\xb1python'
```



u开头的字符串可以直接encode

```python
>>> su=u'我爱python'
>>> su.encode('utf-8')
'\xe6\x88\x91\xe7\x88\xb1python'
```



## python3

### **重点**：

python3默认**内部全部用unicode编码**，所以定义字符串都是unicode。

所以可以直接用encode方法。 



### 例子：

```python
>>> s='我爱python'
>>> s.encode('utf-8')
b'\xe6\x88\x91\xe7\x88\xb1python'
```


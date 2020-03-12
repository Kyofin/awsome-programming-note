# JDBC数据类型

## 参考文档

https://www.cnblogs.com/firstdream/p/10078837.html

## sql类型对应jdbc类型/java类型

JDBC驱动程序将Java数据类型转换为适当的JDBC类型，然后将其发送到数据库。 它为大多数数据类型提供并使用默认映射。 例如，Java `int`类型会被转换为SQL `INTEGER`。 创建默认映射以提供到驱动程序时保持一致性。

下表总结了当调用`PreparedStatement`或`CallableStatement`对象或`ResultSet.updateXXX()`方法的`setXXX()`方法时，将Java数据类型转换为的默认JDBC数据类型。

| SQL类型     | JDBC/Java类型        | setXXX        | updateXXX        |
| ----------- | -------------------- | ------------- | ---------------- |
| VARCHAR     | java.lang.String     | setString     | updateString     |
| CHAR        | java.lang.String     | setString     | updateString     |
| LONGVARCHAR | java.lang.String     | setString     | updateString     |
| BIT         | boolean              | setBoolean    | updateBoolean    |
| NUMERIC     | java.math.BigDecimal | setBigDecimal | updateBigDecimal |
| TINYINT     | byte                 | setByte       | updateByte       |
| SMALLINT    | short                | setShort      | updateShort      |
| INTEGER     | int                  | setInt        | updateInt        |
| BIGINT      | long                 | setLong       | updateLong       |
| REAL        | float                | setFloat      | updateFloat      |
| FLOAT       | float                | setFloat      | updateFloat      |
| DOUBLE      | double               | setDouble     | updateDouble     |
| VARBINARY   | byte[ ]              | setBytes      | updateBytes      |
| BINARY      | byte[ ]              | setBytes      | updateBytes      |
| DATE        | java.sql.Date        | setDate       | updateDate       |
| TIME        | java.sql.Time        | setTime       | updateTime       |
| TIMESTAMP   | java.sql.Timestamp   | setTimestamp  | updateTimestamp  |
| CLOB        | java.sql.Clob        | setClob       | updateClob       |
| BLOB        | java.sql.Blob        | setBlob       | updateBlob       |
| ARRAY       | java.sql.Array       | setARRAY      | updateARRAY      |
| REF         | java.sql.Ref         | SetRef        | updateRef        |
| STRUCT      | java.sql.Struct      | SetStruct     | updateStruct     |

JDBC 3.0增强了对`BLOB`，`CLOB`，`ARRAY`和`REF`数据类型的支持。 `ResultSet`对象现在具有`updateBLOB()`，`updateCLOB()`，`updateArray()`和`updateRef()`方法，使您能够直接操作数据库服务器上的相应数据。

`setXXX()`和`updateXXX()`方法可以将特定的Java类型转换为特定的JDBC数据类型。 方法`setObject()`和`updateObject()`可以将几乎任何Java类型映射到JDBC数据类型。

`ResultSet`对象为每个数据类型提供相应的`getXXX()`方法来检索列值。每个方法都可以使用列名或其序数位置来检索列值。

| SQL类型     | JDBC/Java类型        | setXXX        | updateXXX     |
| ----------- | -------------------- | ------------- | ------------- |
| CHAR        | java.lang.String     | setString     | getString     |
| VARCHAR     | java.lang.String     | setString     | getString     |
| LONGVARCHAR | java.lang.String     | setString     | getString     |
| BIT         | boolean              | setBoolean    | getBoolean    |
| NUMERIC     | java.math.BigDecimal | setBigDecimal | getBigDecimal |
| TINYINT     | byte                 | setByte       | getByte       |
| SMALLINT    | short                | setShort      | getShort      |
| INTEGER     | int                  | setInt        | getInt        |
| BIGINT      | long                 | setLong       | getLong       |
| REAL        | float                | setFloat      | getFloat      |
| FLOAT       | float                | setFloat      | getFloat      |
| DOUBLE      | double               | setDouble     | getDouble     |
| VARBINARY   | byte[ ]              | setBytes      | getBytes      |
| BINARY      | byte[ ]              | setBytes      | getBytes      |
| DATE        | java.sql.Date        | setDate       | getDate       |
| TIME        | java.sql.Time        | setTime       | getTime       |
| TIMESTAMP   | java.sql.Timestamp   | setTimestamp  | getTimestamp  |
| CLOB        | java.sql.Clob        | setClob       | getClob       |
| BLOB        | java.sql.Blob        | setBlob       | getBlob       |
| ARRAY       | java.sql.Array       | setARRAY      | getARRAY      |
| REF         | java.sql.Ref         | SetRef        | getRef        |
| STRUCT      | java.sql.Struct      | SetStruct     | getStruct     |

## 日期和时间数据类型

`java.sql.Date`类映射到SQL `DATE`类型，`java.sql.Time`和`java.sql.Timestamp`类分别映射到SQL `TIME`和SQL `TIMESTAMP`数据类型。

以下示例显示了`Date`和`Time`类如何格式化为标准Java日期和时间值以匹配SQL数据类型要求。



```java
import java.sql.Date;
import java.sql.Time;
import java.sql.Timestamp;
import java.util.*;

public class SqlDateTime {
   public static void main(String[] args) {
      //Get standard date and time
      java.util.Date javaDate = new java.util.Date();
      long javaTime = javaDate.getTime();
      System.out.println("The Java Date is:" + 
             javaDate.toString());

      //Get and display SQL DATE
      java.sql.Date sqlDate = new java.sql.Date(javaTime);
      System.out.println("The SQL DATE is: " + 
             sqlDate.toString());

      //Get and display SQL TIME
      java.sql.Time sqlTime = new java.sql.Time(javaTime);
      System.out.println("The SQL TIME is: " + 
             sqlTime.toString());
      //Get and display SQL TIMESTAMP
      java.sql.Timestamp sqlTimestamp =
      new java.sql.Timestamp(javaTime);
      System.out.println("The SQL TIMESTAMP is: " + 
             sqlTimestamp.toString());
     }//end main
}//end SqlDateTime


Java
```

编译并执行上面代码，得到以下结果 -



```shell
F:\worksp\jdbc>javac SqlDateTime.java

F:\worksp\jdbc>java SqlDateTime
The Java Date is:Wed May 31 23:54:57 CST 2017
The SQL DATE is: 2017-05-31
The SQL TIME is: 23:54:57
The SQL TIMESTAMP is: 2017-05-31 23:54:57.937

F:\worksp\jdbc>


Shell
```

## 处理NULL值

SQL使用`NULL`值和Java使用`null`是不同的概念。 所以，要在Java中处理SQL `NULL`值，可以使用三种策略 -

- 避免使用返回原始数据类型的`getXXX()`方法。
- 对原始数据类型使用包装类，并使用`ResultSet`对象的`wasNull()`方法来测试接收`getXXX()`方法的返回值的包装器类变量是否应设置为`null`。
- 使用原始数据类型和`ResultSet`对象的`wasNull()`方法来测试接收到由`getXXX()`方法返回的值的原始变量是否应设置为表示`NULL`的可接受值。

下面是一个用来处理`NULL`值的例子 -



```java
Statement stmt = conn.createStatement( );
String sql = "SELECT id, first, last, age FROM Employees";
ResultSet rs = stmt.executeQuery(sql);

int id = rs.getInt(1);
if( rs.wasNull( ) ) {
   id = 0;
}
```
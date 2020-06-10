# spark如何自定义外部数据源

关注接口：BaseRelation

```scala
abstract class BaseRelation {
  def sqlContext: SQLContext
  def schema: StructType

```



关注类：JdbcRelationProvider

```scala
class JdbcRelationProvider extends CreatableRelationProvider
  with RelationProvider with DataSourceRegister {

  override def shortName(): String = "jdbc"
```

这里的**shortName**即spark读数据源时指定的format。



```scala
  case SaveMode.Overwrite =>
            if (options.isTruncate && isCascadingTruncateTable(options.url) == Some(false)) {
              // In this case, we should truncate table and then load.
              truncateTable(conn, options)
              val tableSchema = JdbcUtils.getSchemaOption(conn, options)
              saveTable(df, tableSchema, isCaseSensitive, options)
            } else {
              // Otherwise, do not truncate the table, instead drop and recreate it
              dropTable(conn, options.table)
              createTable(conn, df, options)
              saveTable(df, Some(df.schema), isCaseSensitive, options)
            }

          case SaveMode.Append =>
            val tableSchema = JdbcUtils.getSchemaOption(conn, options)
            saveTable(df, tableSchema, isCaseSensitive, options)

          case SaveMode.ErrorIfExists =>
            throw new AnalysisException(
              s"Table or view '${options.table}' already exists. SaveMode: ErrorIfExists.")

          case SaveMode.Ignore =>
```

这里是jdbc写出时不同mode的实现。





关注类：JDBCRDD

```scala
def scanTable(
      sc: SparkContext,
      schema: StructType,
      requiredColumns: Array[String],
      filters: Array[Filter],
      parts: Array[Partition],
      options: JDBCOptions): RDD[InternalRow] = {
    val url = options.url
    val dialect = JdbcDialects.get(url)
    val quotedColumns = requiredColumns.map(colName => dialect.quoteIdentifier(colName))
    new JDBCRDD(
      sc,
      JdbcUtils.createConnectionFactory(options),
      pruneSchema(schema, requiredColumns),
      quotedColumns,
      filters,
      parts,
      url,
      options)
  }
```


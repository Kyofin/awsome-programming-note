## 设置索引的映射mapping

#### 如果没有加@Field注解在对象的属性上，是不会在建立mapping时加入该属性的。

具体逻辑可以参考下面👇代码中的

```java
//如果没有加@Field注解在对象的属性上，此处为null
Field singleField = field.getAnnotation(Field.class);
	.....
	//此段代码是判断是否要将对象属性加入到mapping中
	if (isRootObject && singleField != null && isIdField(field, idFieldName)) {
				applyDefaultIdFieldMapping(xContentBuilder, field);
			} else if (multiField != null) {
				addMultiFieldMapping(xContentBuilder, field, multiField, isNestedOrObjectField(field));
			} else if (singleField != null) {
				addSingleFieldMapping(xContentBuilder, field, singleField, isNestedOrObjectField(field));
			}
```

`field.getAnnotation(Field.class)`为null值时，无法执行方法

`addSingleFieldMapping(xContentBuilder, field, singleField, isNestedOrObjectField(field))`



#### springboot发送设置mapping的请求到es服务器

ElasticsearchTemplate

```java
@Override
	public boolean putMapping(String indexName, String type, Object mapping) {
		Assert.notNull(indexName, "No index defined for putMapping()");
		Assert.notNull(type, "No type defined for putMapping()");
		PutMappingRequestBuilder requestBuilder = client.admin().indices().preparePutMapping(indexName).setType(type);
		if (mapping instanceof String) {
			requestBuilder.setSource(String.valueOf(mapping));
		} else if (mapping instanceof Map) {
			requestBuilder.setSource((Map) mapping);
		} else if (mapping instanceof XContentBuilder) {
			requestBuilder.setSource((XContentBuilder) mapping);
		}
    //发送请求
		return requestBuilder.execute().actionGet().isAcknowledged();
	}
```

requestBuilder的source属性可以看到发送请求的body

![image-20190325171754515](/Users/huzekang/Library/Application Support/typora-user-images/image-20190325171754515.png)





## 插入数据到es服务器

#### springboot是利用jackson将对象转成字符串，再发送给es的。

`ElasticsearchTemplate`中方法`IndexRequestBuilder prepareIndex(IndexQuery query)` 

```java
if (query.getObject() != null) {
				String id = StringUtils.isEmpty(query.getId()) ? getPersistentEntityId(query.getObject()) : query.getId();
				// If we have a query id and a document id, do not ask ES to generate one.
				if (id != null) {
					indexRequestBuilder = client.prepareIndex(indexName, type, id);
				} else {
					indexRequestBuilder = client.prepareIndex(indexName, type);
				}
//此处用到jackson转换成字符串				indexRequestBuilder.setSource(resultsMapper.getEntityMapper().mapToString(query.getObject()));
			} else if (query.getSource() != null) {
				indexRequestBuilder = client.prepareIndex(indexName, type, query.getId()).setSource(query.getSource());
			} else {
				throw new ElasticsearchException(
						"object or source is null, failed to index the document [id: " + query.getId() + "]");
			}
```

Debug可以看到转换成字符串的结果

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190325153307.png)



#### 因此如果想忽略某些对象的属性，可以在对象中使用jackson的注解。

例如在实体类`PatientEntityExt`上加上注解

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190325153812.png)

即可在oject转换成字符串时忽略这些属性。 



## 批量索引bulk方法底层实现

saveAll方法用的其实是`ElasticsearchTemplate`中的bulkIndex

```
@Override
	public void bulkIndex(List<IndexQuery> queries) {
		BulkRequestBuilder bulkRequest = client.prepareBulk();
		for (IndexQuery query : queries) {
			bulkRequest.add(prepareIndex(query));
		}
		checkForBulkUpdateFailure(bulkRequest.execute().actionGet());
	}
```

循环将list的每个对象add到包装的请求体中，上面add方法最后会放入一个list中

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190326142840.png)


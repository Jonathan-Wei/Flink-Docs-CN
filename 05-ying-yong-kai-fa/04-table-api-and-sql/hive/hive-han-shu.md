# Hive函数

## Hive用户定义的函数

用户可以在Flink中使用现有的Hive用户定义函数。

支持的UDF类型包括：

* UDF
* GenericUDF
* GenericUDTF
* UDAF
* GenericUDAFResolver2

在查询计划和执行时，Hive的UDF和GenericUDF会自动转换为Flink的ScalarFunction，Hive的GenericUDTF会自动转换为Flink的TableFunction，而Hive的UDAF和GenericUDAFResolver2会被转换为Flink的AggregateFunction。

要使用Hive用户定义函数，用户必须这样做

* 设置由Hive Metastore支持的HiveCatalog，该HiveCatalog包含该功能作为会话的当前目录
* 包含一个在Flink的类路径中包含该函数的jar
* 使用Blink planner。

## 使用Hive用户定义的函数

假设我们在Hive Metastore中注册了以下Hive函数：

```java
/**
 * Test simple udf. Registered under name 'myudf'
 */
public class TestHiveSimpleUDF extends UDF {

	public IntWritable evaluate(IntWritable i) {
		return new IntWritable(i.get());
	}

	public Text evaluate(Text text) {
		return new Text(text.toString());
	}
}

/**
 * Test generic udf. Registered under name 'mygenericudf'
 */
public class TestHiveGenericUDF extends GenericUDF {

	@Override
	public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
		checkArgument(arguments.length == 2);

		checkArgument(arguments[1] instanceof ConstantObjectInspector);
		Object constant = ((ConstantObjectInspector) arguments[1]).getWritableConstantValue();
		checkArgument(constant instanceof IntWritable);
		checkArgument(((IntWritable) constant).get() == 1);

		if (arguments[0] instanceof IntObjectInspector ||
				arguments[0] instanceof StringObjectInspector) {
			return arguments[0];
		} else {
			throw new RuntimeException("Not support argument: " + arguments[0]);
		}
	}

	@Override
	public Object evaluate(DeferredObject[] arguments) throws HiveException {
		return arguments[0].get();
	}

	@Override
	public String getDisplayString(String[] children) {
		return "TestHiveGenericUDF";
	}
}

/**
 * Test split udtf. Registered under name 'mygenericudtf'
 */
public class TestHiveUDTF extends GenericUDTF {

	@Override
	public StructObjectInspector initialize(ObjectInspector[] argOIs) throws UDFArgumentException {
		checkArgument(argOIs.length == 2);

		// TEST for constant arguments
		checkArgument(argOIs[1] instanceof ConstantObjectInspector);
		Object constant = ((ConstantObjectInspector) argOIs[1]).getWritableConstantValue();
		checkArgument(constant instanceof IntWritable);
		checkArgument(((IntWritable) constant).get() == 1);

		return ObjectInspectorFactory.getStandardStructObjectInspector(
			Collections.singletonList("col1"),
			Collections.singletonList(PrimitiveObjectInspectorFactory.javaStringObjectInspector));
	}

	@Override
	public void process(Object[] args) throws HiveException {
		String str = (String) args[0];
		for (String s : str.split(",")) {
			forward(s);
			forward(s);
		}
	}

	@Override
	public void close() {
	}
}
```

从Hive CLI，我们可以看到它们已注册：

```sql
hive> show functions;
OK
......
mygenericudf
myudf
myudtf
```

然后，用户可以在SQL中使用：

```sql
Flink SQL> select mygenericudf(myudf(name), 1) as a, mygenericudf(myudf(age), 1) as b, s from mysourcetable, lateral table(myudtf(name, 1)) as T(s);
```

### 限制

Flink目前不支持Hive内置函数。要使用Hive内置函数，用户必须首先在Hive Metastore中手动注册它们。

仅在Blink规划器中对Flink批处理测试了对Hive功能的支持。

Hive功能目前不能在Flink的目录中使用。

有关数据类型限制，请参阅[Hive](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/hive/index.html)。


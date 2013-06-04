# Cloudera Development Kit - Examples Module

The Examples Module is a collection of examples for the CDK.

## Example - User Dataset

This example shows basic usage of the CDK Data API for performing streaming writes
to (and reads from) a dataset.

From the examples module, build with:

```bash
mvn compile
```

Then create the dataset with:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.CreateUserDatasetPojo"
```

You can look at the files that were created with:

```bash
hadoop fs -ls -R /tmp/data/users
```

Note: The above assumes that you are running against a single-node localhost HDFS installation.
If this is not the case, then you can change `fs.default.name` in
`src/main/resources/core-site.xml`, e.g. to `file:///` to use the local filesystem.
Alternatively, you can pass in extra arguments to the command, as follows:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.CreateUserDatasetPojo" \
  -Dexec.args="-fs file:///"
```

For the rest of the examples we will assume a single-node localhost HDFS installation.

Once we have created a dataset and written some data to it, the next thing to do is to
read it back. We can do this with the `ReadUserDatasetPojo` program. 

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadUserDatasetPojo"
```

Finally, drop the dataset:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.DropUserDataset"
```

### Generic records vs. POJOs

The previous examples used POJOs, since they are the most familiar data transfer
objects for most Java programmers. Avro supports generic records too,
which are more efficient, since they don't require reflection,
and also don't require either the reader or writer to have the POJO class available.

Run the following to use the generic writer and reader:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.CreateUserDatasetGeneric"
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadUserDatasetGeneric"
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.DropUserDataset"
```

### Partitioning

The API supports partitioning, so that records are written to different partition files
according to the value of particular partition fields.

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.CreateUserDatasetGenericPartitioned"
hadoop fs -ls -R /tmp/data/users # see how partitioning affects the data layout
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadUserDatasetGeneric"
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadUserDatasetGenericOnePartition"
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.DropUserDataset"
```

### Parquet Columnar Format

Parquet is a new columnar format for data. Columnar formats provide performance
advantages over row-oriented formats like Avro data files (which is the default in CDK),
when the number of columns is large (typically dozens) and the typical queries that you perform
over the data only retrieve a small number of the columns.

Note that Parquet support is still experimental in this release.

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.CreateUserDatasetGenericParquet"
hadoop fs -ls -R /tmp/data/users # see the parquet file extension
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadUserDatasetGeneric"
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.DropUserDataset"
```

### HCatalog

So far all metadata has been stored in the _.metadata_ directory on the filesystem.
It's possible to store metadata in HCatalog so that other HCatalog-aware applications
like Hive can make use of it.

Run the following to create the dataset:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.CreateHCatalogUserDatasetGeneric"
```

Hive/HCatalog's metastore directory is set to _/tmp/user/hive/warehouse/_ (see
_src/main/resources/hive-site.xml_), which is where the data is written to:

```bash
hadoop fs -ls -R /tmp/user/hive/warehouse/
```

Notice that there is no metadata stored there, since the metadata is stored in
Hive/HCatalog's metastore:

```bash
hive -e 'describe users'
```

You can use Hive to query the data directly:

```bash
hive -e 'select * from users'
```

Alternatively, you can use the Java API to read the data:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadHCatalogUserDatasetGeneric"
```

Dropping the dataset deletes the metadata from the metastore and the data from the
filesystem:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.DropHCatalogUserDataset"
```

## Scala

Run the equivalent example with:

```bash
scala -cp "$(mvn dependency:build-classpath | grep -v '^\[')" src/main/scala/createpojo.scala
```

Or for the generic example:

```bash
scala -cp "$(mvn dependency:build-classpath | grep -v '^\[')" src/main/scala/creategeneric.scala
```

The Java examples can be used to read (and drop) the dataset written from Scala:

```bash
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.ReadUserDatasetGeneric"
mvn exec:java -Dexec.mainClass="com.cloudera.cdk.examples.data.DropUserDataset"
```

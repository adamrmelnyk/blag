+++
title = "Reading Parquet Files With Java"
date = 2018-04-21T12:41:54-05:00
description = "How to read Parquet Files in Java without Spark"
draft = false
toc = false
categories = ["technology"]
tags = ["Java", "Parquet"]
images = [] # overrides the site-wide open graph image
+++

A simple way of reading Parquet files without the need to use Spark.

I recently ran into an issue where I needed to read from [Parquet](https://parquet.apache.org/) files in a simple way without having to use the entire Spark framework. Though inspecting the contents of a Parquet file turns out to be pretty simple using the `spark-shell`, doing so without the framework ended up being more difficult because of a lack of documentation about how to read the actual content of Parquet files, the columnar format used by Hadoop and Spark.

<!--more-->

If all you need to do is inspect the contents of a parquet file you can do so pretty easily if you already have spark set up like so
{{< highlight shell>}}

$ spark-shell
scala> val sqlContext = new org.apache.spark.sql.SQLContext(sc)
scala> val parqfile = sqlContext.read.parquet("Objects.parquet")
scala> Parqfile.registerTempTable("object")
scala> val allrecords = sqlContext.sql("SELECT * FROM object")
scala> allrecords.show()
{{< / highlight >}}

However if that's not an option or you need a more programmatic way to inspect the elements you'll have to use the [libraries internally used by Hadoop and Spark](https://github.com/apache/parquet-mr).

If you're using Maven you'll want to get the following dependancies
{{< highlight xml "linenos=table,linenostart=1" >}}
<dependencies>
    <dependency>
        <groupId>org.apache.parquet</groupId>
        <artifactId>parquet-avro</artifactId>
        <version>1.10.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
{{< / highlight >}}

{{< highlight java "linenos=table,linenostart=1" >}}
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.parquet.column.page.PageReadStore;
import org.apache.parquet.example.data.simple.SimpleGroup;
import org.apache.parquet.example.data.simple.convert.GroupRecordConverter;
import org.apache.parquet.hadoop.ParquetFileReader;
import org.apache.parquet.hadoop.util.HadoopInputFile;
import org.apache.parquet.io.ColumnIOFactory;
import org.apache.parquet.io.MessageColumnIO;
import org.apache.parquet.io.RecordReader;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.Type;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class ParquetReaderUtils {

    public static Parquet getParquetData(String filePath) throws IOException {
        List<SimpleGroup> simpleGroups = new ArrayList<>();
        ParquetFileReader reader = ParquetFileReader.open(HadoopInputFile.fromPath(new Path(filePath), new Configuration()));
        MessageType schema = reader.getFooter().getFileMetaData().getSchema();
        List<Type> fields = schema.getFields();
        PageReadStore pages;
        while ((pages = reader.readNextRowGroup()) != null) {
            long rows = pages.getRowCount();
            MessageColumnIO columnIO = new ColumnIOFactory().getColumnIO(schema);
            RecordReader recordReader = columnIO.getRecordReader(pages, new GroupRecordConverter(schema));

            for (int i = 0; i < rows; i++) {
                SimpleGroup simpleGroup = (SimpleGroup) recordReader.read();
                simpleGroups.add(simpleGroup);
            }
        }
        reader.close();
        return new Parquet(simpleGroups, fields);
    }
}
{{< / highlight >}}

And to make things easier A Parquet data object
{{< highlight java "linenos=table,linenostart=1" >}}
import org.apache.parquet.example.data.simple.SimpleGroup;
import org.apache.parquet.schema.Type;

import java.util.List;

public class Parquet {
    private List<SimpleGroup> data;
    private List<Type> schema;

    public Parquet(List<SimpleGroup> data, List<Type> schema) {
        this.data = data;
        this.schema = schema;
    }

    public List<SimpleGroup> getData() {
        return data;
    }

    public List<Type> getSchema() {
        return schema;
    }
}
{{< / highlight >}}


Once we return the Parquet Object we can read whats inside each of the SimpleGroups like so:

{{< highlight java "linenos=table,linenostart=1" >}}
  Parquet parquet = ParquetReaderUtils.getParquetData();
  SimpleGroup simpleGroup = parquet.getData().get(0)
  String storedString = simpleGroups.get(0).getString("theFieldIWant", 0);
{{< / highlight >}}


The only caveat here being, we must determine the name of the field as well as the type in order to extract it. The fields we already have from the schema we stored on the Parquet object, however the type we may want to determine by manually inspecting the data with a debugger, though it can be determined dynamically using

{{< highlight java "linenos=table,linenostart=1" >}}
  parquet.getSchema().get(0).getOriginalType();
  parquet.getSchema().get(0).asPrimitiveType();
{{< / highlight >}}
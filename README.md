## crunch-quickstart

### Introduction

* Open source implementation of Flume Java
* It implements the jobs as pipelines
* Transformations through fn (dofn, mapfn, filterfn)

### Pipeline

* DAG of program flow.
* Supports lazy execution.
* Supports memory mode, MR mode & Spark mode.
* A pipeline object is used to populate crunch collections & write to disk.
* There are 3 pipeline impl as shown below.

Examples:
MRPipeline: Used for running MR jobs.
```
Pipeline pipeline = new MRPipeline(WordCountPipeline.class, getConf());
```

SparkPipeline: Used for running in Spark instead of MR.
```
Pipeline pipeline = new SparkPipeline(sparkContext, "spark word count example");
```

MemPipeline: Used for testing purposes.
```
Pipeline pipeline = MemPipeline.getInstance();
```

### Source

* Creates collections for processing and atleast 1 source required for pipeline.
* Reads inputs from various sources.
* Wraps Hadoop InputFormat to read files.

Examples:

Read from Text File HDFS:

```
PCollection<String> lines = pipeline.read(new TextFileSource(new Path("/dev/raw/text/"), Writables.strings()));
```
(or)
```
PType<String> stringType = Writables.strings()
PCollection<String> lines = pipeline.read(new TextFileSource(new Path("/dev/raw/text/"), stringType));
```

The above line is telling to read text file using Writables.strings() serialization
which is nothing but PType which we see later.

Read from Text File HDFS:
```
pipeline.readTextFile(inputPath);
```

Read from Avro File
```
pipeline.read(From.avroFile(order_items_path));
```

### PTypes & PTableType

* We use PTypes to tell crunch how to deserialize data while populating PCollection & how to serialize data while writing to disk from a PCollection.

```
PType<String> stringType = Writables.strings()
PCollection<String> lines = pipeline.read(new TextFileSource(new Path("/dev/raw/text/"), stringType));
```

In the above line we are telling crunch to read text file data in disk as Strings in memory.

* PTableType will return a PTable (key,value) instead of PCollection.

### Targets

* Targets wrap OutputFormat for writing data to disk.

```
pipeline.write(records,To.sequenceFile(outputPath));
```
The above line writes records in Pcollection as sequilefile into outputPath specified.

### Collections

#### PCollection<T>
* A representation of an immutable, unsorted, distributed collection of elements of type T.
* Distributed mean they are stored in multiple nodes in the cluster on Disk.

```
PCollection<String> lines = pipeline.readTextFile(inputPath);
```
* A PCollection of strings is formed by reading TextFile from the inputPath.
* PCollection object contains parallelDo method, to which we can pass a DoFn which does the required
logic as below. (Tokenizer if DoFn)

```
PCollection<String> words = lines.parallelDo(new Tokenizer(), Writables.strings());
```
* Element by element is passed to DoFN (Tokenizer) from PCollection.

#### PTable<K,V>

* A sub-interface of PCollection that represents an immutable, distributed, unordered multi-map of keys and values.
* It can have duplicate keys too as its multi map.

```
PTable<String, Long> counts = words.count();
```

#### PGroupedTable<K, V>

* Distributed, sorted map of keys (K) to an Iterable of values (V).
* A PGroupedTable is created by calling PTable#groupByKey() which triggers the sort and shuffle phase of a map-reduce job.

### code samples

* [word count](https://github.com/needforcode/bigdata-essentials/blob/master/crunch-quickstart/src/main/java/com/needforcode/crunch/pipeline/WordCountPipeline.java)
* [avro-pipeline](https://github.com/needforcode/bigdata-essentials/blob/master/crunch-quickstart/src/main/java/com/needforcode/crunch/pipeline/AvroPipeline.java)
* [parquet-pipeline](https://github.com/needforcode/bigdata-essentials/blob/master/crunch-quickstart/src/main/java/com/needforcode/crunch/pipeline/ParquetPipeline.java)
* [text-to-sequence](https://github.com/needforcode/bigdata-essentials/blob/master/crunch-quickstart/src/main/java/com/needforcode/crunch/pipeline/TextToSeqPipeline.java)
* [spark with crunch](https://github.com/needforcode/bigdata-essentials/blob/master/crunch-quickstart/src/main/java/com/needforcode/crunch/pipeline/WordCountSparkPipeline.java)
* [how to write mapfn](https://github.com/needforcode/bigdata-essentials/tree/master/crunch-quickstart/src/main/java/com/needforcode/crunch/mapfn)
* [how to write filterfn](https://github.com/needforcode/bigdata-essentials/tree/master/crunch-quickstart/src/main/java/com/needforcode/crunch/filterfn)
* [how to write dofn](https://github.com/needforcode/bigdata-essentials/tree/master/crunch-quickstart/src/main/java/com/needforcode/crunch/dofn)
* [crunch in scala](https://github.com/needforcode/bigdata-essentials/tree/master/crunch-quickstart/src/main/scala/com/needforcode/scrunch/pipeline)

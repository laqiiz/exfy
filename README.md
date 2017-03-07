# exfy
Lightweight ETL Libray.


# Work In Progress

This repository is work in progress.
Current status is api design.


# API Design Memo

## Quick Start :)

```
// File Copy
Pipeline.from("from_flle_path")
	.to("to_file_path")
	;
```

```
// List
List<List<String>> lists = PipeLine.from("from_file_path")
	.collect()
	;
```

```
// Stream
Stream<ExampleBrean> stream = PipeLine.from("from_file_path")
	.stream(ExampleBrean.class)
	;
```


## I/O
```
// Enable to set options
Pipeline.from("from_flle_path", Charsets.forName("Windows-31J"))
	to("to_file_path", StandardCharset.UTF_8)
	;
```

```
// Custom Reader & Writer
Pipeline.from(new CustomFixedWidthReader("from_file_path"))
	.to(new CustomKvsDataStoreWriter("url", "to_table_name"))
	;
```

## Filter

```
// Filter
Pipeline.from("from_file_path")
	.filter(list -> list.size() == 5)
	.filterByField(1, s -> "1".equals(s))
	.to("table_name")
	;
```


## Transformer
```
// Field Transformer
Pipeline.from("from_file_path")
	.transform(i -> i * 10, 1, 0, 1, 1, 0)	// TODO 無理なんじゃね疑惑。
	.transform(String::toUpperCase, 1)
	.transform(String::toUpperCase, 1, 2, 4)
	.transform(s -> s.substring(0, 10), 1, 6, 7)
	.transform(s -> lpad(s, 10, " "), 1)
	.to("to_file_path")	// when stream item is only one then write as it is.
	;
```

```
// Custom Transformer
Pipeline.from("from_file_path")
	// Field level transformer
	.transformField(3, new CustomFieldTransformer() {
		public String convertFiled(String field) {
			// do transform
			return field;
		}
	})
	// List level transformer
	.transform(new CustomListTansformer() {
		List<String> convert(List<String> list, int lineNo) {
			// do transform
			return list;
		}
	})
	.to("to_file_path")
	;
```

```
// Schema Based Transformer
Pipeline.from("from_file_path")
	.transform(new ShemaBasedValidator(
		new YamlFileConfiguration("configuration_path.yml")))
	.to("to_file_path")
	;
```

## Validator

```
// Validator
Pipeline.from("from_file_path")
	.validator(1, s.length <= 5)
	.validator(1, IntegerType.class)
	.validator(1, new Range(0, 10000))
	.validator(1, new NGList(500, 600, 700, 900))
	.error("error_file_path")
	.to("table_name")
	;
```

```
// Custom Validation & Error Handler
Pipeline.from("from_file_path")
	// Field level validator
	.validate(1, new CustomItemValidator() {
		public void validate(Context context, String field, int lineNo) {
			// do validation
		}
	})
	// List level validator
	.validate(new CustomListValidator() {
		public void validate(Context context, List<String> list, int lineNo) {
			// do validation
		}
	})
	.uniq("key2", "key2", true)
	.errorHadnler(DefalutErrorHandler.class, "option_1")	// toの前に無いとダメ
	.to(CustomCsvWriter.class, "option_1", "option_2")
	;
```

```
// Schema Based Validator
Pipeline.from("from_file_path")
	.validator(new ShemaBasedValidator(
		new YamlFileConfiguration("configuration_path.yml")))
	.error("error_file_path")
	.to("to_file_path")
	;
```

```
// Validator Set
CompositeValitor standardCompositeValidator = new CompositeValidator()
	.validate(new DefaultTypeValidaor())
	.validate(new LengthValidaor())
	.validate(new RequiredValidator())
	;
Pipeline.from("from_file_path")
	.validate(compositeValidator)
	.transform(/* 省略 */)
	.to("to_file_path")
	;
```

## Aggregator

```
// Aggregate
Pipeline.from("from_file_path")
	.aggregateKey("key_1", new Aggregator() {
		public List<String> aggregate(List<List<String>> lines) {
			// do aggregation
		}
	})
	.aggregateCount(10, new Aggregator() {
		public List<String> aggregate(List<List<String>> lines) {
			// do aggregation
		}
	})
	;
```

```
// Custom Aggregation Trigger
Pipeline.from("from_file_path")
	.aggregationTrigger(new CustomAggregationTriger())
	.aggregate(new Aggregator() {
		public List<String> aggregate(List<List<String>> lines) {
			// do aggregation
		}
	})
	.to("to_file_path")
	;
```

## Splitter
```
// Split
Pipeline.from("from_file_path")
	.splitter(new CustomSplitter() {
		public List<List<String>> split(List<String> list) {
			// do Split
		}
	})
	.to("to_file_path")
	;
```



## Label(Key) Binding
```
Pipeline.from("from_file_path")
	.userHeader(true)
	.to("to_file_path")	// when stream item is only one then write as it is.
	;
```

```
Pipeline.from("from_file_path")
	.bind("col1", "col2", "col3", "col4", "col5)
	.transform(String::toUpperCase, "col2", "col4", "col5")
	.transform(s -> s.substring(0, 10), "col1", "col3", "col6")
	.to("to_file_path")	// when stream item is only one then write as it is.
	;
```

```
// Multi Layout
Pipeline.from("from_file_path")
	.tranform(/**/)
	.when(/*Predicate*/)
		.transform(/*Layout1 Transformer*/).to(this::pipe1) // method reference
	.when(/*Predicate*/)
		.transform(/*Layout2 Transformer*/).to(this::pipe2)
	.otherwise()
		.transform(/*Layout3 Transformer*/).to(this::pipe3)
	.end() // MUST BE SETTED!
	;

// [Work In Progess]
private void pipe1(Pipeline pipeline) {
	pipeline.transform(/**/).to(/**/);
}
```


## Error Output

```
// CAUTION!
List<List<String<> lists = PipeLine.from("from_file_path")
	.validate(/*Validator*/)
	.collect() // drop validation error stream.
	;
```

```
// if you want to use error, then code is like this.
PipeLine pipeline = PipeLine.from("from_file_path")
	.validate(/*Validator*/)
	.transform(/*Transform*/)
	;
List<List<String>> list = pipeLine.collect();
List<Error> errList = pipeLine.error();
```

## Now Working

```
// Error
Error err = ...
long lineNo = err.getLineNo();
long fieldNo = err.getFieldNo(); //TODO index or no(number)
Status status err.getStatus();
String msg = err.getValidationMessage();
String history = err.getHistory();	// TODO concern performance degration
```




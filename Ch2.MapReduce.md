# MapReduce

## MapReduce

맵리듀스는 데이터 처리를 위한 프로그래밍 모델이다.

하둡은 자바, 루비, 파이썬 등 다양한 언어로 맵리듀스를 만들고, 실행할 수 있다.

맵 리듀스는 처음부터 병행성을 위해 설계되었기 때문에 누구나 쉽게(?) 빅 데이터 분석을 할 수 있다.

<br>

## Table of Contents

[Weather Data Set](#Weather-Data-Set)

[Analyzing the Data with Unix Tools](#Analyzing-the-Data-with-Unix-Tools)

[Analyzing the Data with Hadoop](#Analyzing-the-Data-with-Hadoop)

[Scailing Out](#Scailing-Out)

<br>

## Weather Data Set

간단한 예제로 기상 데이터 셋을 활용하여 연도에 따른 기온 데이터를 입력받아, 연도별 최고 기온을 찾는 예제를 진행해보자.

하나의 gzip 파일에는 연도에 따른 기상관측소가 측정한 정보들이 들어있다.

gzip의 각 행에는 기상관측소 식별자를 시작으로, 관측 날짜, 관측 시간, 바람 방향, 구름 고도 등의 다양한 정보가 (구분자 없이) 나열되어 있다.

<br>

## Analyzing the Data with Unix Tools

하둡과의 비교를 위해 처음에는 awk로 데이터를 분석해보자.

awk는 행 기반의 데이터 처리를 위한 유닉스 툴이다.

책에 따르면, 20세기 전체 데이터를 한 대의 EC2 인스턴스에서 처리하는데 42분이 걸렸다고 한다.

<br>

프로세스를 병렬로 수행한다면 처리 속도가 빨라질 것이다.

한 대의 머신이라도 병렬로 처리한다면 속도가 빨라질 수 있지만, 다음과 같은 문제가 발생한다.

1. 일을 똑같은 크기로 나누는 것은 쉽지 않다

   - 연도별로 데이터를 나눈다면, 데이터가 가장 큰 연도를 맡은 프로세스에 의해 전체 시간이 결정된다

     - 대안으로 전체 데이터를 똑같은 크기로 나눠 실행하는 방법이 있다

<br>

2. 병렬로 처리한 결과를 다시 합치는 데 더 많은 처리가 필요할 수도 있다

   - 앞에 말한 것처럼, 같은 연도를 나눠서 처리할 경우, 결과값을 재병합, 처리하는 과정이 필요하다

<br>

3. 아무리 병렬로 처리한다고 한들, 하나의 머신에서 처리하는 데는 한계가 있다

<br>

## Analyzing the Data with Hadoop

하둡이 제공하는 병렬 처리의 이점을 제대로 활용하기 위해서는 맵리듀스로 처리하면 된다.

<br>

### Map and Reduce

맵리듀스는 맵과 리듀스, 2단계로 구분된다.

각 단계의 입출력으로는 Key-Value(키-값)를 갖는데, 변수형은 프로그래머가 선택할 수 있다.

뿐만 아니라 프로그래머는 맵 함수와 리듀스 함수를 각각 작성해야 한다.

<br>

#### Map

이 예제에서 맵은 원본 데이터를 입력으로 받는다.

맵 입력 데이터의 Key는 데이터의 오프셋으로, 무시한다.

맵 입력 데이터의 Value는 원본 데이터다.

맵은 입력 데이터의 Value에서 연도와 기온을 추출하여 출력데이터로 출력한다.

<br>

#### MapReduce Framework

맵리듀스 프레임워크에 의해 맵의 출력값은 리듀스의 입력값으로 보내진다.

이 때, Key-Value는 Key를 기준으로 정렬되고 그룹화된다.

이 예제의 경우, 예를 들어, (1972, 213)과 (1972, 175)는 (1972, [213, 175])로 합쳐진다.

<br>

#### Reduce

리듀스는 입력값으로 연도별 기온들의 묶음을 받는다.

리듀스는 입력값을 돌면서 연도별 최고기온을 찾아 출력한다.

<br>

### JAVA MapReduce

그렇다면 JAVA로 구현한 MapReduce를 살펴보자.

<br>

#### JAVA Mapper

```JAVA
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MaxTemperatureMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
   private static final int MISSING = 9999;

   @Override
   public void map(LongWritable key, Text value, Context context) throws IOExcepion, InterruptedException {
      String line = value.toString();
      String year = line.substring(15, 19);
      int airTemperature;

      if(line.charAt(87) =='+') {
         airTemperature = Integer.parseInt(line.substring(88, 92));
      } else {
         airTemperature = Integer.parseInt(line.substring(87, 92));
      }

      String quality = line.substring(92, 93);

      if(airTemperature != MISSING && quality.matches("[01459]")) {
         context.write(new Text(year), new IntWritable(airTemperature));
      }
   }
}
```

위의 코드는 이 예제의 Mapper를 JAVA로 구현한 코드로, 원본은 [이곳](https://github.com/tomwhite/hadoop-book/blob/master/ch02-mr-intro/src/main/java/MaxTemperatureMapper.java)에서 확인할 수 있다.

1. 이 코드는 추상 메서드 map을 정의하는 Mapper 클래스를 상속받는다

2. Mapper클래스는 입력키와 입력값, 출력키와 출력값 네 가지의 매개변수를 갖는다

3. map 메서드에서 입력키로 key를, 입력값으로 value를 받는다

4. value를 String형으로 변환한 뒤, substring을 통해 연도와 기온, 퀄리티를 추출한다

5. 추출한 데이터의 적합성을 검사한 뒤, 출력값인 context에 적절히 대입한다

<br>

#### JAVA Reducer

```JAVA
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class MaxTemperatureReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

   @Override
   public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
      int maxValue = Integer.MIN_VALUE;

      for(IntWritable value : values) {
         maxValue = Math.max(maxValue, value.get());
      }

      context.wrtie(key, new IntWritable(maxValue));
   }
}
```

위의 코드는 이 예제의 Mapper를 JAVA로 구현한 코드로, 원본은 [이곳](https://github.com/tomwhite/hadoop-book/blob/master/ch02-mr-intro/src/main/java/MaxTemperatureReducer.java)에서 확인할 수 있다.

1. 이 코드는 추상 메서드 reduce를 정의하는 Reducer 클래스를 상속받는다

2. Reducer클래스는 입력키와 입력값, 출력키와 출력값 네 가지의 매개변수를 갖는다

3. reduce 메서드에서 입력키로 key를, 입력값으로 value를 받는다

4. 맵과 리듀스 사이의 맵리듀스 프레임워크에 의해 입력키로 연도를, 입력값으로 연도별 기온들을 받는다

5. 입력값들 중 최고기온을 찾아 연도와 함께 출력값인 context에 적절히 대입한다

<br>

#### JAVA Main

```JAVA
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.iput.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MaxTemperature {
   public static void main(String[] args) throws Exception {
      if(args.length != 2) {
         System.err.println("Usage: MaxTemperature <input path> <output path>");
         System.exit(-1);
      }

      Job job = new Job();
      job.setJarByClass(MaxTemperature.class);
      job.setJobName("Max temperature");

      FileInputFormat.addInputPath(job, new Path(args[0]));
      FileOutputFormat.setOutputPath(job, new Path(args[1]));

      job.setMapperClass(MaxTemperatureMapper.class);
      job.setReducerClass(MaxTemperatureMapper.class);

      job.setOutputKeyClass(Text.class);
      job.setOutputValueClass(IntWritable.class);

      System.exit(job.waitForCompletion(true) ? 0 : 1);
   }
}
```

1. Job 객체는 잡 명세서를 작성한다

2. 하둡 클러스터에서는 잡을 실행하기 전에 코드를 JAR 파일로 묶어야 한다

   - setJarByClass 메서드를 사용하면 하둡은 해당 클래스를 포함한 관련 JAR 파일을 찾아서 클러스터에 배치해준다

   - 혹은 JAR 파일의 이름으로 지정하는 방법도 있다

3. FileInputFormat의 addInputPath 메서드로 job의 입력 경로를 지정한다

4. FileOutputFormat의 setOutputPath 메서드로 job의 출력 경로를 지정한다

   - 이 때, 출력 경로로 지정한 디렉토리는 job을 수행하는 시점에 존재하지 않아야 한다

   - job을 수행하는 시점에 해당 디렉토리가 존재하는 경우 에러가 발생한다

5. setMapperClass, setReducerClass 메서드로 맵과 리듀스를 지정한다

6. waitForCompletion 메서드로 job이 끝날 때까지 기다린 뒤, 무사히 마쳤다면 본 시스템을 정상 종료한다

<br>

## Scailing Out

위 예제는 쉬운 진행을 위해 로컬 파일시스템 환경에서 데이터를 처리했다. (그만큼 크기도 작았다)

프로젝트를 확장하려면 전체 데이터를 키워야하며 이를 관리하기 위해서는 HDFS(Hadoop Distributed File System)라는 분산 파일시스템에 데이터를 저장해야 한다.

하둡은 데이터의 일부분이 저장된 클러스터의 각 머신에서 매리듀스 프로그램을 실행한다.

그리고 이를 위해 하둡은 YARN이라는 하둡 자원 관리 시스템을 이용한다.

<br>

### Data Flow


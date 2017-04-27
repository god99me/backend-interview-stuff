## Big Data

### 2. MapReduce Project - AutoComplete  

#### Docker
Docker 需要利用本机 OS，它不包括操作系统，这是与虚拟机的区别。  
Hadoop, Kafka 理解为一个 Docker 的容器而不是容器中安装的 App。

`docker ps`  列出正在运行的 docker
`docker ps -a` 列出所有 docker
`docker ps -aq` 列出所有 docker id

`docker run [parm] image_name exec-command` 基于某个镜像，指定几个特定参数来运行容器，并在之后执行 exec_command 所指代的命令

- -i 保持容器对容器的 stdin 为打开状态（管道）
- -t 为容器分配一个虚拟终端，启动容器后自动进入
- -d 让容器在后台运行
- -name 指定容器别名
- -rm 容器推出后自动删除

`docker rm` 删除容器
`docker rm container_id` 或简写 `container_id` 前几位  

`docker stop container_id`  
`docker start container_id`    

`docker attach` 少用，退出终端会导致容器也退出  
`docker exec` 在容器内执行命令，并在退出时保持后台运行

`docker cp container_id:some_file target_place` 把 docker 容器里的文件复制到本机（只允许单向）  

`docker images` 镜像
`docker rmi image_id` 

Dockerfile 书写
from  
run  程序构建时进行  
add  
cmd  程序运行时进行  
expose  绑定 docker 端口与本机端口   

`docker build`  执行脚本过程中会检查是否能利用到缓存  

`hub.docker.com`

#### Introduction

`应用场景`
1. 搜索建议
2. 拼写纠正

#### N-Gram Model  
`N-Gram`: 一句话或一段话中**连续的** n 个单词（或字母）  
`Model`: 用概率分布解决问题，预测接下来会出现的单词  

- Predict N-Gram based on 1-Gram
- Predict N-Gram based on N-Gram

`Workflow`  

1. Process raw data
2. Outline process

`数据库表结构`  

1. 输入单词
2. 联想单词
3. 概率

`jobs of map-reduce` 

1. MapReduce job 1  
读入原始数据  
统计 n 个连续出现的单词 出现的次数

2. MapReduce job 2  
计算 m 个单词所能联想到单词的概率   
写数据库

#### Steps  
1. Read a large-scale document collections  
2. Build n-gram library(word count is 1 gram library)  
3. Calculate probability  
4. Run the project on MapReduce  

Document preprocessing  

- Read each document  
 	- line by line   
	- sentence by sentence   
- Remove all non-alphabetical symbols  

```
// Mapper  
// Split to 2 - N words group

// Reducer
// Merge  
public class NGramLibraryBuilder {
  public static class NGramMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
    public void setup(Context context) {
      Configuration conf = context.getConfiguration();
      int nGram = conf.getInt("nGram", 5);
		// 执行且只执行一次
    }
    
    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
    	// Input: read sentence
    	// I love data n=3
    	// I love 1
    	// love data 1
    	// I love data
    	    	
    	String line = value.toString().trim().toLowerCase().replaceAll("[^a-z]", " ");
    	String[] words = line.split("\\s+");
    	if (words.length < 2) {
    		return;
    	}
    	
    	StringBuilder sb;
    	for (int i = 0; i < words.length; i++) {
    		sb = new StringBuilder();
    		sb.append(words[i]);
    		for (int j = 1; i + j < words.length && j < nGram; j++) {
    			sb.append(" ");
    			sb.append(words[i + j]);
    			context.write(new Text(sb.toString()), new IntWritable(1));
    		}
    	}
    }
  }
  
  public static class NGramReducer extends Reducer<Text, IntWritable,  Text, IntWritable>{
    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
    	int sum = 0;
    	for (IntWritable value : values) {
    	  sum += value.get();
    	}
    	context.write(key, new IntWritable(sum))
    }
  }
}

```

#### Frequently used class in MR
- IntWritable // 包装类，考虑到了串行化，而 int 不支持读写等
- Text String // 包装类，可以串行化读写
- Configuration // just like properties in java
- Job
- Context // 交互媒介
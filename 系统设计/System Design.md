## System Design  

### 定义  
为满足需求而定义结构、组件、模块、接口和数据的过程  

#### 方法论  
自顶向下 自底向上  

#### 面试问题
##### 基础
1. 设计系统
2. 评估每秒访问量
3. 扩展系统  

##### 进阶
1. 设计Netflix/YouTube/Spotify
2. 设计基于数据的类和继承
3. 设计用户系统
4. 设计支付系统
5. 在浏览器中输入地址后发生的事情
6. 如何优化网页性能
7. 如何设计秒杀系统
8. 写一个爬虫
9. GFS\BigTable\MapReduce
10. Find the top-10 frequent keywords/mean number with MapReduce
11. 线程安全的生产者消费者模型
12. 实现 google 搜索建议和补全功能
13. 设计 WhatsAPP （聊天功能）
14. 设计 Instagram/ Facebook/ Twitter (信息流)
15. 日志分析（离线／实时）
16. 设计负载均衡
17. TinyURL 短链接分享
18. 面向对象设计 singleton\elevator\parking lot\card game\ file system\hotel reservation
19. 成就系统（游戏公司）


### etc
大数据的作用：知识图谱、O2O 广告  
倒排索引 

#### 设计一个电台
- Scenario **枚举**场景、功能、排序各功能的重要程度
- Necessary 限制条件、访问量
	- 询问**日活**跃用户数
	- 评估**同时在线**用户数、峰值在线用户数、三个月后峰值用户数 
	- 评估流量、内存

- Application 服务、算法
	- 为每个请求创建一个 service	 
	- 合并 services
- Kilobit 数据
	- 为每个请求设计数据集
	- 选择存储引擎
- Evolve

中级：  
www.zhihu.com/question/20059632  
www.zhihu.com/question/23602133

#### 设计一个用户推荐模型
- 每个用户喜欢的音乐集合
- 相似度计算
- 对于每个用户，寻找最相似的几个用户

中级：
www.ibm.com/developerworks/cn/web/1103_zhaoct_recommstudy1
blog.csdn.net/hguisu/article/details/7962350
### 区别
- BeanFactory在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化；
- ApplicationContext在启动的时候就把所有的Bean全部实例化了。它还可以为Bean配置lazy-init=true来让Bean延迟实例化；
- 继承自 ApplicationContextPublisher 接口，拥有发布事件注册监听的能力

### 各自的优点（应用场景）

#### BeanFactory
应用启动的时候占用资源很少；对资源要求较高的应用，比较有优势； 
#### ApplicationContext 
1. 所有的Bean在启动的时候都加载，系统运行的速度快； 

2. 在启动的时候所有的Bean都加载了，我们就能在系统启动的时候，尽早的发现系统中的配置问题 

3. 建议web应用，在启动的时候就把所有的Bean都加载了。（把费时的操作放到系统启动中完成）

### ApplicationContext更多内容
ApplicationContext接口,它由BeanFactory接口派生而来，因而提供BeanFactory所有的功能。ApplicationContext以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能： 
- MessageSource, 提供国际化的消息访问  
- 资源访问，如URL和文件  
- 事件传播  
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层  
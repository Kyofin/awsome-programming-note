# 调度中心代码



# springboot执行器端代码

## 核心逻辑

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190428142908.png)

springboot执行器端启动由于`XxlJobConfig`会初始化bean `XxlJobSpringExecutor`。

初始化的时候会执行`XxlJobSpringExecutor`的父类的`start`方法。

```java
public void start() throws Exception {

        // init JobHandler Repository
        initJobHandlerRepository(applicationContext);

        // refresh GlueFactory
        GlueFactory.refreshInstance(1);


        // super start
        super.start();
    }
```

其中` initJobHandlerRepository(applicationContext)`会将所有标记@JobHandler注解的类初始化实例。并放到类**XxlJobExecutor**的属性`jobHandlerRepository`中维护起来。

```java
private void initJobHandlerRepository(ApplicationContext applicationContext){
        if (applicationContext == null) {
            return;
        }

        // init job handler action
        Map<String, Object> serviceBeanMap = applicationContext.getBeansWithAnnotation(JobHandler.class);

        if (serviceBeanMap!=null && serviceBeanMap.size()>0) {
            for (Object serviceBean : serviceBeanMap.values()) {
                if (serviceBean instanceof IJobHandler){
                    String name = serviceBean.getClass().getAnnotation(JobHandler.class).value();
                    IJobHandler handler = (IJobHandler) serviceBean;
                    if (loadJobHandler(name) != null) {
                        throw new RuntimeException("xxl-job jobhandler naming conflicts.");
                    }
                  // 此处registJobHandler
                    registJobHandler(name, handler);
                }
            }
        }
    }
```



## 核心类

### JobThread

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190428173508.png)

该线程类内部维护了我们自定义的标记了`@JobHandler`注解的类，并由共享变量`toStop`判断是否跳出while循环。

下面其是集成`Thread`实现的`run`方法

```java
 @Override
	public void run() {

    	// init
    	try {
			handler.init();
		} catch (Throwable e) {
    		logger.error(e.getMessage(), e);
		}

		// execute
		while(!toStop){
      //省略无数代码
    }
    
    
    // ========================== 无法到达除非 toStop 为true ===================

		// callback trigger request in queue
		while(triggerQueue !=null && triggerQueue.size()>0){
		}

		// destroy
		try {
			handler.destroy();
		} catch (Throwable e) {
			logger.error(e.getMessage(), e);
		}

		logger.info(">>>>>>>>>>> xxl-job JobThread stoped, hashCode:{}", Thread.currentThread());
	}
```





### ExecutorBizImpl

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190428173846.png)

#### 任务管理的执行会触发run方法。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190428174754.png)

run方法代码比较多，这里主要注意两个地方

##### 第一次执行任务时会将注册jobThread，传入的是自定义的jobhandler

```java
// replace thread (new or exists invalid)
        if (jobThread == null) {
            jobThread = XxlJobExecutor.registJobThread(triggerParam.getJobId(), jobHandler, removeOldReason);
        }
```

我们可以进入` XxlJobExecutor.registJobThread`看到XxlJobExecutor内部维护了一个属性来存放所有jobThread，并且注册进来的线程都立刻都start了，就会执行上面JobThread类的`run`方法

```java
  // ---------------------- job thread repository ----------------------
    private static ConcurrentHashMap<Integer, JobThread> jobThreadRepository = new ConcurrentHashMap<Integer, JobThread>();

    public static JobThread registJobThread(int jobId, IJobHandler handler, String removeOldReason){
        JobThread newJobThread = new JobThread(jobId, handler);
        newJobThread.start();
        logger.info(">>>>>>>>>>> xxl-job regist JobThread success, jobId:{}, handler:{}", new Object[]{jobId, handler});

        JobThread oldJobThread = jobThreadRepository.put(jobId, newJobThread);	// putIfAbsent | oh my god, map's put method return the old value!!!
        if (oldJobThread != null) {
            oldJobThread.toStop(removeOldReason);
            oldJobThread.interrupt();
        }

        return newJobThread;
    }
```

##### 第二次执行时会推入队列中，不会立刻执行。

```java
// push data to queue
 ReturnT<String> pushResult = jobThread.pushTriggerQueue(triggerParam);
```

因为jobthread内部有一个队列来维护要执行的任务。而jobthread的run方法会不断循环从队列中取出任务。

```
// to check toStop signal, we need cycle, so wo cannot use queue.take(), instand of poll(timeout)
				triggerParam = triggerQueue.poll(3L, TimeUnit.SECONDS);
```



#### 点击进去的调度日志中的执行日志对应log方法。

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190429142820.png)

主要是读取执行器端的本地日志文件返回。

```java
 @Override
    public ReturnT<LogResult> log(long logDateTim, int logId, int fromLineNum) {
        // log filename: logPath/yyyy-MM-dd/9999.log
        String logFileName = XxlJobFileAppender.makeLogFileName(new Date(logDateTim), logId);

        LogResult logResult = XxlJobFileAppender.readLog(logFileName, fromLineNum);
        return new ReturnT<LogResult>(logResult);
    }
```

#### 终止任务对应kill方法

```java
 @Override
    public ReturnT<String> kill(int jobId) {
        // kill handlerThread, and create new one
        JobThread jobThread = XxlJobExecutor.loadJobThread(jobId);
        if (jobThread != null) {
            XxlJobExecutor.removeJobThread(jobId, "scheduling center kill job.");
            return ReturnT.SUCCESS;
        }

        return new ReturnT<String>(ReturnT.SUCCESS_CODE, "job thread aleady killed.");
    }
```


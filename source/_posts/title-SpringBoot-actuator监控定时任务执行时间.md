title: SpringBoot actuator监控定时任务执行时间
date: 2020-03-05 21:11:42
tags: java,springboot,actuator,schedule
---
# SpringBoot actuator监控定时任务执行时间


为了能够可视化任务执行情况，我们需要记录相应的度量指标

## 初始化项目
我们通过[start.spring.io](https://start.spring.io/)来快速生成Spring项目，添加Spring Web和Spring Boot Actuator，点击生成
![](images/2020-03-05-15834124213131.jpeg)


我们来编写测试的定时任务，编写后启动，发现我们的job已正常执行

```java
package com.bohanzhang.actuatordemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

@SpringBootApplication
@EnableScheduling
public class ActuatordemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(ActuatordemoApplication.class, args);
	}

	/**
	 * 5秒执行一次
	 * @throws InterruptedException
	 */
	@Scheduled(fixedRate = 5000)
	public void jobDemo() throws InterruptedException {
		System.out.println("start jobDemo");
		Thread.sleep(3000);
		System.out.println("end jobDemo");
	}
}

```

日志如下：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.5.RELEASE)

2020-03-05 20:58:18.180  INFO 17766 --- [           main] c.b.a.ActuatordemoApplication            : Starting ActuatordemoApplication on zhangbohans-MacBook-Pro.local with PID 17766 (/Users/bohan/Downloads/actuatordemo/target/classes started by bohan in /Users/bohan/Downloads/actuatordemo)
2020-03-05 20:58:18.181  INFO 17766 --- [           main] c.b.a.ActuatordemoApplication            : No active profile set, falling back to default profiles: default
2020-03-05 20:58:18.794  INFO 17766 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-03-05 20:58:18.799  INFO 17766 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-03-05 20:58:18.799  INFO 17766 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.31]
2020-03-05 20:58:18.842  INFO 17766 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-03-05 20:58:18.842  INFO 17766 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 637 ms
2020-03-05 20:58:19.046  INFO 17766 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-03-05 20:58:19.191  INFO 17766 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2020-03-05 20:58:19.195  INFO 17766 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 13 endpoint(s) beneath base path '/actuator'
start jobDemo
2020-03-05 20:58:19.271  INFO 17766 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-03-05 20:58:19.274  INFO 17766 --- [           main] c.b.a.ActuatordemoApplication            : Started ActuatordemoApplication in 1.284 seconds (JVM running for 1.601)
end jobDemo
start jobDemo
end jobDemo

```


下面我们来为这个job添加监控，监控主要依赖`MeterRegistry`来进行

```java
package com.bohanzhang.actuatordemo;

import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

@SpringBootApplication
@EnableScheduling
public class ActuatordemoApplication {
	
	private MeterRegistry meterRegistry;

	public ActuatordemoApplication(MeterRegistry meterRegistry) {
		this.meterRegistry = meterRegistry;
	}

	public static void main(String[] args) {
		SpringApplication.run(ActuatordemoApplication.class, args);
	}

	/**
	 * 5秒执行一次
	 * @throws InterruptedException
	 */
	@Scheduled(fixedRate = 5000)
	public void jobDemo() throws InterruptedException {
		meterRegistry.timer("job.demo").record(() -> {
			System.out.println("start jobDemo");
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("end jobDemo");
		});
		
	}
}

```

记录后如果查看呢？我们来访问`/actuator`，默认只打开了health,info，

```shell
$ curl http://localhost:8080/actuator
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"info":{"href":"http://localhost:8080/actuator/info","templated":false}}}%
```

我们可以在通过修改`application.properties`来展示所有的功能

```conf
management.endpoints.web.exposure.include=*
```

再次访问`/actuator`

```shell
curl http://localhost:8080/actuator
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},"caches-cache":{"href":"http://localhost:8080/actuator/caches/{cache}","templated":true},"caches":{"href":"http://localhost:8080/actuator/caches","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"info":{"href":"http://localhost:8080/actuator/info","templated":false},"conditions":{"href":"http://localhost:8080/actuator/conditions","templated":false},"configprops":{"href":"http://localhost:8080/actuator/configprops","templated":false},"env":{"href":"http://localhost:8080/actuator/env","templated":false},"env-toMatch":{"href":"http://localhost:8080/actuator/env/{toMatch}","templated":true},"loggers-name":{"href":"http://localhost:8080/actuator/loggers/{name}","templated":true},"loggers":{"href":"http://localhost:8080/actuator/loggers","templated":false},"heapdump":{"href":"http://localhost:8080/actuator/heapdump","templated":false},"threaddump":{"href":"http://localhost:8080/actuator/threaddump","templated":false},"metrics-requiredMetricName":{"href":"http://localhost:8080/actuator/metrics/{requiredMetricName}","templated":true},"metrics":{"href":"http://localhost:8080/actuator/metrics","templated":false},"scheduledtasks":{"href":"http://localhost:8080/actuator/scheduledtasks","templated":false},"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false}}}%
```

想看我们的监控信息可以通过`/actuator/metrics/job.demo`来查看

```shell
$ curl http://localhost:8080/actuator/metrics/job.demo
{"name":"job.demo","description":null,"baseUnit":"seconds","measurements":[{"statistic":"COUNT","value":21.0},{"statistic":"TOTAL_TIME","value":63.06746522},{"statistic":"MAX","value":3.005089397}],"availableTags":[]}%
```

这里可以看到我们任务的执行次数、总用时及当前用时等






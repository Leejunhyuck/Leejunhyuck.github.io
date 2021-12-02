---
layout: post
title: Spring @Async 관련
date: 2021-11-17 11:30:23 +0900
category: JPA
---
# @Async 어노테이션 - 비동기 처리

결과 요청 API를 비동기식으로 변경하기 위하여 @Async 어노테이션을 사용하였다.

실제로 Java에서 가장 기본적인 방법으로 비동기를 구현하려면 Thread를 사용하여야 한다.

Spring에서는 어노테이션으로 비슷한 기능을 제공한다.

실제로 Thread를 생성하여 Run되기 때문에, 컨텍스트 흐름은 이어지고 난 뒤 Thread 생성 후 따로 실행된다.

응답결과를 먼저 주고 후 비동기식으로 처리가 가능하다.

1. method 레벨별 사용하기위한 Config 정의

```
@configuration
@EnableAsync
class SpringAsyncConfig {
    @bean(name = ["threadPoolTaskExecutor"])
    fun threadPoolTaskExecutor(): Executor {
        return ThreadPoolTaskExecutor()
    }
}

@Async("threadPoolTaskExecutor")
fun methName(){
}
```


2. application 레벨별 사용하기위한 Config 정의


```
@configuration
@EnableAsync
class SpringAsyncConfig : AsyncConfigurer {
    override fun getAsyncExecutor(): Executor? {
        return ThreadPoolTaskExecutor()
    }
}
```



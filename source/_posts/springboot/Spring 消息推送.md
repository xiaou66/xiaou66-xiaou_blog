---
title: Spring 消息推送
date: 2022/03/10 13:45:53
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# Spring 消息推送

常见的消息推送的技术有

1. SSE 单向推送
2. WebSocket 双向推送
3. HTTP 长连接

## SSE 单向推送

:::info

SseEmitter 是 SpringMVC(4.2+) 提供的一种技术 , 它是基于 Http 协议的，相比 WebSocket，它更轻量，但是它只能从服务端向客户端单向发送信息。在 SpringBoot 中我们无需引用其他 jar 就可以使用。

:::

### 后端代码

1. 请求开启 SSE

```java
@RequestMapping("/start")
public SseEmitter startSseEmitter(String clientId) {
    SseEmitter sseEmitter = new SseEmitter(0L);
    sseEmitterMap.put(clientId, new Result(clientId, System.currentTimeMillis(), sseEmitter));
    clearSseEmitterConnect();
    return sseEmitter;
}
```

2. 推送消息

```java
@RequestMapping("/send")
public String setSseEmitter(String clientId) {
    try {
        Result result = sseEmitterMap.get(clientId);
        if (Objects.nonNull(result) && result.sseEmitter != null) {
            long timestamp = System.currentTimeMillis();
            result.sseEmitter.send(timestamp);
        }
    }catch (IOException e) {
        log.warn("clientId: {} 出现问题", clientId);
        log.error(e);
        sseEmitterMap.remove(clientId);
        return "error";
    }
    return "succeed";
}
```

3. 结束连接请求

```java
@RequestMapping("/end")
public String completeSseEmitter(String clientId) {
    Result result = sseEmitterMap.get(clientId);
    if (Objects.nonNull(result)) {
        sseEmitterMap.remove(clientId);
        result.sseEmitter.complete();
    }
    return "succeed";
}
```

全部代码

```java
/**
 * 使用 SSE 推送消息
 */
@RestController
@Log4j2
public class SSEController {
    private static Map<String, Result> sseEmitterMap = new ConcurrentHashMap<>();
    @Autowired
    private ObjectMapper objectMapper;
    @RequestMapping("/start")
    public SseEmitter startSseEmitter(String clientId) {
        SseEmitter sseEmitter = new SseEmitter(0L);
        sseEmitterMap.put(clientId, 
                          new Result(clientId, System.currentTimeMillis(), sseEmitter));
        clearSseEmitterConnect();
        return sseEmitter;
    }
    @RequestMapping("/send")
    public String setSseEmitter(String clientId) {
        try {
            Result result = sseEmitterMap.get(clientId);
            if (Objects.nonNull(result) && result.sseEmitter != null) {
                long timestamp = System.currentTimeMillis();
                result.sseEmitter.send(timestamp);
            }
        }catch (IOException e) {
            log.warn("clientId: {} 出现问题", clientId);
            log.error(e);
            sseEmitterMap.remove(clientId);
            return "error";
        }
        return "succeed";
    }
    @RequestMapping("/end")
    public String completeSseEmitter(String clientId) {
        Result result = sseEmitterMap.get(clientId);
        if (Objects.nonNull(result)) {
            sseEmitterMap.remove(clientId);
            result.sseEmitter.complete();
        }
        return "succeed";
    }

    /**
     * 清理连接
     */
    private void clearSseEmitterConnect() {
        Iterator<Result> iterator = sseEmitterMap.values().iterator();
        while (iterator.hasNext()) {
            Result result = iterator.next();
            try {
                result.sseEmitter.send(objectMapper.writeValueAsString(API.success("ok")));
            } catch (JsonProcessingException e) {
                log.error("json 格式错误", e);
            } catch (IOException e) {
                sseEmitterMap.remove(result.clientId);
            }
        }
        log.info("当前在线: {}", sseEmitterMap.size());
    }
    private class Result {
        public String clientId;
        public long timestamp;
        public SseEmitter sseEmitter;

        public Result(String clientId, long timestamp, SseEmitter sseEmitter) {
            this.clientId = clientId;
            this.timestamp = timestamp;
            this.sseEmitter = sseEmitter;
        }
    }
}
```

### 前端代码

```js
const source = new EventSource("http://localhost:8080/start?clientId=2");
source.addEventListener("message", ({ data }) => {
    console.log(data);
});
source.addEventListener("open", (e) => {
    console.log('连接打开');
}, false);
source.addEventListener("error", (e) => {
    console.log(e);
}, false)
// 在关闭网页的时候发生断开连接
window.onbeforeunload =  () => {
    fetch('http://localhost:8080/end?clientId=2')
};
```

## WebSocket 双向推送

:::info

WebSocket 协议提供了一种在浏览器和服务器之间建立持久连接来交换数据的方法。数据可以作为 “数据包” 在两个方向上传递，而不会断开连接和其他 HTTP 请求。

:::


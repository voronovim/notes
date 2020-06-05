# Feign client templates

## Enable logging
```kotlin
@FeignClient(name = "service", url = "\${path}", configuration = [FeignClientConfig::class])
interface FeignClient {}

// If you set a @Configuration annotion it will applied to all feign clients!
class FeignClientConfig {
    @Bean
    fun feignLoggerLevel(): Logger.Level? {
        return Logger.Level.FULL
    }
}
```

## Handle exceptions

```kotlin
    val response = try {
        log.info("Calling feignClient service with request body: {}", request)
        feignClient.call(
            request = request
        ).also {
            log.info(
                "Service feignClient return answer with http statusCode:{} and http statusCodeValue:{}",
                it.statusCode,
                it.statusCodeValue
            )
        }
    } catch (re: RetryableException) {
        // retryable exception
    } catch (fe: FeignException) {
        // common exception
    }
```

> 20230309

最近在工作中需要用到 rate limiter，简单记录下自己学习 rate limiter 的过程。该部分通过两个角度来讲解 rate limiter，第一部分是基本的原理和底层实现，第二部分来看看在 Golang 中如何通过 rate limiter 来对某个 API 进行限流。

# Definition

rate limiter 是对某个系统做限流的一层组件，该组件可以避免过多的请求进入某个系统。通常来说通过一个预设的值来设定好期望的限定流量，当流量超过该值的时候，通过不同的策略（可以返回，也可以阻塞等待）来达到限流的目的。

![](Pasted%20image%2020230309234621.png)

[Rate-limiting strategies and techniques - Google](https://cloud.google.com/architecture/rate-limiting-strategies-techniques) 这片文章中提到的 rate limiter 的目的有四个：
- Preventing resource starvation
- Managing policies and quotas
- Controlling flow
- Avoiding excess costs

主要实现技术有几种：
-   **Token bucket**: A [token bucket](https://wikipedia.org/wiki/Token_bucket) maintains a rolling and accumulating budget of usage as a balance of _tokens_. This technique recognizes that not all inputs to a service correspond 1:1 with requests. A token bucket adds tokens at some rate. When a service request is made, the service attempts to withdraw a token (decrementing the token count) to fulfill the request. If there are no tokens in the bucket, the service has reached its limit and responds with backpressure. For example, in a GraphQL service, a single request might result in multiple operations that are composed into a result. These operations may each take one token. This way, the service can keep track of the capacity that it needs to limit the use of, rather than tie the rate-limiting technique directly to requests.
-   **Leaky bucket**: A [leaky bucket](https://wikipedia.org/wiki/Leaky_bucket) is similar to a token bucket, but the rate is limited by the amount that can drip or leak out of the bucket. This technique recognizes that the system has some degree of finite capacity to hold a request until the service can act on it; any extra simply spills over the edge and is discarded. This notion of buffer capacity (but not necessarily the use of leaky buckets) also applies to components adjacent to your service, such as load balancers and disk I/O buffers.
-   **Fixed window**: Fixed-window limits—such as 3,000 requests per hour or 10 requests per day—are easy to state, but they are subject to spikes at the edges of the window, as available quota resets. Consider, for example, a limit of 3,000 requests per hour, which still allows for a spike of all 3,000 requests to be made in the first minute of the hour, which might overwhelm the service.
-   **Sliding window**: Sliding windows have the benefits of a fixed window, but the rolling window of time smooths out bursts. Systems such as Redis facilitate this technique with expiring keys.

## Token bucket

该方法是最主流的一种策略。

首先在 bucket 中预先定义好 token 的总量，当有请求来的时候，如果桶内有 token，就消耗一个 token，该请求可以进入，否则的话请求就被拒绝。在固定的时间内会向桶内重新添加一定量的 token，后续的请求以同样的策略决定是否进入系统。

因此该算法有两个变量：
- **Bucket size**: bucket 可以容纳 token 的最大数量
- **Refill rate**: 每秒钟往 bucket 中添加的 token 数量

**那么在一个系统中需要多少个 Bucket 呢？**

这个数字取决于系统设计，如果需要对不同的 API 进行不同的限流，那么每个 API 都要有一个 bucket，但如果是对维度更大的粒度进行限流，就可以在全局上设计一个 bucket 即可。

# Implement in Golang

在 Golang 中有一个 rate 库，封装了 rate limiter 常见的方法。[rate package](https://pkg.go.dev/golang.org/x/time/rate)

该库采用的是 Bucket token 算法，通过动态改变 Bucket 的大小和放 token 的速率来控制 rate limiter 的控制策略。

```Go
// A Limiter controls how frequently events are allowed to happen.  
// It implements a "token bucket" of size b, initially full and refilled  
// at rate r tokens per second.  
// Informally, in any large enough time interval, the Limiter limits the  
// rate to r tokens per second, with a maximum burst size of b events.  
// As a special case, if r == Inf (the infinite rate), b is ignored.  
// See https://en.wikipedia.org/wiki/Token_bucket for more about token buckets.  
//  
// The zero value is a valid Limiter, but it will reject all events.  
// Use NewLimiter to create non-zero Limiters.  
//  
// Limiter has three main methods, Allow, Reserve, and Wait.  
// Most callers should use Wait.  
//  
// Each of the three methods consumes a single token.  
// They differ in their behavior when no token is available.  
// If no token is available, Allow returns false.  
// If no token is available, Reserve returns a reservation for a future token  
// and the amount of time the caller must wait before using it.  
// If no token is available, Wait blocks until one can be obtained  
// or its associated context.Context is canceled.  
//  
// The methods AllowN, ReserveN, and WaitN consume n tokens.  
type Limiter struct {  
   mu     sync.Mutex  
   limit  Limit  
   burst  int  
   tokens float64  
   // last is the last time the limiter's tokens field was updated  
   last time.Time  
   // lastEvent is the latest time of a rate-limited event (past or future)  
   lastEvent time.Time  
}

// Limit defines the maximum frequency of some events.  
// Limit is represented as number of events per second.  
// A zero Limit allows no events.  
type Limit float64
```

- burst 表示 bucket 的大小 
- Limit 表示 token 的速率
- tokens 表示 token 的数量

该 Limiter 的主要方法有三个，Allow, Reserve, Wait，对应了三种不同的限流策略：
- allow 表示允许当前事件（请求）发生
- reserve 表示提前预支 token 给当前事件
- wait 表示对当前事件阻塞

```go
// A Reservation holds information about events that are permitted by a Limiter to happen after a delay.  
// A Reservation may be canceled, which may enable the Limiter to permit additional events.  
type Reservation struct {  
   ok        bool  
   lim       *Limiter  
   tokens    int  
   timeToAct time.Time  
   // This is the Limit at reservation time, it can change later.  
   limit Limit  
}
```

> 为什么 Reservation 中又有一个 Limiter？



# References

- [Rate-limiting strategies and techniques - Google](https://cloud.google.com/architecture/rate-limiting-strategies-techniques)
- System Design Interview: An Insider’s Guide - Chapter 4: Design a rate limiter
- [rate package](https://pkg.go.dev/golang.org/x/time/rate)

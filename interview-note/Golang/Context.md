	1、概念
	
		在Go语言中，context包提供了一个Context类型，主要用于在多个Goroutine之间传递跨API边界和多个Goroutine之间共享的信号（如：取消信号、截止时间、超时等）。context允许控制请求的生命周期，传递元数据，并在超时、取消、截止时终止操作。主要应用于处理并发操作、取消请求、超时控制等场景中，特别是在网络请求、数据库操作和分布式系统中。

	2、作用
	
		- 取消操作：让 Goroutine 在不再需要时能够正确地中止。
		- 超时管理：为多个 Goroutine 设置统一的超时限制。
		- 请求范围的信息传递：例如：传递请求 ID、用户信息等。
		- 并发协调：多个操作可以在某个条件下统一退出，例如请求失败或超时。

	3、Context是一个接口

		type Context interface {
		    Deadline() (deadline time.Time, ok bool)
		    Done() <-chan struct{}
		    Err() error
		    Value(key interface{}) interface{}
		}

	4、应用场景
	
		取消操作：
			Context提供了取消信号，在多个Goroutine之间传播。当某个操作需要被取消时，调用cancel函数可以在其它Goroutine中监听并退出。
		
		超时控制：
			通过cintext.withTineout或context.withDedaline设置操作的最大执行时间，超时后自动取消，避免无限等待。
		
		请求链：
			在web应用或者微服务中，context可以传递请求信息，如请求ID、用户认证信息等，这样可以在多个服务之间追踪和关联请求。
			
		并发控制：
			在需要控制多个并发任务之行的场景，context可以提供取消信号或超时控制。

	5、为什么context是并发安全？
	
		（1）不可变性
			`Context` 是不可变的（immutable）。创建新的 `Context` 实际上是基于已有的 `Context` 创建一个新的实例，而原有的 `Context` 不会被改变。这意味着你可以放心地在多个 Goroutine 中传递相同的 `Context` 实例而不会发生竞争条件。
		
		（2）线程安全操作
			`Context` 的方法（如 `Deadline()`、`Done()` 等）被设计为线程安全的，它们的内部实现可以支持并发调用。这些方法内部通过合适的同步机制（如锁、原子操作）来确保多 Goroutine 并发访问时不会产生竞争条件。
			
		（3）传播信号方式
			`Done()` 通道是一个只读通道，用于通知上下游 Goroutine 何时取消操作。Go 内部使用 `select` 和事件驱动的方式来监听这个通道，并通过 `Done()` 通道的关闭来通知取消事件，而不需要任何额外的同步机制，保持了高效的并发性。
		


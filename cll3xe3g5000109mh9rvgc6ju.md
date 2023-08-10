---
title: "Embracing Go 1.21.0's slog: A Unified Logging Interface with Benchmarks against zerolog and zap"
seoDescription: "Examine Go 1.21.0's slog, comparing performance, memory, and allocations with zerolog and zap in this benchmark study"
datePublished: Wed Aug 09 2023 16:09:18 GMT+0000 (Coordinated Universal Time)
cuid: cll3xe3g5000109mh9rvgc6ju
slug: go-1-21-release-slog-with-benchmarks-zerolog-and-zap
tags: go, performance, debugging, logging, zerolog

---

The release of Go 1.21.0 marks an exciting moment in the Go community, with the introduction of a new logging package, `log/slog`. This addition brings more choices to developers for structured logging and raises the question: How does it compare with existing solutions like [`zerolog`](https://github.com/rs/zerolog) and [`uber-go/zap`](https://github.com/uber-go/zap)?

In this article, we will highlight the `slog` package and use it as a basis to explore a unified logging strategy using interfaces. Then, we'll benchmark `slog` against `zerolog` and `zap` to see how it stacks up in terms of performance.

## Introducing `slog` in Go 1.21.0

The new `slog` package comes as a part of the standard library, offering a zero-allocation logger optimized for performance and structured logging. Its interface and usage align with modern logging requirements, enabling developers to easily incorporate it into their existing applications.

### **The Importance of Interfaces in Logging**

Interfaces in Go provide a way to specify the behavior that types must implement, without detailing how they should do so. When it comes to logging, interfaces enable us to:

1. Define a standard logging behavior across our application.
    
2. Achieve modularity and flexibility by allowing multiple logger implementations.
    
3. Make unit testing easier by mocking the logger during tests.
    

### **Defining a Logging Interface**

Let's start by defining a simple logging interface:

```go
// Logger defines the standard behavior for our loggers.
type Logger interface {
    Debug(msg string, keysAndValues ...interface{})
    Info(msg string, keysAndValues ...interface{})
    Warn(msg string, keysAndValues ...interface{})
    Error(msg string, keysAndValues ...interface{})
    Fatal(msg string, keysAndValues ...interface{})
}
```

The interface specifies a set of methods, each corresponding to a log level. The `keysAndValues` variadic parameter allows us to pass structured data to our logs.

### **Implementing the Logger with ZeroLog and Zap**

[`zerolog`](https://github.com/rs/zerolog) is a zero-allocation JSON logger for Go that is both fast and reliable. It's a great choice when you need performance and structured logging. Let's implement our logger interface using ZeroLog:

```go
package logger

import (
	"os"

	"github.com/rs/zerolog"
)

type ZeroLogger struct {
	log zerolog.Logger
}

func NewZeroLogger() Log {
	zl := zerolog.New(os.Stdout).With().Timestamp().Logger()
	return &ZeroLogger{log: zl}
}

func (zl *ZeroLogger) Debug(msg string, keysAndValues ...interface{}) {
	zl.log.Debug().Fields(keysAndValues).Msg(msg)
}

func (zl *ZeroLogger) Info(msg string, keysAndValues ...interface{}) {
	zl.log.Info().Fields(keysAndValues).Msg(msg)
}

func (zl *ZeroLogger) Warn(msg string, keysAndValues ...interface{}) {
	zl.log.Warn().Fields(keysAndValues).Msg(msg)
}

func (zl *ZeroLogger) Error(msg string, keysAndValues ...interface{}) {
	zl.log.Error().Fields(keysAndValues).Msg(msg)
}

func (zl *ZeroLogger) Fatal(msg string, keysAndValues ...interface{}) {
	zl.log.Fatal().Fields(keysAndValues).Msg(msg)
}
```

```go
package logger

import (
	"go.uber.org/zap"
)

type ZapLogger struct {
	log *zap.SugaredLogger
}

func NewZapLogger() Log {
	logger, _ := zap.NewProduction()
	sugar := logger.Sugar()
	return &ZapLogger{log: sugar}
}

func (zl *ZapLogger) Debug(msg string, keysAndValues ...interface{}) {
	zl.log.Debugw(msg, keysAndValues...)
}

func (zl *ZapLogger) Info(msg string, keysAndValues ...interface{}) {
	zl.log.Infow(msg, keysAndValues...)
}

func (zl *ZapLogger) Warn(msg string, keysAndValues ...interface{}) {
	zl.log.Warnw(msg, keysAndValues...)
}

func (zl *ZapLogger) Error(msg string, keysAndValues ...interface{}) {
	zl.log.Errorw(msg, keysAndValues...)
}

func (zl *ZapLogger) Fatal(msg string, keysAndValues ...interface{}) {
	zl.log.Fatalw(msg, keysAndValues...)
}
```

With this implementation, we've married the power and efficiency of ZeroLog with the flexibility of our defined logging interface.

### Implementing a Unified Logging Interface with `slog`

Using interfaces to manage logging is a powerful pattern. Here's how we can include `slog` into an interface-based logging strategy:

```go
package logger

import (
	"log"
	"log/slog"
	"os"
)

type SlogLogger struct {
	log *slog.Logger
}

func NewSlogLogger() Log {
	sl := slog.New(slog.NewJSONHandler(os.Stdout, nil))

	return &SlogLogger{log: sl}
}

func (zl *SlogLogger) Debug(msg string, keysAndValues ...interface{}) {
	zl.log.Debug(msg, keysAndValues...)
}

func (zl *SlogLogger) Info(msg string, keysAndValues ...interface{}) {
	zl.log.Info(msg, keysAndValues...)
}

func (zl *SlogLogger) Warn(msg string, keysAndValues ...interface{}) {
	zl.log.Warn(msg, keysAndValues...)
}

func (zl *SlogLogger) Error(msg string, keysAndValues ...interface{}) {
	zl.log.Error(msg, keysAndValues...)
}

func (zl *SlogLogger) Fatal(msg string, keysAndValues ...interface{}) {
	zl.log.Error(msg, keysAndValues...)
	log.Fatal(msg)
}
```

## Benchmarking `slog`, `zerolog`, and `zap`

A critical aspect of evaluating `slog` is understanding its performance compared to existing solutions. We'll conduct a benchmark against `zerolog` and `zap`.

### Benchmark Setup:

We'll define benchmarks for `slog`, `zerolog`, and `zap` using the same logging interface, as shown earlier in the article.

```go
package main

import (
	"testing"

	"github.com/dwarvesf/test-log/logger"
)

func BenchmarkZeroLog(b *testing.B) {
	log := logger.NewZeroLogger()
	benchmarkLogger(log, b)
}

func BenchmarkSlog(b *testing.B) {
	log := logger.NewSlogLogger()
	benchmarkLogger(log, b)
}

func BenchmarkZapLog(b *testing.B) {
	log := logger.NewZapLogger()
	benchmarkLogger(log, b)
}

func benchmarkLogger(log logger.Log, b *testing.B) {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		log.Info("Benchmarking log performance", "iteration", i)
	}
}
```

### Benchmark Results:

After running the benchmarks, you may get results like:

```plaintext
goos: darwin
goarch: arm64
pkg: github.com/dwarvesf/test-log
BenchmarkZeroLog-8       6475746               174.4 ns/op            40 B/op          2 allocs/op
BenchmarkZeroLog-8       6915920               173.6 ns/op            40 B/op          2 allocs/op
BenchmarkZeroLog-8       6898648               173.7 ns/op            40 B/op          2 allocs/op
BenchmarkZeroLog-8       6909384               173.8 ns/op            40 B/op          2 allocs/op
BenchmarkZeroLog-8       6790539               175.2 ns/op            40 B/op          1 allocs/op
BenchmarkSlog-8          2070051               585.2 ns/op            40 B/op          1 allocs/op
BenchmarkSlog-8          2051962               580.0 ns/op            40 B/op          1 allocs/op
BenchmarkSlog-8          2061958               582.1 ns/op            40 B/op          1 allocs/op
BenchmarkSlog-8          2066503               580.5 ns/op            40 B/op          1 allocs/op
BenchmarkSlog-8          2067446               579.4 ns/op            40 B/op          1 allocs/op
BenchmarkZapLog-8        3109299               385.1 ns/op           168 B/op          3 allocs/op
BenchmarkZapLog-8        3103274               385.7 ns/op           168 B/op          3 allocs/op
BenchmarkZapLog-8        3100112               385.8 ns/op           168 B/op          3 allocs/op
BenchmarkZapLog-8        3109857               385.8 ns/op           168 B/op          3 allocs/op
BenchmarkZapLog-8        3101862               386.1 ns/op           168 B/op          3 allocs/op
PASS
ok      github.com/dwarvesf/test-log    23.824s
```

We ran multiple iterations for each logging library. Here are the summarized results:

### `zerolog`**:**

* Average Operation Time: **174.1 ns/op**
    
* Memory Allocated per Operation: **40 B/op**
    
* Number of Allocations per Operation: **1.8 allocs/op**
    

### `slog`**:**

* Average Operation Time: **581.4 ns/op**
    
* Memory Allocated per Operation: **40 B/op**
    
* Number of Allocations per Operation: **1 allocs/op**
    

### `zap`**:**

* Average Operation Time: **385.9 ns/op**
    
* Memory Allocated per Operation: **168 B/op**
    
* Number of Allocations per Operation: **3 allocs/op**
    

### **Analysis:**

* **Performance**: `zerolog` emerges as the fastest logging library in terms of the operation time, followed closely by `zap`. The new kid on the block, `slog`, is somewhat behind in this metric.
    
* **Memory Footprint**: Both `slog` and `zerolog` are efficient, allocating only **40 B/op**. In comparison, `zap` requires a significantly larger memory footprint, with **168 B/op**.
    
* **Allocations**: `slog` has the fewest allocations per operation, closely followed by `zerolog`. `zap` requires the most, with **3 allocs/op**.
    

## Conclusion

While `slog` might not be the fastest logger in the block, but it offers efficient memory usage and fewer allocations. This might make it a preferred choice for scenarios where memory efficiency is more critical than sheer speed.

`zerolog` proves to be a strong performer, being both fast and memory-efficient. On the other hand, `zap`, while being faster than `slog`, requires more memory and makes more allocations.

The introduction of `slog` in Go 1.21.0 invites developers to consider and balance their needs between speed, memory usage, and allocations. As always, the "best" logger depends on the specific use case and requirements of the application. This benchmark simply adds more data for making an informed decision in the evolving world of Go logging.
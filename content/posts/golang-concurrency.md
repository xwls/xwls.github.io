+++
title = 'Golang 并发编程示例'
date = 2022-06-28T15:44:40+08:00
categories = ['Golang']
tags = ['go']
summary = 'golang 并发编程的几个小例子。'
+++

## Goroutines

```go
func main() {
  // A "channel"
  ch := make(chan string)

  // Start concurrent routines
  go push("Moe", ch)
  go push("Larry", ch)
  go push("Curly", ch)

  // Read 3 results
  // (Since our goroutines are concurrent,
  // the order isn't guaranteed!)
  fmt.Println(<-ch, <-ch, <-ch)
}

func push(name string, ch chan string) {
  msg := "Hey, " + name + "! "
  ch <- msg
}
```

```text
hey, Curly!  hey, Moe!  hey, Larry!
```

Channels are concurrency-safe communication objects, used in goroutines.

See: [Goroutines](https://tour.golang.org/concurrency/1), [Channels](https://tour.golang.org/concurrency/2)

## Buffered channels

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
ch <- 3
// fatal error:
// all goroutines are asleep - deadlock!
```

```text
fatal error: all goroutines are asleep - deadlock!
```

Buffered channels limit the amount of messages it can keep.

See: [Buffered channels](https://go.dev/tour/concurrency/3)

## Closing channels

```go
ch <- 1
ch <- 2
ch <- 3
close(ch)
```

Iterates across a channel until its closed

```go
for i := range ch {
  ···
}
```

Closed if ok == false

```go
v, ok := <- ch
```

See: [Range and close](https://tour.golang.org/concurrency/4)

## WaitGroup

```go
import "sync"

func main() {
  var wg sync.WaitGroup
  
  for _, item := range itemList {
    // Increment WaitGroup Counter
    wg.Add(1)
    go doOperation(&wg, item)
  }
  // Wait for goroutines to finish
  wg.Wait()
  
}

func doOperation(wg *sync.WaitGroup, item string) {
  defer wg.Done()
  // do operation on item
  // ...
}
```

A WaitGroup waits for a collection of goroutines to finish. The main goroutine calls Add to set the number of goroutines to wait for. The goroutine calls wg.Done() when it finishes. See: [WaitGroup](https://golang.org/pkg/sync/#WaitGroup)

## 参考资料

1. [https://devhints.io/go](https://devhints.io/go)
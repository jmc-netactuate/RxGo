# RxGo
[![Build Status](https://travis-ci.org/ReactiveX/RxGo.svg?branch=master)](https://travis-ci.org/ReactiveX/RxGo)    [![Coverage Status](https://coveralls.io/repos/github/jochasinga/RxGo/badge.svg?branch=master)](https://coveralls.io/github/jochasinga/RxGo?branch=master)    
Reactive Extensions for the Go Language

## Contributions
All contributions are welcome, both in development and documentation! Be sure you check out [contributions](wiki/Contributions) for more.

## Getting Started
[ReactiveX](http://reactivex.io/), or Rx for short, is an API for programming with observable streams. This is a ReactiveX API for the Go language.

*ReactiveX* is a new, alternative way of asychronous programming to callbacks, promises and deferred. It is about processing streams of events or items, with events being any occurances or changes within the system.

>In Go, it is simpler to think of a observable stream as a channel which can `Subscribe` to a set of handler functions.

The pattern is that you `Subscribe` to an `Observable` using an `Observer`:

```go

subscription := observable.Subscribe(observer)

```

An `Observer` is a type consists of three `EventHandler` fields, the `NextHandler`, `Errhandler`, and `DoneHandler`, respectively. These handlers can be evoked with `OnNext`, `OnError`, and `OnDone` methods, respectively.

The `Observer` itself is also an `EventHandler`. This means all types mentioned can be subscribed to an `Observable`.

```go

nextHandler := func(item interface{}) interface{} {
	if num, ok := item.(int); ok {
		nums = append(nums, num)
	}
}

// Only next item will be handled. 
sub := observable.Subscribe(handlers.NextFunc(nextHandler))

```

**NOTE**: Observables are not active in themselves. They need to be subscribed to make something happen. Simply having an Observable lying around doesn't make anything happen, like sitting and watching time flies.

## Install

```bash

go get -u github.com/reactivex/rxgo

```

## Importing the Rx package
Certain types, such as `observer.Observer` and `observable.Observable` are organized into subpackages for namespace-sake to avoid redundant constructor like `NewObservable`. 

```go

import (
	"github.com/reactivex/rxgo"
	"github.com/reactivex/rxgo/observer"
	"github.com/reactivex/rxgo/observable"
	//...
)

```

## Simple Usage

```go

watcher := observer.Observer{

	// Register a handler function for every next available item.
	NextHandler: func(item interface{}) {
		fmt.Printf("Processing: %v\n", item)
	},

	// Register a handler for any emitted error.
	ErrHandler: func(err error) {
		fmt.Printf("Encountered error: %v\n", err)
	},

	// Register a handler when a stream is completed.
	DoneHandler: func() {
		fmt.Println("Done!")
	},
}

it, _ := iterable.New([]interface{}{1, 2, 3, 4, errors.New("bang"), 5})
source := observable.From(it)
sub := source.Subscribe(watcher)

// wait for the channel to emit a Subscription
<-sub

```

The above will:
- print the format string for every number in the slice up to 4.
- print the error "bang"

It is important to remember that only an `OnError` or `OnDone` can be called in a 
stream. If there's an error in the stream, the processing stops and `OnDone` will
never be called, and vice versa.

The concept is to group all side effects into these handlers and let an `Observer` or any `EventHandler` to handle them. 

```go

package main
import (
	"fmt"
	"time"

	"github.com/reactivex/rxgo"
	"github.com/reactivex/rxgo/handlers"
)

func main() {

	score := 9

	onNext := handlers.NextFunc(func(item interface{}) {
		if num, ok := item.(int); ok {
			score += num
		}
	})

	onDone := handlers.DoneFunc(func() {
		score *= 2
	})

	watcher := observer.New(onNext, onDone)

	// Create an `Observable` from a single item and subscribe to the observer.
	sub := observable.Just(1).Subscribe(watcher)
	<-sub

	fmt.Println(score) // 20
}

```

Please check out the [examples](examples/) to see how it can be applied to reactive applications.

## Recap

An `Observable` is a synchronous stream of "emitted" values which can be either an empty interface{} or error. Below is how an `Observable` can be visualized:

```bash

                                time -->

(*)-------------(o)--------------(o)---------------(x)----------------|>
 |               |                |                 |                 |
Start          value            value             error              Done

```

In **RxGo**, it's useful to think of `Observable` and `Connectable` as channels with additional ability to `Subscribe` handlers. In fact, they are basically channels. When `Subscribe` method is called on a `Observable` (or `Connect` method in case of `Connectable`), one or more goroutines are spawned to handle asynchronous processing.

Most Observable methods and operators will return the Observable itself, making it chainable.

```go

f1 := func() interface{} {
	
	// Simulate a blocking I/O
	time.Sleep(2 * time.Second)
	return 1 
}

f2 := func() interface{} {

	// Simulate a blocking I/O
	time.Sleep(time.Second)
	return 2
}

onNext := handlers.NextFunc(func(v iterface{}) {
	val := encodeVal(v)
	saveToDB(val)
})

wait := observable.Start(f1, f2).Subscribe(onNext)
sub := <-wait

if err := sub.Err(); err != nil {
	saveToLog(err)	
}

```

## This is an early project and your contributions will help shape its direction. 

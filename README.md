# go-retry

[English](./README.md)|[中文](./README-ZH.md)

## Introduction

A simple retry method.

In reality, there is often a need to retry processes that may fail, such as `IO operations`, `API interactions`, `remote calls`, and so on, so here is the current small project.

You can simply configure the number of retry attempts, interval time, conditions to stop retry, etc. If the final result does not meet the expectation after max retry attempts, it will throw a `goretry.ErrMaxApt` error.

## Get the package

Download into local or import directly.

```shell
go get -u github.com/weston-shih/go-retry
```

or

```go
import "github.com/weston-shih/go-retry"
```

## Usage

The retry option configuration is performed first.

```go
package main

import (
   "log"

   "github.com/weston-shih/go-retry"
)

func main() {
   // First of all, create a retry option, 
   // then set max attempt number and interval.
   op := NewRetryOption()
   
   // Set retry attempts
   _, err := op.SetAttempt(3)
   if err != nil {
      log.Fatal("Set attempt number ended with failure: ", err)
   }

   // Set retry attempt intervals
   _, err := op.SetBackoff(1)
   if err != nil {
      log.Fatal("Set retry interval ended with failure: ", err)
   }

   // Or you can force the configuration,
   // and invalid input will result in a panic
   op = NewRetryOption().MustSetAttempt(3).MustSetBackoff(1)
}
```

You can customize the condition that when the `Judgment` method returns `true`, the expectation is met and the retry process will be terminated.

```go
// e.g. the retry will continue if err equals ErrTest.
var ErrTest = errors.New("Just a test.")
op.SetJudgment(func(err ...interface{}) bool { return err[0] == ErrTest })
```

- Use retries for functions with error return values only
  
  ```go
    var (
      ErrOdd  = errors.New("Odd number")
      ErrEven = errors.New("Even number")
    )
    got := op.ReDo(
    func() error {
      if mod := test.seed % 2; mod == 0 {
        return ErrEven
      }
      return ErrOdd
    })
  ```

- Use retries for functions containing data and error return values

   ```go
   import "time"

   // Set a judgment condition to get a even number.
   op.SetJudgment(func(arg ...interface{}) bool { return arg[0] != 1 })
   got, err := op.ReTry(
      func() (interface{}, error){
         mod := time.Now().UTC().Second() % 2
         return mod, nil
      }
   )
   // Need to check if it fails after max retry attempts
   if err != nil {
      log.Fatal("Failed after max retry: ", err)
   }
   ```

## Feedback

Any problems encountered in the use is welcome to feedback, I'm going to follow up as soon as possible.

Also welcome to optimize the code together :)

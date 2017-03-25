# promise-go

Simple, generic promises/futures with go.

**Note:** This package is still in development.

### Installation

```bash
go get -u github.com/charlieduong94/promise-go
```

### Usage

#### Simple Promises

Creating a promise is quite easy, first create a function that you want to run
asynchronously.

**Note:** The function needs to have no arguments and return a generic interface and an error.

```go
myFunc := func () (interface{}, error) {
  return true, nil
}
```

After creating that function, pass it into the package's `Create` function. This will produce
a pointer to a `Promise` object. This object contains a method `GetResult()`, which will
block until the promise is resolved.

Here's what the `Result` struct looks like:

```go
type Result struct {
  Value interface{}
  Error error
}
```

Below is a example of how a promise can be used.

```go
package main

import (
  "fmt"
  "github.com/charlieduong94/promise-go" // exported package name is "promise"
)

type myStruct struct {
  Message string
}

func main () {
  // pass in a function that returns a generic interface and an error
  myPromise := promise.Create(func () (interface{}, error) {
    // do something you want handled async here
    time.Sleep(5 * time.Second)

    val := myStruct{"this is a return value"}

    // return any results
    return val, nil
  })

  // perform any other work here...

  // when you are ready, grab the result from your promise
  result := myPromise.GetResult()
  if result.Error != nil {
    // handle errors
  }

  // the result of your function can be accessed via the "Value" attribute
  fmt.Println(result.Value)

  // of course, you can cast the value back to whatever return type you need
  myVal, err := result.Value.(myStruct)
  if err != nil {
    // handle error
  }

  fmt.Println(myVal.Message)
}
```

#### Multiple concurrent promises

Sometimes, you need to kick off multiple concurrent functions off at the same time. With the `CreateAll`
function, you can do that easily. This returns a pointer to a `CombinedPromise`, which will resolve with
multiple values.

Calling `GetResults` with this object will return `CombinedResult` struct.

```go
type CombinedResult struct {
  Values []interface{}
  Error error
}
```

```go
multiplePromises := promise.CreateAll(
  func () (interface{}, error) {
    time.Sleep(1 * time.Second)
    return 26, nil
  },
  func () (interface{}, error) {
    time.Sleep(2 * time.Second)
    return 58, nil
  },
)

combinedResult := multiplePromises.GetResult()
if combinedResult.Error != nil {
  // handle errors
}

fmt.Println(combinedResult.Values) // prints "[26 58]"
```

There may also be times when you may want to start promises at different times, but then await for all
of them to be resolved at a later phase.

```go
promiseA := promise.Create(func () (interface{}, error) {
  return doSomethingA()
})

// do some work additional work

promiseB := promise.Create(func () (interface{}, error) {
  return doSomethingB()
})

combinedPromise := promise.All(promiseA, promiseB)
combinedResult := combinedPromise.GetResult()
if combinedResult.Error != nil {
  // handle errors
}

fmt.Println(combinedResult.Values)
```

**NOTE:** At the moment, a `CombinedPromise` cannot be passed into `All` (this is coming soon).

---
layout: post
title: Golang and Errors
image: /img/golang.png
tags: [go, programming]
comments: true
---

In go, error is an interface. Therefore, we can define our own errors for our types.

```go
type error interface {
    Error() string
}
```

For instance, I am going to define a type called custom type.

```go
type ComedyError string
```

What I have done above!
_string_ is a built type in go, but I need to declare that this is a ComedyError.

There is no implements keyword like java. So how can we implement an error interface to our own error defined type?
Actually, when we use **receive method** with a name of the function which declared in the interface, Go will understand that this type implements that interface.

Now we have a new term which is receiving method.
Let's understand what is receive methods firstly.

```go
type MyString string

func (c MyString) hello() {
	fmt.Print(c)
}
```

This is a receive method that belongs to ComedyError. Go is not Object-Oriented language. Therefore, we haven't objects. That's mean we haven't functions which bind with types. In Go, we can bind our function to type with receive methods.

There is a hello function, now we can use,

```go
var foo MyString
foo = MyString("hi")
foo.hello()
```

Now I declare a MyString type and I can use hello function with type. The output of code will be the value of MyString. This called the receive method.

I need to write a receive method to my ComedyError type and the name of the function should be Error because this name declared into error interface.

```go
type ComedyError string

func (c ComedyError) Error() string {
    return string(c)
}
```

Now I declared my own error type. Now let's use it!

```go
var err error
err = ComedyError("Keep slience")
fmt.Println(err)
```

In the first line, you can see an error interface with the name of err. I can assign my error type to error interface because I already declared its function.

No panic !

There is a built-in function in order to stop program execution. We can easily use panic.

```go
panic("oh no!)
```

Error handling is one of the most important parts of programming. During our flow, there could be errors. Even, we can throw an error in order to cut execution of program.
In Go there is also a built-in **recover** function in order to catch panic errors.

```go
func calmDown() {
	p := recover()
	err, ok := p.(error)
	if ok {
		fmt.Println(err.Error())
	}
}
```

When we call this function after panic. Recover will catch value of panic. This value will be any type, so we need to assert that type which is called type assertions. When we catch value which comes from panic, we need to be sure that this value is **error**. Therefore we can use Error function.

```go
err, ok := p.(error)
```

Thanks to this assertion, we can understand that p is an error.

Then we can print error.

Wait for a minute. How can we call this function after panic. As I said panic stops execution of the program.
There is a defer function in order to execute functions after panics.

```go
defer calmDown()
err := ComedyError("Keep slience")
panic(err)
```

With defer keyword we sure that calmDown will be executed after panic!

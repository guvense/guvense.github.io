---
layout: post
title: Go Recover Error
image: /img/go.png
tags: [go, programming]
comments: true
---

In go, error is a interface. Therefore we can define our own errors for our types.

```
type error interface {
    Error() string
}
```

For instance, I am going to define a type which called custom type.

```
type ComedyError string
```

What I done above!
_string_ is a built type in go, but I need to declare that this is a ComedyError.

There is no implements keyword like java. So how can we implement error interface to our own error defined type?
Actually, when we use **receive method** with name of function which declared in interface, go will understand that this type implements that interface.

Now we have a new term which is receive method.
Lets understand that what is receive methods firstly.

```
type MyString string

func (c MyString) hello() {
	fmt.Print(c)
}
```

This is receive method which belongs to ComedyError. Go is not consist of OOP. Therefore we haven't objects. Thats mean we haven't function which bind with types. In go, we can bind our function to type with receive methods.

There is a hello function, now we can use

```
	var foo MyString
	foo = MyString("hi")
	foo.hello()
```

Now I declare a MyString type and I can use hello function with type. The output of code will be value of MyString. This called receive method.

Now I need to write a recieve method to my ComedyError type and the name of function should be Error because this name declared into error interface.

```
    type ComedyError string

    func (c ComedyError) Error() string {
	    return string(c)
}
```

Now I declared my own error type. Now lets use it!

```
	var err error
	err = ComedyError("Keep slience")
	fmt.Println(err)
```

First line, you can see a error interface with name of err. I can assign my error type to error interface because I already declared its function.

No panic !

There is a built-in function in order to stop program execution. We can easly use panic.

```
panic("oh no!)
```

Error handling is one of the most important part of programming. During our flow there could be errors. Even we can throw an error in order to cut excution of program.
In go there is also a buit-in **recover** function in order to catch panic errors.

```
func calmDown() {
	p := recover()
	err, ok := p.(error)
	if ok {
		fmt.Println(err.Error())
	}
}
```

When we call this function after panic. Recover will catch a value of panic. This value will be any type, so we need to assert that type which is called type assertions. When we catch value which comes from panic, we need to be sure that this value is **error**. Therefore we can use Error function.

```
	err, ok := p.(error)
```

Thanks to this assertion, we can understand that p is a error.

Then we can print error.

Wait a minite. How can we call this function after panic. As I said panic stops execution of program.
There is a defer function in order to execute functions after panics.

```
	defer calmDown()
	err := ComedyError("Keep slience")
	panic(err)
```

With defer keyword we sure that calDown will be excuted after panic!

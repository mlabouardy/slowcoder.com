---
title: Errors and Logging in Go
date: 2018-12-03 12:59:40
tags:
- Go
- Programming
categories:
- Programming
cover_index: /assets/Errors-and-Logging-in-Go.png
---
*Learn how to deal with errors in Go in this article by Tarik Guney, a hands-on software engineer with a passion for programming. His years of experience in the industry always led him to explore new technologies and using them in real-world scenarios.*

https://github.com/PacktPublishing/Hands-On-Go-Programming/tree/master/Chapter06

This article will show you how to deal with errors and return an error when you need to. Go's error mechanism is different than some of the other popular languages, and this section will teach you how to work with errors the Go way. You’ll also learn how to perform simple logging operations in your application to get insights into your running application for better debugging. 

**Creating custom error types**

If you come from languages such as C# and Java, you may find the error mechanism a little different in Go. Moreover, the way you create your own custom error is very simple because Go is a duck-typed language, which means that you are good to go as long as your struct satisfies an interface. 

Go ahead and create your own custom error using a new type. So, you’ll have two fields: *ShortMessage* and *DetailedMessage* of string type. You can have as many fields as you want to capture more information about your errors. Furthermore, to satisfy the error interface, implement a new method, **MyError*, which will return a string value, and you can output this error to either your console or some log file.

Then, return the error message. So, the way you do this is very simple: you can just return this error type from your method. Imagine that you have a *doSomething* method that returns an error. Imagine you did some lines of codes in that method and it returns an error for some reason, such as a *ShortMessage* instance of "Wohoo something happened!". 

Of course, you’ll probably need to use more meaningful messages here, and don't forget to use this & operator. It will get the address of your **MyError* object since you are working with a pointer here. If you don't do this, you will see that there's a type error, and one way to fix this is to remove that * pointer. But you probably don't want to have multiple copies of the same object, so instead send a reference back so that you have better memory management. Here’s the entire code:

{% codeblock %}
package main

import "fmt"

type MyError struct{
  ShortMessage string
  DetailedMessage string
  //Name string
  //Age int
}

func (e *MyError) Error() string {
  return e.ShortMessage + "\n" +e.DetailedMessage

}
  func main(){
    err:= doSomething()
    fmt.Print(err)
}
func doSomething() error {
  //Doing something here...
  return &MyError{ShortMessage:"Wohoo something happened!", DetailedMessage:"File cannot found!"}
}
{% endcodeblock %}

So, run this and it will return some errors; you're just going to add err here and then run it to the console. You can see that your message or error message is written to the console, which is shown in the following screenshot:

![](https://d255esdrn735hr.cloudfront.net/graphics/9781789531756/graphics/019cb7b9-e27a-44af-9056-bb4599a947a0.png)


That's how you can simply create your own error message types. In the next section, you’ll learn the try...catch equivalent in Go.

**The try...catch equivalent in Go**

Unlike in other languages, there is no try...catch block in Go. In this section, you’ll see how Go handles basic errors. So, the first thing is to handle the errors returned by an API calls. You can use the time.Parse() method for that as it accepts a layout and a value string. It returns two things, one is the parsedDate and the other one is an error. Instead of returning an exception, Go returns an error as its second parameter most of the time.
 
 
Now, the way you can handle this is to check whether the parsedDate is nil. If it's not nil in Go, then you know that an error has happened and you need to handle it. If nothing happens, you can safely proceed to your next line to write the content of your parsedDate to output. So, for this, check the following code example:

{% codeblock %}
package main

import (
  "time"
  "fmt"
)

func main(){
  parsedDate, err:= time.Parse("2006", "2018")
  if err != nil {
    fmt.Println("An error occured", err.Error())
  }else{
    fmt.Println(parsedDate)
  }
}
{% endcodeblock %}

The preceding code will give you the following output:

![](https://d1ldz4te4covpm.cloudfront.net/graphics/9781789531756/graphics/b2a1289f-51c1-48e6-b03a-5b4c155e93c6.png)

You can see that it works fine. What happens if you add some string values after 2018? Add abc and run the code. If you see the following screenshot, you can see that an error occurred in parsing time; it also added the error message An error occured parsing time "2018 abc": extra text: abc, as shown in the following screenshot:
 
 ![](https://d255esdrn735hr.cloudfront.net/graphics/9781789531756/graphics/defd34e1-7294-420d-8bfc-11afcf72a08e.png)

Now, the second part of this section is when you return an error yourself. Suppose you have the doSomething function and it returns an err type. Check the following code:

{% codeblock %}
package main
import (
  "fmt"
  "errors"
)
func main(){
  _, err := doSomething()
  if err != nil {
    fmt.Println(err)
  }
}
func doSomething() (string,error) {
  return "", errors.New("Something happened.")
}
{% endcodeblock %}

The preceding code will give the following output:

![](https://d1ldz4te4covpm.cloudfront.net/graphics/9781789531756/graphics/02c0ec49-f77e-46fd-98bb-6fc8d7de58e8.png)

So, that's how you can do a simple try...catch equivalent in Go. In the next section, you’ll see how to do simple logging in your application.

**Doing simple logging in your application**

There are various ways to do logging, and there are also third-party packages that allow you to do it as well. But you’ll use the log package that Go provides. So, create a new file using the os package, and if, somehow, when creating the log file, an error happens, write it to the console. You can also use the defer function. Before the main method exits, this defer function will be called, and the next step will be to set the output:

{% codeblock %}
package main
import (
  "os"
  "fmt"
  "log"
)
func main(){
  log_file, err := os.Create("log_file")
  if err != nil{
    fmt.Println("An error occured...")
  }
  defer log_file.Close()
  log.SetOutput(log_file)

  log.Println("Doing some logging here...")
  log.Fatalln("Fatal: Application crashed!")
}
{% endcodeblock %}

When you run the preceding code, a new file called log_file is created with the following content:

![](https://d1ldz4te4covpm.cloudfront.net/graphics/9781789531756/graphics/3cca96f2-e896-4a61-a3da-47c0d8a9431d.png)

You may wonder what the difference is between a fatal error and a normal info error. Reorder the two lines and see how that new order behaves. Hence, you’ll run Fatalln first and then Println as follows:

{% codeblock %}
package main
import (
  "os"
  "fmt"
  "log"
)
func main(){
  log_file, err := os.Create("log_file")
  if err != nil{
    fmt.Println("An error occured...")
  }
  defer log_file.Close()
  log.SetOutput(log_file)
  log.Fatalln("Fatal: Application crashed!")
  log.Println("Doing some logging here...")
}
{% endcodeblock %}

If you now run the preceding code and check the content of the log_file, you will see that the second Println did not get written:

![](https://d1ldz4te4covpm.cloudfront.net/graphics/9781789531756/graphics/3d2ea2b3-2681-45aa-9ae5-8b09c8fe5614.png)

The difference is that Fatalln is similar to Println but only followed by a call to os.Exit. So, it basically writes a log and exits out of the application, which is the simple difference between the two. This is how you can simply do logging in your application. 

*If you found this article interesting, you can explore [Hands-On Go Programming](https://www.amazon.com/Hands-Go-Programming-real-world-challenges-ebook/dp/B07GDHC34N) to learn Go programming with concise examples providing solutions to common problems. [Hands-On Go Programming](https://www.amazon.com/Hands-Go-Programming-real-world-challenges-ebook/dp/B07GDHC34N) helps you understand how various answers are programmed in the Go language.*

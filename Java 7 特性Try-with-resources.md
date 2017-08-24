# Java 7 特性Try-with-resources

- [Resource Management With Try-Catch-Finally, Old School Style](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html#resource-management-old-school)
- [Try-with-resources](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html#try-with-resources)
- [Using Multiple Resources](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html#using-multiple-resources)
- [Custom AutoClosable Implementations](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html#custom-autoclosable-implementations)

Try-with-resources in Java 7 is a new exception handling mechanism that makes it easier to correctly close resources that are used within a try-catch block.

Here is a list of topics covered in this text:


## Resource Management With Try-Catch-Finally, Old School Style

Managing resources that need to be explicitly closed is somewhat tedious before Java 7.

Look at the following method which reads a file and prints it to the `System.out`:

```java
private static void printFile() throws IOException {
    InputStream input = null;

    try {
        input = new FileInputStream("file.txt");

        int data = input.read();
        while(data != -1){
            System.out.print((char) data);
            data = input.read();
        }
    } finally {
        if(input != null){
            input.close();
        }
    }
}
```

The code marked in bold is where the code can throw an `Exception`. As you can see, that can happen in 3 places inside the `try`-block, and 1 place inside the `finally`-block.

The `finally` block is always executed no matter if an exception is thrown from the `try` block or not. That means, that the `InputStream` is closed no matter what happens in the `try` block. Or, attempted closed that is. The `InputStream`'s `close()` method may throw an exception too, if closing it fails.

Imagine that an exception is thrown from inside the `try` block. Then the `finally` block is executed. Imagine then, that an exception is also thrown from the `finally` block. Which exception do you think is propagated up the call stack?

The exception thrown from the `finally` block would be propagated up the call stack, even if the exception thrown from the `try` block would probably be more relevant to propagate.



## Try-with-resources

In Java 7 you can write the code from the example above using the `try-with-resource` construct like this:

```java
private static void printFileJava7() throws IOException {

    try(FileInputStream input = new FileInputStream("file.txt")) {

        int data = input.read();
        while(data != -1){
            System.out.print((char) data);
            data = input.read();
        }
    }
}
```

Notice the first line inside the method:

```java
try(FileInputStream input = new FileInputStream("file.txt")) {
```

This is the `try-with-resources` construct. The `FileInputStream` variable is declared inside the parentheses after the `try` keyword. Additionally, a `FileInputStream` is instantiated and assigned to the variable.

When the `try` block finishes the `FileInputStream` will be closed automatically. This is possible because `FileInputStream` implements the Java interface `java.lang.AutoCloseable`. All classes implementing this interface can be used inside the `try-with-resources`construct.

If an exception is thrown both from inside the `try-with-resources` block, and when the `FileInputStream` is closed (when `close()` is called), the exception thrown inside the `try` block is thrown to the outside world. The exception thrown when the `FileInputStream` was closed is suppressed. This is opposite of what happens in the example first in this text, using the old style exception handling (closing the resources in the `finally` block).

## Using Multiple Resources

You can use multiple resources inside a `try-with-resources` block and have them all automatically closed. Here is an example:

```java
private static void printFileJava7() throws IOException {

    try(  FileInputStream     input         = new FileInputStream("file.txt");
          BufferedInputStream bufferedInput = new BufferedInputStream(input)
    ) {

        int data = bufferedInput.read();
        while(data != -1){
            System.out.print((char) data);
    data = bufferedInput.read();
        }
    }
}
```

This example creates two resources inside the parentheses after the `try` keyword. An `FileInputStream` and a `BufferedInputStream`. Both of these resources will be closed automatically when execution leaves the `try` block.

The resources will be closed in reverse order of the order in which they are created / listed inside the parentheses. First the `BufferedInputStream` will be closed, then the `FileInputStream`.

## Custom AutoClosable Implementations

The `try-with-resources` construct does not just work with Java's built-in classes. You can also implement the `java.lang.AutoCloseable` interface in your own classes, and use them with the `try-with-resources` construct.

The `AutoClosable` interface only has a single method called `close()`. Here is how the interface looks:

```java
public interface AutoClosable {

    public void close() throws Exception;
}
```

Any class that implements this interface can be used with the `try-with-resources` construct. Here is a simple example implementation:

```java
public class MyAutoClosable implements AutoCloseable {

    public void doIt() {
        System.out.println("MyAutoClosable doing it!");
    }

    @Override
    public void close() throws Exception {
        System.out.println("MyAutoClosable closed!");
    }
}
```

The `doIt()` method is not part of the `AutoClosable` interface. It is there because we want to be able to do something more than just closing the object.

Here is an example of how the `MyAutoClosable` is used with the `try-with-resources` construct:

```java
private static void myAutoClosable() throws Exception {

    try(MyAutoClosable myAutoClosable = new MyAutoClosable()){
        myAutoClosable.doIt();
    }
}
```

Here is the output printed to `System.out` when the method `myAutoClosable()` is called:

```java
MyAutoClosable doing it!
MyAutoClosable closed!
```

As you can see, `try-with-resources` is a quite powerful way of making sure that resources used inside a `try-catch` block are closed correctly, no matter if these resources are your own creation, or Java's built-in components.


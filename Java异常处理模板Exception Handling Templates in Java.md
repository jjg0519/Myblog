# Java异常处理模板Exception Handling Templates in Java

- [Correct Error Handling is Tedious to Write](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html#correct-error-handling)
- [Using Interfaces Instead of Subclassing](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html#using-interfaces)
- [Static Template Methods](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html#static-template-methods)
- [Summary](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html#summary)

Before you read this text, it is a good idea to have read the text ["Fail Safe Exception Handling"](http://tutorials.jenkov.com/java-exception-handling/fail-safe-exception-handling.html).

## Correct Error Handling is Tedious to Write

Correct exception handling code can be tedious to write. Try-catch blocks also clutter the code and makes it harder to read. Look at the example below:

```java
   InputStream       input            = null;
    IOException processException = null;
    try{
        input = new FileInputStream(fileName);

        //...process input stream...
    } catch (IOException e) {
        processException = e;
    } finally {
       if(input != null){
          try {
             input.close();
          } catch(IOException e){
             if(processException != null){
                throw new MyException(processException, e,
                  "Error message..." +
                  fileName);
             } else {
                throw new MyException(e,
                    "Error closing InputStream for file " +
                    fileName;
             }
          }
       }
       if(processException != null){
          throw new MyException(processException,
            "Error processing InputStream for file " +
                fileName;
    }
```

In this example no exceptions are lost. If an exception is thrown from within the try block, and another exception is thrown from the input.close() call in the finally block, both exceptions are preserved in the MyException instance, and propagated up the call stack.

That is how much code it takes to handle the processing of an input stream without any exceptions being lost. In fact it only even catches IOExceptions. RuntimeExceptions thrown from the try-block are not preserved, if the input.close() call also throws an exception. Isn't it ugly? Isn't it hard to read what is actually going on? Would you remember to write all that code everytime you process an input stream?

Luckily there is a simple design pattern, the Template Method, that can help you get the exception handling right everytime, without ever seeing or writing it in your code. Well, maybe you will have to write it once, but that's it.

What you will do is to put all the exception handling code inside a template. The template is just a normal class. Here is a template for the above input stream exception handling:



```java

package com.pikai.demo;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * Created by wangm on 2017/7/28.
 */
public abstract class InputStreamProcessingTemplate {
    public void process(String fileName) {
        IOException processException = null;
        InputStream input = null;
        try {
            input = new FileInputStream(fileName);

            doProcess(input);
        } catch (IOException e) {
            processException = e;
        } finally {
            if (input != null) {
                try {
                    input.close();
                } catch (IOException e) {
                    if (processException != null) {
                        throw new MyException(processException, e,
                                "Error message..." +
                                        fileName);
                    } else {
                        throw new MyException(e,
                                "Error closing InputStream for file " +
                                        fileName);
                    }
                }
            }
            if (processException != null) {
                throw new MyException(processException,
                        "Error processing InputStream for file " +
                                fileName);
            }
        }
    }

//override this method in a subclass, to process the stream.

    public abstract void doProcess(InputStream input) throws IOException;
}

```

All the exception handling is encapulated inside the InputStreamProcessingTemplate class. Notice how the process() method calls the doProcess() method inside the try-catch block. You will use the template by subclassing it, and overriding the doProcess() method. To do this, you could write:

```jaVA
   new InputStreamProcessingTemplate(){
        public void doProcess(InputStream input) throws IOException{
            int inChar = input.read();
            while(inChar !- -1){
                //do something with the chars...
            }
        }
    }.process("someFile.txt");
```

This example creates an anonymous subclass of the InputStreamProcessingTemplate class, instantiates an instance of the subclass, and calls its process() method.

This is a lot simpler to write, and easier to read. Only the domain logic is visible in the code. The compiler will check that you have extended the InputStreamProcessingTemplate correctly. You will typically also get more help from your IDE's code completion when writing it, because the IDE will recognize both the doProcess() and process() methods.

You can now reuse the InputStreamProcessingTemplate in any place in your code where you need to process a file input stream. You can easily modify the template to work for all input streams and not just files.

## Using Interfaces Instead of Subclassing

Instead of subclassing the InputStreamProcessingTempate you could rewrite it to take an instance of an InputStreamProcessor interface. Here is how it could look:

```java
public interface InputStreamProcessor {
    public void process(InputStream input) throws IOException;
}


public class InputStreamProcessingTemplate {

    public void process(String fileName, InputStreamProcessor processor){
        IOException processException = null;
        InputStream input = null;
        try{
            input = new FileInputStream(fileName);

            processor.process(input);
        } catch (IOException e) {
            processException = e;
        } finally {
           if(input != null){
              try {
                 input.close();
              } catch(IOException e){
                 if(processException != null){
                    throw new MyException(processException, e,
                      "Error message..." +
                      fileName;
                 } else {
                    throw new MyException(e,
                        "Error closing InputStream for file " +
                        fileName);
                 }
              }
           }
           if(processException != null){
              throw new MyException(processException,
                "Error processing InputStream for file " +
                    fileName;
        }
    }
}
```

Notice the extra parameter in the template's process() method. This is the InputStreamProcessor, which is called from inside the try block (processor.process(input)). Using this template would look like this:

```java
  new InputStreamProcessingTemplate()
        .process("someFile.txt", new InputStreamProcessor(){
            public void process(InputStream input) throws IOException{
                int inChar = input.read();
                while(inChar !- -1){
                    //do something with the chars...
                }
            }
        });
```

It doesn't look much different from the previous usage, except the call to the InputStreamProcessingTemplate.process() method is now closer to the top of the code. This may be easier to read.

## Static Template Methods

It is also possible to implement the template method as a static method. This way you don't need to instantiate the template as an object every time you call it. Here is how the InputStreamProcessingTemplate would look as a static method:

```java
public class InputStreamProcessingTemplate {

    public static void process(String fileName,
    InputStreamProcessor processor){
        IOException processException = null;
        InputStream input = null;
        try{
            input = new FileInputStream(fileName);

            processor.process(input);
        } catch (IOException e) {
            processException = e;
        } finally {
           if(input != null){
              try {
                 input.close();
              } catch(IOException e){
                 if(processException != null){
                    throw new MyException(processException, e,
                      "Error message..." +
                      fileName);
                 } else {
                    throw new MyException(e,
                        "Error closing InputStream for file " +
                        fileName;
                 }
              }
           }
           if(processException != null){
              throw new MyException(processException,
                "Error processing InputStream for file " +
                    fileName;
        }
    }
}
```

The process(...) method is simply made static. Here is how it looks to call the method:

```java
  InputStreamProcessingTemplate.process("someFile.txt",
        new InputStreamProcessor(){
            public void process(InputStream input) throws IOException{
                int inChar = input.read();
                while(inChar !- -1){
                    //do something with the chars...
                }
            }
        });
```

Notice how the call to the template's process() method is now a static method call.

## Summary

Exception handling templates are a simple yet powerful mechanism that can increase the quality and readability of your code. It also increases your productivity, since you have much less code to write, and less to worry about. Exceptions are handled by the templates. And, if you need to improve the exception handling later in the development process, you only have a single spot to change it in: The exception handling template.

The Template Method design pattern can be used for other purposes than exception handling. The iteration of the input stream could also have been put into a template. The iteration of a ResultSet in JDBC could be put into a template. The correct execution of a transaction in JDBC could be put into a template. The possibilities are endless.

Context reuse and templates are a
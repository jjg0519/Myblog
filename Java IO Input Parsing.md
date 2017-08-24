# Java IO: Input Parsing

Some of the classes in the Java IO API are designed to help you parse input. These classes are:

- [PusbackInputStream](http://tutorials.jenkov.com/java-io/pushbackinputstream.html)
- [PusbackReader](http://tutorials.jenkov.com/java-io/pushbackreader.html)
- [StreamTokenizer](http://tutorials.jenkov.com/java-io/streamtokenizer.html)
- [PushbackReader](http://tutorials.jenkov.com/java-io/pushbackreader.html)
- [LineNumberReader](http://tutorials.jenkov.com/java-io/linenumberreader.html)

It is not the purpose of this text to give you a complete course in parsing of data. The purpose was rather to give you above quick list of classes related to parsing of input data.

If you have to parse data you will often end up writing your own classes that use some of the classes in this list. I know I did when I wrote the parser for the Butterfly Container Script. I used the `PushbackInputStream` at the core of my parser, because sometimes I needed to read ahead a character or two, to determine what the character at hand meant.

I have a real life example that uses the `PushbackReader` in my article about [Replace Strings in Streams, Arrays, Files](http://tutorials.jenkov.com/java-howto/replace-strings-in-streams-arrays-files.html) tutorial. The example creates a `TokenReplacingReader` which can replace tokens of the format `${tokenName}` in data read from an underlying `Reader` with values of your own choosing. The user of the`TokenReplacingReader` cannot see that this replacement takes place.
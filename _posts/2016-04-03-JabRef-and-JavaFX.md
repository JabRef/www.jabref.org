---
title: JabRef and JavaFX
id: jabref-javafx
bg: white
color: black
style: left
---


JabRef as of today is using swing components to build its gui.
However as of java version 1.8 a new more promissing technology is available called JavaFX 8 which is a more advanced version of JavaFX 1.X or 2.X.
Since JabRef recently switched to java version 1.8 it is getting more and more interesting how to migrate the user interface to this new technology.

If we try to just include JavaFX components right now we would face a _IllegalStateException_ with the message _Toolkit not initialized_.
The JavaFX toolkit gets automatically initialized when an application is started through the _launch_ method of a class that extends the _Application_ class from the package _javafx.application_.
As JabRef has not been using any JavaFX components yet it does not start this way and we need to change its start behaviour.
For this we have to look into the class _JabRefMain_ located in the package _net.sf.jabref_ of the source folder _src/main/java_.
As mentioned earlier we need to inherit the _Application_ class which means we need to add `extends Application` after the class name.  
Following is the new class signature.

```java
    public class JabRefMain extends Application {
        ...
    }
```

Once we did this change eclipse or any other good IDE will give us a warning or error message saying we have to implement the inherited _start_ method.
This is where our JavaFX application will begin running and therefore we need to put our call to the _start_ method of JabRef in it.
Unfortunately the array of specified start arguments are only available in the java _main_ method which is why we need to create a static array holding the start arguments.
We also have to be aware that the JavaFX runtime will implicitly shutdown after the last JavaFX window has closed.
This is an unwanted behaviour as for example we would also like to reopen a JavaFX implemented dialog even though our main JabRef window is still implemented as a swing frame. This would give us the chance to slowly migrate swing component to JavaFX without having to start with the biggest frame first.
Fortunately we can change this behaviour by adding the line `Platform.setImplicitExit(false);`.
We just need to keep in mind that we have to manually close the JavaFX application thread when closing the application but we will come to this later.
The overwritten _start_ method and the array for our starting arguments of the class _JabRefMain_ looks like the following.

```java
    private static String[] arguments;

    @Override
    public void start(Stage primaryStage) throws Exception {
        new JabRef().start(arguments);
        Platform.setImplicitExit(false);
    }
```
  
The call to the JavaFX _launch_ method needs to go in the _main_ method which is the entry point for all java applications.
As we created an array for the starting arguments we also initialize it in the _main_ method.  
The _main_ method of the class _JabRefMain_ looks like the following.

```java
    public static void main(String[] args) {
        arguments = args;
        launch(arguments);
    }
```

As we specified earlier the JavaFX thread does no longer implicitly shutdown so we have to manually call the _exit_ method on the JavaFX application thread when closing the application.
For this we locate the inner class _CloseAction_ of the class _JabRefFrame_ in the package _net.sf.jabref.gui_ which takes care of the closing process of JabRef. The class\' _actionPerformed_ method is called when we press the close button so we add the line `Platform.exit();` at the end of it.
This changes the method to the following.

```java
    @Override
    public void actionPerformed(ActionEvent e) {
        quit();
        Platform.exit();
    }		
```

With these modifications we would have a good base and could start to migrate the user interface to JavaFX.
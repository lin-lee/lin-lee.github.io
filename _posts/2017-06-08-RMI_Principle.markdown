---

# Header1

layout : post
title: "RMI Principle"
date: 2017-06-08
categories: Blog
tags: RMI
---

This article is talk about `RMI principle`, source code is `openjdk-7-fcs-src-b147-27_jun_2011.zip` .

Let start, in oracle docs <http://docs.oracle.com/javase/7/docs/technotes/guides/rmi/index.html> and <http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136424.html> .
at first link, see the Tutorals->Getting Started. next is a demo about RMI.

1, Define the remote interface
======
A remote object is an instance of a class that implements a remote interface.
<br />A remote interface extends the interface `java.rmi.Remote` and declares a set of
remote methods. Each remote method must declare `java.rmi.RemoteException` or a
superclass of RemoteException) in its throws clause
Here is the interface definition for the remote interface used in this example
> Hello.java

	import java.rmi.Remote;
	import java.rmi.RemoteException;

	public interface Hello extends Remote {
  	  String sayHello() throws RemoteException;
	}

Remote method invocations can fail in many additional ways compared to local method invocations(such as network-related communication problems and server problems), and remote methods will report such failures by throwing a `java.rmi.RemoteException`.

2, Implement the server
======
A "server" class, in this context, is the class which has a `main` method that creates an instance of the remote object implementation, exports the remote object, and then binds that instance to a name in a Java RMI registry. The class that contains this `main` method could be the implementation class itself, or another class entirely.
<br />
The server's main method does the following:
+ Create and export a remote object
+ Register the remote object with a Java RMI registry

> this is Server.java


	import java.rmi.registry.Registry;
	import java.rmi.registry.LocateRegistry;
	import java.rmi.RemoteException;
	import java.rmi.server.UnicastRemoteObject;
	import sun.rmi.registry.RegistryImpl_Stub;

	public class Server implements Hello {

    public Server() {}

    public String sayHello() {
        return "Hello, world!";
    }

    public static void main(String args[]) {

        try {
	    // this is create Registry in this Jvm.
	    //or you could terminal rmic Server.
            LocateRegistry.createRegistry(1099);
            
            Server obj = new Server();
            Hello stub = (Hello) UnicastRemoteObject.exportObject(obj, 0);
            //RegistryImpl_Stub
            // Bind the remote object's stub in the registry
            Registry registry = LocateRegistry.getRegistry();
            registry.bind("Hello", stub);

            System.err.println("Server ready");
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}

The implementation class Server.sayHello() does not need to declare that it throw any exception because the method implementation itself does not throw `RemoteException` nor does it throw any other checked exceptions.

#### Create and export a remote object

# wom

hello[^hello]

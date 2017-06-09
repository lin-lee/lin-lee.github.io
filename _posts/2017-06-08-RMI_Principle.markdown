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
	    //or you could terminal rmiregistry & or
	    // start rmiregistry 2001.
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
The main method of the server needs to create the remote object that provides the service.Additionally, the remote object must be `export` to the `Java RMI runtime` so that it may receive incoming remote calls. This can be done as follows:
> Server object = new Server(); <br />
> Hello stub = UnicastRemoteObject.exportObject(obj, 0);

The static method `UnicastRemoteObject.exportObject exports the supplied remote object to receive incoming remote method invocations on an anonymous TCP port and returns the stub for the remote object to pass to clients. As a result of the 
`exportObject` call, the runtime may begin to listen on a new server socket or may use a shared server socket to accept incoming remote calls for the remote object. The returned stub implements the same set of remote interfaces as the remote object's class and contains the host name and port over whick the remote object can be contacted.

*Note:*As of the J2SE 5.0 release, stub classes for remote objects no longer need to be pregenerated using the `rmic` stub compiler, unless the remote object needs to support clients running in pre-5.0 VMs

#### Register the remote object with a Java RMI registry
For a caller(client,peer, or applet)to be able to invoke a method on a remote object, that caller must first obtain a stub for the remote object. For bootstrapping, Java RMI provides a registry API for applications to bind a name to a remote object's stub and for clients to look up remote objects by name in order to obtain their stubs.

A Java RMI registry is a simplified name service that allows clients to get a reference(a stub) to a remote object. In general, a registry is used(if at all) only to locate the first remote obejct a client needs to use. Then, typically, that first object would in turn provide application-specific support for finding other objects. For example, the reference can be obtained as a parameter to, or a return value from, another remote method call. For a discussing on how this works, please take a look at [Applying the Factory Pattern to Java RMI](http://docs.oracle.com/javase/7/docs/technotes/guides/rmi/Factory.html).

Once a remote object is registered on the server, callers can look up the object by name, obtain a remote object reference, and then invoke remote methods on the object.

The following code in the server obtains a stub for a registry on the local host and default registry port and then uses the registry stub to bind the name "Hello" to the remote object's stub in that registry:
> Registry registry = LocateRegistry.getRegistry();
> <br /> registry.bind("Hello", stub);

The static method `LocateRegistry.getRegistry` that takes no arguments returns a stub that implements the remote interface `java.rmi.registry.Registry` and sends invocations to the registry on server's local host on the default registry port of `1099`. The `bind` method is then invoked on the `registry` stub in order to bind the remote object's stub to the name "Hello" in the registry.

*Note:* The call to `LocateRegistry.getRegistry` simply returns an appropriate stub for a registry. The call does not check to see if a registry is actually running. If no registry is running on TCP port 1099 of the local host when the bind method is invoked, the server will fail with a `RemotException`.

3, Implement the client
======
The client program obtains a stub for the registry on the server's host, look up the remote object's stub by name in the registry, and then invokes the `sayHello` method on the remote object using the stub.

Here is the source code for the client:



	import java.rmi.registry.LocateRegistry;
	import java.rmi.registry.Registry;

	public class Client {

    private Client() {}

    public static void main(String[] args) {

        String host = (args.length < 1) ? null : args[0];
        try {
            Registry registry = LocateRegistry.getRegistry(host);
            Hello stub = (Hello) registry.lookup("Hello");
            String response = stub.sayHello();
            System.out.println("response: " + response);
        } catch (Exception e) {
            System.err.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}


This client first obtains the stub for the registry by invoking the static `LocateRegistry.getRegistry` method with the hostname specified on the command line. if no hostname is specified, then `null` is used as the hostname indicating that the localhost address should be used.

Next, the client invokes the remote method `lookup` on the registry stub to obtain the stub for the remote object form the server's registry.

Finally, the client invokes the `sayHello` mehtod on the remote object's stub, which causes the following actions to happen:
+ The client-side runtime opens a connection to the server using the host and port information in the remote object's stub and then serialized the call data.
+ The server-side runtime accepts the incoming call, dispatches the call to the remote object, and serialized the result(the reply string "Hello, world!") to the client
+ The client-side runtime receives, deserializes, and returns the result to the caller.

The response message returned form the remote invocation on the remote object is then printed to `System.out`.

hello[^hello]

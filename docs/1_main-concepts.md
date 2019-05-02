### Basic entities

Gkernel framework determines next main entities for request processing:

* Request - represents an HTTP request received by a server. Gkernel uses standard [net/http](https://golang.org/pkg/net/http/) Request type.
* Response - represents result of request processing, that should be sent to user. 
* Controller - function that receive request object and have to return response object.
* Event - some event that happened during request processing (or during life of the whole application).
This events are dispatched by framework.
* Event listener - function that receive and process events (dispatched by framework) and can affect request processing flow.


### Configuration

Gkernel uses yaml file to describe application configuration. Path to this file should be passed to Kernel constructor.
Config file describes application parameters (like port to listen on, environment etc.), routes, services and event listeners.


### Request lifecycle

Request processing cycle (also it can be called request lifecycle) has next main parts:

1. Framework receives request and performs routing to determine controller for request processing.
If required route not exists - framework creates `NotFoundHttpError` and dispatches `RuntimeError` event.
2. `RequestReceived` event is being dispatched. If listeners for that event returned response object - framework goes to p.5.
3. Controller starts. 
4. `RequestProcessed` event is being dispatched.
5. `ResponseBeforeSend` event is being dispatched.
6. Framework sends response to user.
7. `RequestTermination` event is being dispatched. Notice: `RequestTermination` is dispatched in separate goroutine 
in order not to delay response sending to user.
8. If some panic occurs during request processing framework recovers it and dispatches `RuntimeError`.


### Services and DI container

Service is a logical concept of some complete component or functionality block inside a program with a purpose that 
different clients can reuse it for different purposes.
 
Mostly all parts of application can be referenced as services. Controller is a service, event listener is a service etc.

As one of it's main parts Gkernel framework has DI container. It used for managing dependencies of application 
components (services) and performing dependency injection.
(If you are not familiar with Dependency injection concept, you can read [this article](https://en.wikipedia.org/wiki/Dependency_injection)).
As DI container [bassbeaver/gioc](https://github.com/bassbeaver/gioc) library is used.


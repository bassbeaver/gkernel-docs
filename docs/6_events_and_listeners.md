Gkernel is event-based framework, so Event and Listener are very important concepts in Gkernel's architecture.

Event is an object that represents some happening inside application during it's operation. Every event have to implement interface:
```go
type Event interface {
	StopPropagation()
	IsPropagationStopped() bool
}
```

To process Event Listener object is used. Listener is just a function with one argument, and type of that argument
have to correspond to Event's type. 

There can be multiple listeners for one event, this is called *Listeners Chain* and Event is *propagating* through that chain.
Order of Listeners in chain is determined by Listeners priority (the lower the priority first).
Event's propagation inside the Listeners Chain can be stopped by calling `StopPropagation()` method of the Event object.

Main way to register Listened is to declare it in Configuration file:
```yaml
IndexController:privatePage:
  url: "/private-page"
  methods: ["GET"]
  controller: "IndexController:PrivatePage"
  event_listeners:
    - {event: kernelEvent.RequestReceived, listener: "AuthService:RedirectToLoginIfNotAuthenticated", priority: 41}
```
This example shows registration of Listener for `RequestReceived` Event, with priority `41` and 
where Listener is method `RedirectToLoginIfNotAuthenticated` from `AuthService` Service.


#### Event types

Gkernel's events can be **application level** and **request level**.

##### Application level events

Application level events are used to manage request-processing flow.

There are next application level events:

###### ApplicationLaunched
Is being dispatched after application started, and configured but before it started to listen it's port. Can be used to establish DB connection, cache warming, etc.

###### ApplicationTermination
Is being dispatched during application shut down process. Can be used to close DB connections, exporting of some cached data, etc. 

&nbsp;

Application level events has next methods:
* `StopPropagation()` - to stop Event's propagation inside Listeners Chain	
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetContainer() *gioc.Container` - returns DI Container object used by Application.

&nbsp;

##### Request level events

There are next request level events:

###### RequestReceived
Is being dispatched after request was received by framework but before it was passed to controller. 
Can be used to read user's session from the storage, for authentification & authorization, etc.


`RequestReceived` Event has next methods: 
* `StopPropagation()` - to stop Event's propagation inside Listeners Chain	
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponse() response.Response` - gets Response object provided to this Event. Initially `RequestReceived` Event has no Response (method returns `nil`)
* `SetResponse(responseObj response.Response)` - stops Event's propagation and sends provided Response to user. 

###### RequestProcessed
Is being dispatched after Controller has processed Request. Contains Response object returned from Controller.

`RequestProcessed` Event has next methods: 
* `StopPropagation()` - to stop Event's propagation inside Listeners Chain	
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponse() response.Response` - gets Response object provided to this Event. Initially this will be Response returned by Controller
* `SetResponse(responseObj response.Response)` - sets Response object. Notice: unlike `RequestReceived.SetResponse()` this method does not stop Event's propagation

 
###### ResponseBeforeSend
Is being dispatched after `RequestProcessed` Event was processed. Can modify Response but can not replace it with new object.

`ResponseBeforeSend` Event has next methods: 
* `StopPropagation()` - to stop Event's propagation inside Listeners Chain	
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `GetResponse() response.Response` - gets Response object

###### RequestTermination
Is being dispatched after Response was sent to user. Can be used for logs exporting and others after-request activities.

`RequestTermination` Event has next methods:
* `StopPropagation()` - to stop Event's propagation inside Listeners Chain	
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `GetResponse() response.Response` - gets Response object


###### RuntimeError
In case of (panic)[https://blog.golang.org/defer-panic-and-recover] during Request processing 
Gkernel automatically recovers that panic, creates `RuntimeError` object to represent that panic and dispatches RuntimeError.

`RuntimeError` Event has next methods:
* `StopPropagation()` - to stop Event's propagation inside Listeners Chain	
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponse() response.Response` - gets Response object provided to this Event. Initially `RuntimeError` Event has no Response (method returns `nil`)
* `SetResponse(responseObj response.Response)` - stops Event's propagation and sends provided Response to user. 
* `GetError() *kernelError.RuntimeError` - returns `RuntimeError` object that represents recovered panic

Gkernel is event-based framework, so Event and Listener are very important concepts in Gkernel's architecture.

Event is an object that represents some happening inside application during it's operation. Every event have to implement interface:
```go
type Event interface {
	StopPropagation()
	IsPropagationStopped() bool
}
```

To process Event Listener object is used. Listener itself is method with one argument, and type of that argument
have to correspond to Event's type. Generally, Event Listener object have to be a Service with method of Listener type.

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

Gkernel's events can be **application level** and **request level** (web request or CLI command).

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

Application level events has next attributes:

* `Errors *[]error` - slice of errors occurred during application shutdown process

&nbsp;

##### Request level events

About request level events you can reed in corresponding sections: [web](/v1/5-4_web_events_and_listeners/) and [CLI]().

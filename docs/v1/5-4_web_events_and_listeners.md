There are next request level events dispatched by web Kernel of Gkernel framework:

###### RequestReceived
Is being dispatched after request was received by the framework but before it was passed to controller.
Can be used to read user's session from the storage, for authentication & authorization, etc.


`RequestReceived` Event has next methods:

* `StopPropagation()` - to stop Event's propagation inside Listeners Chain
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponseWriter() http.ResponseWriter` - returns ResponseWriter object associated with current Request
* `GetResponse() response.Response` - gets Response object provided to this Event. Initially `RequestReceived` Event has no Response (method returns `nil`)
* `SetResponse(responseObj response.Response)` - stops Event's propagation and sends provided Response to user.

&nbsp;
###### RequestProcessed
Is being dispatched after Controller has processed Request. Contains Response object returned from Controller.

`RequestProcessed` Event has next methods:

* `StopPropagation()` - to stop Event's propagation inside Listeners Chain
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponseWriter() http.ResponseWriter` - returns ResponseWriter object associated with current Request
* `GetResponse() response.Response` - gets Response object provided to this Event. Initially this will be Response returned by Controller
* `SetResponse(responseObj response.Response)` - sets Response object. Notice: unlike `RequestReceived.SetResponse()` this method does not stop Event's propagation

&nbsp;
###### ResponseBeforeSend
Is being dispatched after `RequestProcessed` Event was processed. Can modify Response but can not replace it with new object.

`ResponseBeforeSend` Event has next methods:

* `StopPropagation()` - to stop Event's propagation inside Listeners Chain
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponseWriter() http.ResponseWriter` - returns ResponseWriter object associated with current Request
* `GetResponse() response.Response` - gets Response object

&nbsp;
###### RequestTermination
Is being dispatched after Response was sent to user. Can be used for logs exporting and others after-request activities.

`RequestTermination` Event has next methods:

* `StopPropagation()` - to stop Event's propagation inside Listeners Chain
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `GetResponse() response.Response` - gets Response object

&nbsp;
###### RuntimeError
In case of [panic](https://blog.golang.org/defer-panic-and-recover) during Request processing
Gkernel automatically recovers that panic, creates `RuntimeError` object to represent that panic and dispatches RuntimeError.

`RuntimeError` Event has next methods:

* `StopPropagation()` - to stop Event's propagation inside Listeners Chain
* `IsPropagationStopped() bool` - returns if propagation of Event was stopped
* `GetRequest() *http.Request` - returns Request object
* `RequestContextAppend(key, val interface{})` - appends provided val object to Request's context
* `GetResponseWriter() http.ResponseWriter` - returns ResponseWriter object associated with current Request
* `GetResponse() response.Response` - gets Response object provided to this Event. Initially `RuntimeError` Event has no Response (method returns `nil`)
* `SetResponse(responseObj response.Response)` - stops Event's propagation and sends provided Response to user.
* `GetError() *kernelError.RuntimeError` - returns `RuntimeError` object that represents recovered panic

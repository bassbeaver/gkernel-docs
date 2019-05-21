HTTP web applications work in request-response mode and Gkernel uses certain objects to represent and work with that requests and responses.


#### Request

Gkernel uses standard [net/http](https://golang.org/pkg/net/http/) Request type to represent HTTP request received by a server.

&nbsp;
#### Response

Response represents result of request processing that should be sent to user.  Response can be of any type that
implements next interface:
```go
type Response interface {
	GetHttpStatus() int
	GetHeaders() http.Header
	GetBodyBytes() *bytes.Buffer
}
```

Gkernel framework provides bunch of ready to use Response types:

&nbsp;
##### BytesResponse

Represents simples response - slices of bytes. 

BytesResponse has next fields:

* `Body *bytes.Buffer` - stores response bytes, that should be sent to user. 

BytesResponse has next methods:

* `func (r *basicResponse) GetHeaders() http.Header`
* `func (r *basicResponse) HeaderSet(key, value string)` - intended to simplify adding new headers to response.
You can call 
```
r.GetHeaders().Set("MyHeader", "my header value")
``` 
or just 
```
r.HeaderSet("MyHeader", "my header value")
```
* `func (r *basicResponse) GetHttpStatus() int` - returns response's [HTTP status](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
* `func (r *basicResponse) SetHttpStatus(status int)` - sets response's HTTP status. 
* `func (r *BytesResponse) ClearBody()` - removes old Body bytes buffer, creates new and empty 
* `func (r *BytesResponse) GetBodyBytes() *bytes.Buffer` - returns a pointer to the Body bytes buffer

To create new BytesResponse you can use factory method `NewBytesResponse()`:
```go
import (
    "github.com/bassbeaver/gkernel/response"
)

responseObj := response.NewBytesResponse()
```

&nbsp;
##### BytesResponseWriter

Is a simple wrapper around `BytesResponse` to use in case when you need to use Gkernel's BytesResponse as `http.ResponseWriter`.

For example, adding [pprof](https://golang.org/pkg/net/http/pprof/) endpoints to Gkernel's routing:
```go
package main

import (
	"github.com/bassbeaver/gkernel"
	kernelResponse "github.com/bassbeaver/gkernel/response"
	"net/http"
	"net/http/pprof"
)

...

kernelObj, kernelError := gkernel.NewKernel("/path/to/config.yaml")
if nil != kernelError {
    panic(kernelError)
}

...

registerPprofRoute := func(name, url string, handlerFunc http.HandlerFunc) {
    kernelObj.RegisterRoute(&gkernel.Route{
        Name:    name,
        Url:     url,
        Methods: []string{http.MethodGet},
        Controller: func(request *http.Request) kernelResponse.Response {
            w := kernelResponse.NewBytesResponseWriter()
            handlerFunc(w, request)
            return w
        },
    })
}

registerPprofRoute("pprof:index", "/debug/pprof/", pprof.Index)
registerPprofRoute("pprof:cmdline", "/debug/pprof/cmdline", pprof.Cmdline)
registerPprofRoute("pprof:profile", "/debug/pprof/profile", pprof.Profile)
registerPprofRoute("pprof:symbol", "/debug/pprof/profile", pprof.Symbol)
registerPprofRoute("pprof:trace", "/debug/pprof/trace", pprof.Trace)
registerPprofRoute("pprof:heap", "/debug/pprof/heap", pprof.Index)
```

&nbsp;
##### RedirectResponse

Represents response that should be sent to useragent to perform redirect.
 
To create new BytesResponse you can use factory method:
```
NewRedirectResponse(request *http.Request, url string, httpStatus int) *RedirectResponse
```
Where:

* `request` is current Request object
* `url` - URL to redirect to
* `httpStatus` HTTP status to set for this response. As default status you can use `http.StatusSeeOther` `(303)` status.

&nbsp;
##### ViewResponse

You can use ViewResponse when you need to return rendered template (usually html template).

ViewResponse has next methods:

* `func (r *basicResponse) GetHeaders() http.Header`
* `func (r *basicResponse) HeaderSet(key, value string)` - sets request's header value
* `func (r *basicResponse) GetHttpStatus() int` - returns response's [HTTP status](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
* `func (r *basicResponse) SetHttpStatus(status int)` - sets response's HTTP status. 
* `func (r *BytesResponse) GetBodyBytes() *bytes.Buffer` - returns a pointer to the Body bytes buffer
* `func (r *ViewResponse) SetData(data interface{})` - sets template's data (variables used for templates rendering)
* `func (r *ViewResponse) SetTemplate(tpl *template.Template)`- is used with Gkernel and usually you do not need this method.

In most cases, to return rendered template (view) as a Response you will be enough to use factory method: 
```
NewViewResponse(templateName string) *ViewResponse
``` 
Example (from [bassbeaver/gkernel-skeleton IndexController](https://github.com/bassbeaver/gkernel-skeleton/blob/master/controller/IndexController.go#L15)):

```go
func (c *IndexController) PageWithParam(request *http.Request) kernelResponse.Response {
	var viewData struct {
		CsrfToken string
		User      auth.UserInterface
		Header    struct {
			Title string
		}
		H1    string
		Param string
	}

	viewData.Header.Title = "gkernel skeleton site"
	viewData.H1 = "This is page with URL parameter"
	viewData.CsrfToken = csrfService.GetTokenFromRequestContext(request)
	if auth.GetUser(request) != nil {
		viewData.User = auth.GetUser(request).(*user_provider.UserStub)
	}

	viewData.Param = request.URL.Query().Get(":parameterValue")

	response := kernelResponse.NewViewResponse("index/page-with-param.gohtml")
	response.SetData(viewData)

	return response
}
```
**Notice:** `templateName` from `NewViewResponse(templateName string) *ViewResponse` is relative path to template file.
Full path to template file Gkernel determines according to the following algorithm:
```
<path to application Config file>/<templates_path>/<templateName>
```
Where:

* `<path to application Config file>` - path to application's configuration file
* `<templates_path>` - value application's configuration file
* `<templateName>` - template's name from `NewViewResponse(templateName string)` factory method

&nbsp;
##### JsonResponse

Represents json encoded response.

JsonResponse has next fields:

* `Body interface{}` - response value, that should be json encoded and sent to user 

JsonResponse has next methods:

* `func (r *basicResponse) GetHeaders() http.Header`
* `func (r *basicResponse) HeaderSet(key, value string)` - sets request's header value
* `func (r *basicResponse) GetHttpStatus() int` - returns response's [HTTP status](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
* `func (r *basicResponse) SetHttpStatus(status int)` - sets response's HTTP status. 
* `func (r *BytesResponse) GetBodyBytes() *bytes.Buffer` - returns a pointer to the Body bytes buffer

To create new JsonResponse you can use factory method: 
```
NewJsonResponse() *JsonResponse
```
this factory method sets 
`Content-Type` header to `application/json` for created Response object.

Also, Gkernel provides factory method to create [Json Api](https://jsonapi.org/) Response: 
```
NewJsonApiResponse() *JsonResponse
``` 
It is similar to `NewJsonResponse()` except that sets `Content-Type` header to `application/json`.

&nbsp;
##### WebsocketUpgradeResponse

Represents Response used to upgrade protocol to WebSocket.

To create new WebsocketUpgradeResponse you can use factory method 
```
NewWebsocketUpgradeResponse(upgrader *websocket.Upgrader, controller WebSocketController) *WebsocketUpgradeResponse
```

Where:

* `upgrader` - instance of `gorilla/websocket.Upgrader`
* `controller` - function with signature: `func(*websocket.Conn)`

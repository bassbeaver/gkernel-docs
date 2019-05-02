#### Routing

Route determines connection between URL and Controller, designed to process requests to that URL.

To represent Route in code, Gkernel provides `Route` type. `Route` has next fields:
* Name, `string` - name of the route
* Methods, `[]string` - array of HTTP method names. Route will only process requests made using that methods.
* Url, `string`
* Controller, `Controller` - Controller to process request

Main way to create routes in Gkernel application is to describe routes in Configuration file. This way is described in 
**Configuration** section of this documentation.

But also, you can register routes within the code. Gkernel provides `func (k *Kernel) RegisterRoute(route *Route) *Kernel`
method for this.

##### Route URL parameters

Sometimes you will need to capture segments of the URI within your route. To achieve this you have to declare rote with parameter in URL.
Configuration file example for this case:
```yaml
IndexController:pageWithParam:
  url: "/page-with-param/:parameterValue"
  methods: ["GET"]
  controller: "IndexController:PageWithParam"
```
Where declared parameter is `:parameterValue`.

Captured parameters live in Request object. To obtain captured parameter inside controller you have to:
```go
import (
	"github.com/bassbeaver/gkernel/response"
	"net/http"
)

func (c *IndexController) PageWithParam(request *http.Request) kernelResponse.Response {
    ...
    
	urlParam := request.URL.Query().Get(":parameterValue")

    ...
}
```


#### Controller

Basically Controller is function that receive request object and have to return response object.
Controller function should have next signature: 
```go
import (
	"github.com/bassbeaver/gkernel/response"
	"net/http"
)

func(*http.Request) response.Response
``` 

Gkernel's approach to Controllers is to register Service where some methods have appropriate (Controller's) signature. 
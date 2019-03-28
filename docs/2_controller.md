### Response

Response represents result of request processing, that should be sent to user.  Basically response can be of any type that
implements next interface:
```go
type Response interface {
	GetHttpStatus() int
	GetHeaders() http.Header
	GetBodyBytes() *bytes.Buffer
}
```

Gkernel framework provides bunch of ready to use Response types:


### Controller

Basically Controller is function that receive request object and have to return response object.
Controller function should have next signature: 
```go
import (
	"github.com/bassbeaver/gkernel/response"
	"net/http"
)

func(*http.Request) response.Response
``` 


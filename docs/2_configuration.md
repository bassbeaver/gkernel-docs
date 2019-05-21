Gkernel uses yaml file to describe application configuration. Path to this file should be passed to Kernel constructor.
Config file describes application parameters (like port to listen on, environment etc.), routes, services and event listeners.

Simple example:
```go
import (
	"github.com/bassbeaver/gkernel"
)

...

var configPath string
configPath = "/path/to/config.yaml"
kernelObj, kernelError := gkernel.NewKernel(configPath)
``` 

It's a good idea to get this path as program argument. This approach is used in [gkernel skeleton application](https://github.com/bassbeaver/gkernel-skeleton):
```go
import (
	"flag"
	"fmt"
	"github.com/bassbeaver/gkernel"
	kernelResponse "github.com/bassbeaver/gkernel/response"
	"net/http"
	"net/http/pprof"
	"os"
)

...

// --- Processing application's arguments and flags
flags := flag.NewFlagSet("flags", flag.PanicOnError)
configPathFlag := flags.String("config", "", "Path to config file")
flagsErr := flags.Parse(os.Args[1:])
if nil != flagsErr {
    panic(flagsErr)
}

// --- Creating application Kernel
var configPath string
if "" == *configPathFlag {
    curBinDir, curBinDirError := os.Getwd()
    if curBinDirError != nil {
        fmt.Println("Failed to determine path to own binary file")
        panic(curBinDirError)
    }
    configPath = curBinDir + "/config.yaml"
} else {
    configPath = *configPathFlag
}

kernelObj, kernelError := gkernel.NewKernel(configPath)
if nil != kernelError {
    panic(kernelError)
}
```
Code above searches for program argument `config` passed to application during its start, if no argument found it tries to found `config.yaml` file.
in its working directory and panics if failed to locate config file path.

For example, if you build you app binary to `/etc/gkernel-app` and put your config file to `/configs/gkernel-app-config.yaml`  it could be like:
```bash
/etc/gkernel-app -config /configs/gkernel-app-config.yaml
```

### Configuration file overview
Configuration file has next root keys:

* http_port
* app_env &nbsp;
* templates_path &nbsp;
* services &nbsp;
* routing &nbsp;
* event_listeners &nbsp;
* parameters &nbsp;

#### http_port
Determines on what port application would be listening for requests.
 
#### app_env
Determines current application's environment. This is just a flag thet signals to application's code in what regime it is running.
Typically application can have two environments: `dev` and `prod`, but you are free to use as many environments as you want.

#### templates_path
Gkernel uses go [html/template](https://golang.org/pkg/html/template/) package as a template engine to render pages.
This templates are parsed during Kernel startup and `templates_path` key determines where Kernel should search for your templates.

#### services
`services` block describes all services in the application. Reading this block Kernel learns what services should be registered in
DI container.

Each service should have alias and arguments.  
Example:
```yaml
services:

  Sessions:
    arguments: ["sid", "@SessionsRedisConnection"]
```
Where `Sessions` are service alias and 
```yaml
arguments: ["sid", "@SessionsRedisConnection"]
``` 
indicates that `Sessions` service factory requires two arguments and
first argument is just simple string with value `sid` and second argument should be a pointer to the `SessionsRedisConnection` 
service (this `SessionsRedisConnection` service have also to be described in config file).

I want to bring your attention to that config file **only describes** services and service-factories arguments, and you have to register
that factories in your application code.

For more information adout services and DI container see "Services and DI container" section.

#### routing
This block describes routes provided by application. A route is a map from a URL path to the program logic, 
designed to process requests to that URL (we call that logic - Controller).

Routing block has next sub-blocs:
* routes - list of available routes
* event_listeners - request-level event listeners common for all routes

Route description looks like:
```yaml
IndexController:privatePage:
  url: "/private-page"
  methods: ["GET"]
  controller: "IndexController:PrivatePage"
  event_listeners:
    - {event: kernelEvent.RequestReceived, listener: "AuthService:RedirectToLoginIfNotAuthenticated", priority: 41}
``` 
Where:

* `IndexController:privatePage` - route name
* `url` - url or the route, url can contain parameters: `"/some-page/:param1"`
* `methods` - list of HTTP methods allowed for this route
* `controller` - controller to process request matched to this route. `IndexController` is the service alias in DI container and `PrivatePage` is the name of method to be called to process request.
* `event_listeners` - list of request-level event listeners for this route, for more information see "Events and listeners" section.

Whole `routing` block can look like:
```yaml
routing:

  routes:

    IndexController:index:
      url: "/"
      methods: ["GET"]
      controller: "IndexController:Index"

    IndexController:loginPage:
      url: "/login"
      methods: ["GET"]
      controller: "IndexController:LoginPage"
      event_listeners:
        - {event: kernelEvent.RequestReceived, listener: "AuthService:RedirectIfAuthenticated", priority: 41}

  event_listeners:
    - {event: kernelEvent.RequestReceived, listener: "RequestLoggerSetter:CreateLogger", priority: 15}
    - {event: kernelEvent.RequestTermination, listener: "RequestLoggerSetter:CloseLogger", priority: 100}
```
We can see here two routes `IndexController:index` and `IndexController:loginPage`. Routes have two common event listeners (`RequestLoggerSetter:CreateLogger`, `RequestLoggerSetter:CloseLogger`),
this listeners will run at every request.

Also `IndexController:loginPage` has `AuthService:RedirectIfAuthenticated` listener, this listener will run only during `GET /login` request.

 
#### event_listeners

This block describes application level event listeners. Can look like:
```yaml
event_listeners:
  - {event: kernelEvent.ApplicationLaunched, listener: "SessionsMiddleware:InitRedisConnection", priority: 10}
  - {event: kernelEvent.ApplicationTermination, listener: "SessionsMiddleware:CloseRedisConnection", priority: 10}
```
For more information see "Events and listeners" section.


#### parameters

This block is simple key-value storage for some application parameters that can change in different environments or run cases. 

For example, you need some timeout value for some process and you want this timeout be 30 sec on **prod** server and 60 sec on **dev** server.
You need to pass this value to corresponding service (in `services` configuration block).
In such case you can define `process_timeout` parameter, use it by name as service-factory parameter in `services` section and it's will be stored in `parameters` section.
So for different servers you should only change `parameters` section of config file.

Example part of config file for **prod** server:
```yaml
services:

  SomeService:
    arguments: ["#timeout_value"]
    
parameters:
  timeout_value: 30
```  

Example part of config file for **dev** server:
```yaml
services:

  SomeService:
    arguments: ["#timeout_value"]
    
parameters:
  timeout_value: 60
```  
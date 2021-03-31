Gkernel uses yaml files to describe application configuration. Path to folder containing this files should be passed to Kernel constructor.
Config files describe application parameters (like port to listen on), web routes, CLI commands, services and event listeners.

Simple example:
```go
import (
	webKernel "github.com/bassbeaver/gkernel/web"
)

...

var configPath string
configPath = "/path/to/config"
kernelObj, kernelError := webKernel.NewKernel(configPath)
``` 

It's a good idea to get this path as program argument. This approach is used in [gkernel skeleton application](https://github.com/bassbeaver/gkernel-skeleton):
```go
import (
	"flag"
	"fmt"
	webKernel "github.com/bassbeaver/gkernel/web"
	kernelResponse "github.com/bassbeaver/gkernel/web/response"
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
    configPath = curBinDir + "/config"
} else {
    configPath = *configPathFlag
}

kernelObj, kernelError := webKernel.NewKernel(configPath)
if nil != kernelError {
    panic(kernelError)
}
```
Code above searches for program argument `config` passed to application during its start, if no argument found it tries 
to find `config` directory in its working directory and panics if failed to locate config files path.

For example, if you build you app binary to `/etc/gkernel-app` and put your config files to `/configs/gkernel-app-config`  it could be like:
```bash
/etc/gkernel-app -config /configs/gkernel-app-config
```

### Configuration file overview
Configuration file has next root keys:

* services
* web
* cli
* event_listeners
* parameters

#### services
`services` block describes all services in the application. Reading this block Kernel learns what services should be registered in
DI container.

Each service should have alias and arguments.  
Example:
```yaml
services:

  SessionsMiddleware:
    arguments: ["sid", "@RedisConnection"]
```
Where `SessionsMiddleware` are service alias and 
```yaml
arguments: ["sid", "@RedisConnection"]
``` 
indicates that `SessionsMiddleware` service factory requires two arguments and
first argument is just simple string with value `sid` and second argument should be a pointer to the `RedisConnection` 
service (this `RedisConnection` service have also to be described in config file).

I want to bring your attention to that config file **only describes** services and service-factories arguments, and you have to register
that factories in your application code.

For more information about services and DI container see "Services and DI container" section.

#### web
This block describes all configuration related to web part of application (and used by web Kernel).

Web block has next keys:
* **http_port**  
  Determines on what port application would be listening for HTTP requests.
* **templates_path**  
  Web Kernel of Gkernel uses go [html/template](https://golang.org/pkg/html/template/) package as a template engine to render pages.
  This templates are parsed during Kernel startup and `web.templates_path` key determines where Kernel should search for your templates.
* **shutdown_timeout**  
  Graceful shutdown timeout for http server in milliseconds, if not set default value of 500 will be used.
* **routing**  
  Routing key describes routes provided by application. More detailed description will be further.

###### web.routing
This sub block describes web routes provided by application. A route is a map from a URL path to the program logic, 
designed to process requests to that URL (we call that logic - Controller).

Routing block has next sub-blocks:
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
* `controller` - controller to process request matched to this route. `IndexController` is the service alias in DI container and `PrivatePage`
  is the method name of `IndexController` service to be called to process request.
* `event_listeners` - list of request-level event listeners for this route, for more information see "Events and listeners" section.

Whole `web` block can look like:
```yaml
web:
  http_port: 8081
  templates_path: "web/templates"
  shutdown_timeout: 5000

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
      - {event: kernelEvent.RequestReceived, listener: "RequestLoggerSetter:SetLoggerToRequestContext", priority: 15}
      - {event: kernelEvent.RequestTermination, listener: "RequestLoggerSetter:CloseLogger", priority: 100}
```
We can see here two routes `IndexController:index` and `IndexController:loginPage`. Routes have two common event listeners
(`RequestLoggerSetter:SetLoggerToRequestContext`, `RequestLoggerSetter:CloseLogger`), these listeners will run at every request.

Also `IndexController:loginPage` has `AuthService:RedirectIfAuthenticated` listener, this listener will run only during `GET /login` request.


### cli

This block describes console commands provided by application. A command is a map from CLI arguments to the program logic,
designed to process such CLI requests.

CLI block has next sub-blocks:
* commands - list of available commands

Command description looks like:
```yaml
CliController:command1:
  name: command1
  controller: "CliController:Command1"
  help: "first cli command"
```

Where:

* `CliController:command1` - command alias (not used by Gkernel)
* `name` - name of command. Gkernel search this name in CLI arguments passed to program
* `controller` - controller to process command. `CliController` is the service alias in DI container and `Command1`
  is the method name of `CliController` service to be called to process command.
* `help` - string for full command description shown when running the command with the `--help` option.
  Also, `help` strings of all commands CLI Kernel returns by `Kernel.GetHelp()` or `Kernel.FormatHelp()` methods.

Whole `cli` block can look like:
```yaml
cli:

  commands:

    CliController:command1:
      name: command1
      controller: "CliController:Command1"
      help: "first cli command"
```

#### event_listeners

This block describes application level event listeners. Can look like:
```yaml
event_listeners:
  - {event: kernelEvent.ApplicationLaunched, listener: "RedisConnectionMiddleware:InitRedisConnection", priority: 10}
  - {event: kernelEvent.ApplicationTermination, listener: "RedisConnectionMiddleware:CloseRedisConnection", priority: 10}
```
For more information see "Events and listeners" section.


#### parameters

This block is simple key-value storage for some application parameters that can change in different environments or run cases. 

For example, you need some timeout value for some process and you want this timeout be 30 sec on **prod** server and 60 sec on **dev** server.
You need to pass this value to corresponding service (in `services` configuration block).
In such case you can define `timeout_value` parameter, use it by name as service-factory parameter in `services` section and it's will be stored in `parameters` section.
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
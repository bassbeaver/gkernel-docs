Gkernel framework provides opportunities to configure HTTP server timeouts (ReadHeaderTimeout, ReadTimeout, WriteTimeout, IdleTimeout)
and also to configure per-route timeouts.

###### HTTP server timeouts

HTTP server configuration is done by next config expressions:
```yaml
web:

  server_read_header_timeout: 2500
  server_read_timeout: 7500
  server_write_timeout: 15000
  server_idle_timeout: 7500
```
Timeout values are in milliseconds


###### Route timeouts

As you know, server timeouts (ReadHeaderTimeout, ReadTimeout, WriteTimeout, IdleTimeout) mostly protects you from slow or malicious client,
but server timeouts can not help you against your own slow code. If you have `server_write_timeout` is set to 15 seconds but your controller
takes it up to 30 seconds for request processing client will wait 30 seconds and will get connection closed only when controller ends with processing.
This is how server timeouts working.

For time limiting of request processing we have to use route timeouts. When perroute timeout expires Gkernel framework stops
waiting for request processing result and send timeout response to client.

To configure route timeout next config expression is used: 
```yaml
web:
  routing:
    routes:
      RouteName:
        timeout:
          duration: 10000
          handler: "TimeoutHandler:PerformLoginHandleTimeout"
```
Where `timeout.duration` is timeout in milliseconds and `handler` is handler function used to return timeout response.
Basically timeout handler is function with signature:
```go
import (
    kernelResponse "github.com/bassbeaver/gkernel/web/response"
)

func() kernelResponse.Response
```
Gkernel's approach to Timeout Handlers is to register Service where some method have appropriate (Timeout Handler's) signature.
In configuration `RouteName.timeout.handler` clause value "TimeoutHandler:PerformLoginHandleTimeout" determines service (`TimeoutHandler`) and method of this service (`PerformLoginHandleTimeout`).

Example of full config of route looks like: 
```yaml
web:
  routing:
    routes:
      IndexController:performLogin:
        url: "/login-perform"
        methods: ["POST"]
        controller: "IndexController:PerformLoginLogout"
        event_listeners:
          - {event: kernelEvent.RequestReceived, listener: "AuthService:RedirectIfAuthenticated", priority: 41}
          - {event: kernelEvent.RequestReceived, listener: "AuthService:AuthenticateByLogPass", priority: 42}
        timeout:
          duration: 10000
          handler: "TimeoutHandler:PerformLoginHandleTimeout"
```

###### Common route timeouts

In section above we described how to set route timeout processing in route configuration but often it is not convenient to use
if we want to use same timeout processing for multiple routes. For this case Gkernel framework provides common route timeout configuration.
This common configuration applies to all routes which do not have "personal" configuration.
Common route timeout config looks like:
```yaml
web:
  routing:
    timeout:
      duration: 8000
      handler: "TimeoutHandler:HandleTimeout"
```

Also, full example of timeouts configuration and usage you can see at [gkernel skeleton application](https://github.com/bassbeaver/gkernel-skeleton).
### Basic entities

Main idea of Gkernel framework is to organize request processing flow which is:
 
* controlled by events
* processed by services, and services are managed by Service Container

Framework determines next main entities for request processing:

* Kernel - represents whole piece of framework logic that have to be run in some invocation environment (web, CLI) 
* Controller - function that receive incoming signals (HTTP request object, CLI command) and have to return outcome signals 
  (HTTP response object, CLI command processing error etc.).
* Event - some event that happened during application life, HTTP request processing, or CLI command processing.
  These events are dispatched by the framework.
* Event listener - function that receive and process events (dispatched by the framework) and can affect incoming signals processing flow.
* HTTP Request - represents an HTTP request received by a server. Gkernel uses standard [net/http](https://golang.org/pkg/net/http/) Request type.
* HTTP Response - represents result of request processing, that should be sent to user. 
* CLI Command - represents CLI command to be executed.


### Kernel

Gkernel framework have two kernel implementation - for web and for CLI environments.

`github.com/bassbeaver/gkernel/web`
package contains kernel and other entities for web environment.  

`github.com/bassbeaver/gkernel/cli`
package contains kernel and other entities for CLI environment.


### Configuration

Gkernel uses yaml files to describe application configuration. Path to folder, containing config files, should be passed to Kernel constructor.
Config files describe application parameters (like port to listen on, environment etc.), routes, services and event listeners.

Under the hood to work with config files Gkernel uses [viper](https://github.com/spf13/viper) library. Viper can read 
**json**, **toml**, **yaml**, **yml**, **properties**, **props**, **prop**, **hcl** files, so, in fact, any of that file
types can be used as configuration files for Gkernel, but we suggest to use **yaml**.

While reading config path (directory) Gkernel reads any allowed (that Viper can read) file from that directory and treats
it like configuration file. Therefore it is recommended to store config files in separate folder and not mix config files with other files to avoid erroneous read
of non-config file.  


### Services and DI container

Service is a logical concept of some complete component or functionality block inside a program with a purpose that 
different clients can reuse it for different purposes.
 
Mostly all parts of application can be referenced as services. Controller is a service, event listener is a service, DB connection is a service, etc.

As one of it's main parts Gkernel framework has DI container. It used for managing dependencies of application 
components (services) and performing dependency injection.
(If you are not familiar with Dependency injection concept, you can read [this article](https://en.wikipedia.org/wiki/Dependency_injection)).
[bassbeaver/gioc](https://github.com/bassbeaver/gioc) library is used as DI container.

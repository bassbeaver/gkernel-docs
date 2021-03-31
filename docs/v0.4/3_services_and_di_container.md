Gkernel uses [bassbeaver/gioc](https://github.com/bassbeaver/gioc) as DI container. 
DI container is used to create entities during application run.
Let's call that entities as **Service**. 

To create some Service you have to know what dependencies that service has, and how to build that new Service.
*"Dependency"* - is some object (scalar value, pointer to other service etc.) that is required for Service to operate.
*"How to build"* - is set of operations needed to create and configure Service.

It's good approach to encapsulate knowledge *"how to build"* in object called **Factory** 
(for more information see [Factory pattern](https://en.wikipedia.org/wiki/Factory_method_pattern)). 
 

#### Factory

With [bassbeaver/gioc](https://github.com/bassbeaver/gioc) Factory can be one of:

1. Function. This function must return only one parameter and this parameter must be a pointer to new Service instance.
2. Factory struct from `bassbeaver/gioc` package.
Factory struct has next definition: 
```go
type Factory struct {
    Arguments []string
    Create    interface{}
}
```  

Where :

* `Create` must be a function which knows how to create service (requirements to this function are same as for function from p.1).
* `Arguments` is an array of definitions for `Create` function arguments. N-th element of `Arguments` array is for N-th argument of `Create` function.

But with Gkernel Container usage is simplified - Gkernel provides method: 
```go
func (k *Kernel) RegisterService(alias string, factoryMethod interface{}, enableCaching bool) error
``` 
Where:

1. `alias` - Service alias (must correspond to Service alias from Config's `services` block)
2. `factoryMethod` - function, which knows *how to create* Service. 
3. `enableCaching` - flag signaling the Container if container have to create this Service once and save pointer to it in cache,
and for next requests for this Service just get it from cache, or Container have to honestly create new instance of service for each request. 

`factoryMethod` can have arguments, required for Service building. Gkernel reads description of that arguments from Config file
from `arguments` key of Service definition.

Each argument definition is a string and is interpreted in next ways:

* If first symbol of this string is `@` - this definition is interpreted as service alias, so Container will
try to find service with that alias.
* If first symbol of this string is `#` - this definition is interpreted as parameter name, so Container will
  try to find value of that parameter in Container's parameters bag.
* In other cases definition string is interpreted as value for corresponding argument of `Create` function.

Container tries to cast value of each `argument` to required type, if cast failed - Container panics.

For this moment Gkernel can not perform Service registration by himself (because it [has paws](http://memesmix.net/media/created/s3uwfl.jpg)) 
and you have to call `RegisterService` manually, during application startup.


#### Container usage. Service registration and service obtainment

Simple example of Container usage.

At first let's describe Services in Config file: 
```yaml
...
services:

  UserProvider:
    arguments: []

  AuthService:
    arguments: ["@UserProvider", "/login", "/"]
```

Next, let's write Service and Service Factory code: 
```go
import (
	"fmt"
	"github.com/bassbeaver/gkernel"
	"gkernel-skeleton/service/auth"
)


const (
	UserProviderMiddlewareServiceAlias = "UserProvider"
	AuthServiceAlias = "AuthService"
)


type UserProvider struct {
	// UserProvider knows how to get users data from storage, but we will omit that functionality here
}

func newUserProvider() *UserProvider {
	return &UserProvider{}
}

func RegisterUserProvider(kernelObj *gkernel.Kernel) {
	err := kernelObj.RegisterService(UserProviderMiddlewareServiceAlias, newUserProvider, true)
	if nil != err {
		panic(fmt.Sprintf("failed to register %s service, error: %s", UserProviderMiddlewareServiceAlias, err.Error()))
	}
}


type AuthService struct {
	userProvider *UserProvider
	loginPageUrl string
	fallbackUrl  string
}

func newAuthService(userLoader *UserProvider, loginPageUrl, fallbackUrl string) *AuthService {
	return &Service{
		userProvider: userLoader,
		loginPageUrl: loginPageUrl,
		fallbackUrl:  fallbackUrl,
	}
}

func RegisterAuthService(kernelObj *gkernel.Kernel) {
	err := kernelObj.RegisterService(AuthServiceAlias, newAuthService, true)
	if nil != err {
		panic(fmt.Sprintf("failed to register %s service, error: %s", AuthServiceAlias, err.Error()))
	}
}
```

Finally, lets register Services in Container:
```go

kernelObj, kernelError := gkernel.NewKernel("/path/to/config")
if nil != kernelError {
    panic(kernelError)
}

RegisterAuthService(kernelObj)
RegisterUserProvider(kernelObj)
```

After that, Container can create Services (on demand) and inside of created `AuthService` already created `UserProvider` will be available.

You can ask me *"and what about Service obtainment?"*. In real application most part of time you will be working with
Controllers and Event Listeners and both of them should be registered in Container as Services (if you want have good application architecture)
and in that case you do not need explicit Service obtainment, Gkernel and Container will do it for you.

But, if for some reason you really want to get some Service explicitly, you can use: 
```go
kernelObj.GetContainer().GetByAlias("service_alias")
```

**Note:** in our example we called `RegisterAuthService` before `RegisterUserProvider` and you can ask: 
*"Why before? AuthService is dependent from UserProvider, that code will fail."*. 

Answer is: you can register services in any order
because no services are created during registration process. Services are created by Container only when they are really needed.


For more examples you can see [Gkernel skeleton application](https://github.com/bassbeaver/gkernel-skeleton/blob/master/main.go#L63).
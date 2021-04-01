### Command

Gkernel provides `Command` type to represents incoming CLI command of the program. `Command` has next fields:

* Name, `string` - name of Command.
* Controller, `Controller` - Controller to process command
* Help, `string` - string for full command description shown when running the command with the `--help` option.

`help` strings of all commands registered in CLI Kernel are returned by `Kernel.Help()` method.

&nbsp;
#### Controller

Controller is function that receives incoming program CLI arguments and have to return `CliError`
on failure or `nil` in case of success. Controller function should have next signature:
```go
import (
	cliKernelError "github.com/bassbeaver/gkernel/cli/error"
)

func(args []string) cliKernelError.CliError
``` 

Gkernel's approach to Controllers is to register Service where some methods have appropriate (Controller's) signature.
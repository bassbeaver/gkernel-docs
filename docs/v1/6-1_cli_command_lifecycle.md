### CLI command lifecycle

"Incoming request" for CLI environment we suggest to call Command. Command has simpler lifecycle then web and looks like:

1. Framework parses incoming CLI arguments of program and determines the Command corresponding to these arguments.
2. Command contains Controller to process this command. Framework starts this Controller and passes incoming CLI
arguments to Controller.
3. Controller completes request successfully or return error in case of failure.

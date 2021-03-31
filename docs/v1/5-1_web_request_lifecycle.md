### Web Request lifecycle

Request processing cycle (also it can be called request lifecycle) has next main parts:

1. Framework receives request and performs routing to determine controller for request processing.
   If required route not exists - framework creates `NotFoundHttpError` and dispatches `RuntimeError` event.
2. `RequestReceived` event is being dispatched. If listeners for that event returned response object - framework goes to p.5.
3. Controller starts.
4. `RequestProcessed` event is being dispatched.
5. `ResponseBeforeSend` event is being dispatched.
6. Framework sends response to user.
7. `RequestTermination` event is being dispatched. Notice: `RequestTermination` is dispatched in separate goroutine
   in order not to delay response sending to user.
8. If some panic occurs during request processing framework recovers it and dispatches `RuntimeError`.

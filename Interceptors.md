This is just a quick description of the differences between the Java-Node
approach to gRPC interceptors and the Go approach and the advantages of the
Java-Node approach.

First, some background: the grpc libraries are built on top of a common core C
library. They interact with the C library by packaging up operations and
sending them over through functions like
[start_batch](https://github.com/grpc/grpc/blob/master/include/grpc/grpc.h#L231).
The Java-Node approach implements interception at the point were operations are
sent to the C library. An interceptor user would write a class like

```javascript
    var interceptor = {
        start: function(metadata, listener, next) {
            var newListener = {
                onReceiveMetadata: function(metadata, next) {
                    next(metadata);
                },
                onReceiveMessage: function(message, next) {
                    next(message);
                },
                onReceiveStatus: function(status, next) {
                    next(status);
                }
            };
            next(metadata, newListener);
        },
        sendMessage: function(message, next) {
            next(messasge);
        },
        halfClose: function(next) {
            next();
        },
        cancel: function(message, next) {
            next();
        }
    };
```

(taken from the [Node
proposal](https://github.com/drobertduke/proposal/blob/6a01c9a32cc109e8b1d50b780aae3a1ba4b56bc8/L5-NODEJS-CLIENT-INTERCEPTORS.md#simple))
where the methods on the class [correspond](
https://github.com/drobertduke/proposal/blob/6a01c9a32cc109e8b1d50b780aae3a1ba4b56bc8/L5-NODEJS-CLIENT-INTERCEPTORS.md#grpc-operations)
to the [operation types](
https://github.com/grpc/grpc/blob/master/include/grpc/impl/codegen/grpc_types.h#L426)
defined by the C library.

This is opposed to the Go approach (and Python version I wrote) that implments
interception on top of the interfaces that the Go-Python libraries expose to
the user. The main advantage of the Java-Node approach is that all the
different RPC types (unary, steaming, async, etc) can be treated in an
analogous way without having to write special cases (see [also](
https://github.com/drobertduke/proposal/blob/c5ee5592706615285c9cf5d6114e5e4670a8d59b/NODEJS-CLIENT-INTERCEPTORS.md#abstraction-level)).
For example, in the OpenTracing implementation for Python there has to be code
that
[checks](https://github.com/rnburn/grpc-opentracing-1/blob/master/python/grpc_opentracing/_client.py#L104)
whether the invoker is making an async RPC and then has to [customize](
https://github.com/rnburn/grpc-opentracing-1/blob/master/python/grpc_opentracing/_client.py#L52)
the future returned if so, even though the sync and async RPCs both send the
same operations over to the C library, so there's nothing really to distinguish
them from the C library's perspective.

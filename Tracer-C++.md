These are the minimal changes I'd recommend making to the OpenTracing API to
make it usable for LightStep's tracer. I assume the goal is to keep the
OpenTracing API as a C++-98 library and split out the LightStep tracer into
a C++11 BasicTracer library and LightStep specific library.
1. *Don't lose the type when setting tags.* The current API
   [converts](https://github.com/opentracing/opentracing-cpp/blob/master/opentracing/span.h#L190)
   all tag values to a string. This could be a problem for the standardized
   non-string tags such as `error` and `http.status_code`. LightStep's
   implementation solves this by using a [variant
   library](https://github.com/mapbox/variant), though that library requires
   C++11. Something similar could be done for the OpenTracing API, except using
   a C++98 variant (perhaps this
   [one](https://github.com/martinmoene/variant-lite) would work). Given how
   common type erasure is in the OpenTracing APIs (also used for logging), this
   would probably be
   the better long-term solution, but if the goal is to make more minimal changes
   initially it could be accomplished by just adding overloads for each of the tag
   value types.  (Also proposed as part of this
   [PR](https://github.com/jquinn47/opentracing-cpp/blob/1b915dabcdb3c93ca8f2db71ae1efc4350431c8a/opentracing/span.h#L20)).
2. *Support followsFrom relations and multiple span references.* Only a
   single childOf relation can currently be
   [specified](https://github.com/opentracing/opentracing-cpp/blob/master/opentracing/tracer.h#L32)
   when starting a span. I recommend modify the SpanOptions class to take a vector
   of span contexts along with an enumeration value to denote the reference type.

I can think of other modifications that should be made. Using the same class to
represent both a Span and SpanContext is problematic and the [logging
API](https://github.com/opentracing/opentracing-cpp/blob/master/opentracing/span.h#L122)
needs to be rewritten to allow arbitrary key-value pairs, but those could be put
off to a later version. Since LightStep's tracer doesn't implement
[logging](https://github.com/lightstep/lightstep-tracer-cpp/blob/master/src/c%2B%2B11/lightstep/span.h#L51)
yet, you could just not implement the logging portion of the OpenTracing API
without losing functionality from the existing tracer.

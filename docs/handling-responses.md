
# Response Handlers

Response handlers provide a standard way of handling the final response
following a successful request to the API.  These must inherit from
`BaseResponseHandler` and implement the `get_request_data()` method which
will take the `requests.Response` object and parse the data accordingly.

The apiclient supports the following response handlers, by specifying
the class on initialization of the client as follows:

The response handler can be omitted, in which case no formatting is applied to the
outgoing data.

```python
client = ClientImplementation(
   authentication_method=...,
   response_handler=<ResponseHandlerClass>,
   request_formatter=...,
)
```

## `RequestsResponseHandler`

Handler that simply returns the original `Response` object with no
alteration.

Example:

```python
client = ClientImplementation(
    authentication_method=...,
    response_handler=RequestsResponseHandler,
    request_formatter=...,
)
```

## `JsonResponseHandler`

Handler that parses the response data to `json` and returns the dictionary.
If an error occurs trying to parse to json then a `ResponseParseError`
will be raised.

Example:

```python
client = ClientImplementation(
    authentication_method=...,
    response_handler=JsonResponseHandler,
    request_formatter=...,
)
```

## `XmlResponseHandler`

Handler that parses the response data to an `xml.etree.ElementTree.Element`.
If an error occurs trying to parse to xml then a `ResponseParseError`
will be raised.

Example:

```python
client = ClientImplementation(
    authentication_method=...,
    response_handler=XmlResponseHandler,
    request_formatter=...,
)
```

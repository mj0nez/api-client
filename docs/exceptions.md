# Exceptions

The exception handling for `api-client` has been designed in a way so that all exceptions inherit from
one base exception type: `APIClientError`.  From there, the exceptions have been broken down into the
following categories:

### `ResponseParseError`

Something went wrong when trying to parse the successful response into the defined format.  This could be due
to a misuse of the ResponseHandler, i.e. configuring the client with an `XmlResponseHandler` instead of
a `JsonResponseHandler`

### `APIRequestError`

Something went wrong when making the request.  These are broken down further into the following categories to provide
greater granularity and control.

### `RedirectionError`

A redirection status code (3xx) was returned as a final code when making the
request. This means that no data can be returned to the client as we could
not find the requested resource as it had moved.

### `ClientError`

A client error status code (4xx) was returned when contacting the API. The most common cause of
these errors is misuse of the client, i.e. sending bad data to the API.

### `ServerError`

The API was unreachable when making the request.  I.e. a 5xx status code.

### `UnexpectedError`

An unexpected error occurred when using the client.  This will typically happen when attempting
to make the request, for example, the client never receives a response.  It can also occur to
unexpected status codes (>= 600).

## Custom Error Handling

Error handlers allow you to customize the way request errors are handled in the application.

Create a new error handler, extending `BaseErrorHandler` and implement the `get_exception`
static method.

Pass the custom error handler into your client upon initialization.

Example:

```python
from apiclient.error_handlers import BaseErrorHandler
from apiclient import exceptions
from apiclient.response import Response

class MyErrorHandler(BaseErrorHandler):

    @staticmethod
    def get_exception(response: Response) -> exceptions.APIRequestError:
        """Parses client errors to extract bad request reasons."""
        if 400 <= response.get_status_code() < 500:
            json = response.get_json()
            return exceptions.ClientError(json["error"]["reason"])
        
        return exceptions.APIRequestError("something went wrong")
        
```

In the above example, you will notice that we are utilising an internal
`Response` object. This has been designed to abstract away the underlying response
returned from whatever strategy that you are using. The `Response` contains the following
methods:

* `get_original`: returns the underlying response object. This has been implemented
for convenience and shouldn't be relied on.
* `get_status_code`: returns the integer status code.
* `get_raw_data`: returns the textual data from the response.
* `get_json`: should return the json from the response.
* `get_status_reason`: returns the reason for any HTTP error code.
* `get_requested_url`: returns the url that the client was requesting.
# Making API-Requests´

## Request Formatters

Request formatters provide a way in which the outgoing request data can
be encoded before being sent, and to set the headers appropriately.

These must inherit from `BaseRequestFormatter` and implement the `format()`
method which will take the outgoing `data` object and format accordingly
before making the request.

The apiclient supports the following request formatters, by specifying
the class on initialization of the client as follows:

```python
client = ClientImplementation(
   authentication_method=...,
   response_handler=...,
   request_formatter=<RequestFormatterClass>,
)
```

### `JsonRequestFormatter`

Formatter that converts the data into a json format and adds the
`application/json` Content-type header to the outgoing requests.

Example:

```python
client = ClientImplementation(
    authentication_method=...,
    response_handler=...,
    request_formatter=JsonRequestFormatter,
)
```

## Retrying

To add some robustness to your client, the power of [tenacity](https://github.com/jd/tenacity)
has been harnessed to add a `@retry_request` decorator to the `apiclient` toolkit.

This will retry any request which responds with a 5xx status_code (which is normally safe
to do as this indicates something went wrong when trying to make the request), or when an
`UnexpectedError` occurs when attempting to establish the connection.

`@retry_request` has been configured to retry for a maximum of 5 minutes, with an exponential
backoff strategy.  For more complicated uses, the user can use tenacity themselves to create
their own custom decorator.

Usage:

```python
from apiclient import retry_request

class MyClient(APIClient):

    @retry_request
    def retry_enabled_method():
        ...

```

For more complex use cases, you can build your own retry decorator using
tenacity along with the custom retry strategy.

For example, you can build a retry decorator that retries `APIRequestError`
which waits for 2 seconds between retries and gives up after 5 attempts.

```python
import tenacity
from apiclient.retrying import retry_if_api_request_error

retry_decorator = tenacity.retry(
    retry=retry_if_api_request_error(),
    wait=tenacity.wait_fixed(2),
    stop=tenacity.stop_after_attempt(5),
    reraise=True,
)
```

Or you can build a decorator that will retry only on specific status
codes (following a failure).

```python
retry_decorator = tenacity.retry(
    retry=retry_if_api_request_error(status_codes=[500, 501, 503]),
    wait=tenacity.wait_fixed(2),
    stop=tenacity.stop_after_attempt(5),
    reraise=True,
)
```

## Pagination

In order to support contacting pages that respond with multiple pages of data when making get requests,
add a `@paginated` decorator to your client method.  `@paginated` can paginate the requests either where
the pages are specified in the query parameters, or by modifying the url.

Usage is simple in both cases; paginator decorators take a Callable with two required arguments:

- `by_query_params` -> callable takes `response` and `previous_page_params`.
- `by_url` -> callable takes `respones` and `previous_page_url`.

The callable will need to return either the params in the case of `by_query_params`, or a new url in the
case of `by_url`.
If the response is the last page, the function should return None.

Usage:

```python
from apiclient.paginators import paginated


def next_page_by_params(response, previous_page_params):
    # Function reads the response data and returns the query param
    # that tells the next request to go to.
    return {"next": response["pages"]["next"]}


def next_page_by_url(response, previous_page_url):
    # Function reads the response and returns the url as string
    # where the next page of data lives.
    return response["pages"]["next"]["url"]


class MyClient(APIClient):

    @paginated(by_query_params=next_page_by_params)
    def paginated_example_one():
        ...

    @paginated(by_url=next_page_by_url)
    def paginated_example_two():
        ...

```

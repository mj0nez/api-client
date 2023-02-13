# Implementation

## Request Strategy

The design of the client provides a stub of a client, exposing the required methods; `get`,
`post`, etc. And this then calls the implemented methods of a request strategy.

This allows us to swap in/out strategies when needed. I.e. you can write your own
strategy that implements a different library (e.g. `urllib`). Or you could pass in a
mock strategy for testing purposes.

Example strategy for testing:

```python
from unittest.mock import Mock

from apiclient import APIClient
from apiclient.request_strategies import BaseRequestStrategy

def test_get_method():
    """test that the get method is called on the underlying strategy.
    
    This does not execute any external HTTP call.
    """
    mock_strategy = Mock(spec=BaseRequestStrategy)
    client = APIClient(request_strategy=mock_strategy)
    client.get("http://google.com")
    mock_strategy.get.assert_called_with("http://google.com", params=None)
```

## Endpoints

The apiclient also provides a convenient way of defining url endpoints with
use of the `@endpoint` decorator.  In order to decorate a class with `@endpoint`
the decorated class must define a `base_url` attribute along with the required
resources.  The decorator will combine the base_url with the resource.

Example:

```python
from apiclient import endpoint

@endpoint(base_url="http://foo.com")
class Endpoint:
    resource = "search"

>>> Endpoint.resource
"http://foo.com/search"
```

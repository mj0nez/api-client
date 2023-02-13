

# API-Client Documentation

A client for communicating with an api should be a clean abstraction
over the third part api you are communicating with. It should be easy to
understand and have the sole responsibility of calling the endpoints and
returning data.

To achieve this, `APIClient` takes care of the other (often duplicated)
responsibilities, such as authentication and response handling, moving
that code away from the clean abstraction you have designed.
## Mentions

## Quick links

1. [Installation](#installation)
2. [Client in action](#usage)
3. [Adding retries to requests](#Retrying)
4. [Working with paginated responses](#Pagination)
5. [Authenticating your requests](#authentication-methods)
6. [Handling the formats of your responses](#response-handlers)
7. [Correctly encoding your outbound request data](#request-formatters)
8. [Handling bad requests and responses](#exceptions)
9. [Endpoints as code](#endpoints)
10. [Extensions](#extensions)
11. [Roadmap](#roadmap)

## Installation

```bash
pip install api-client
```

## Usage

### Simple Example

```python
from apiclient import APIClient

class MyClient(APIClient):

    def list_customers(self):
        url = "http://example.com/customers"
        return self.get(url)

    def add_customer(self, customer_info):
        url = "http://example.com/customers"
        return self.post(url, data=customer_info)

>>> client = MyClient()
>>> client.add_customer({"name": "John Smith", "age": 28})
>>> client.list_customers()
[
    ...,
    {"name": "John Smith", "age": 28},
]
```

The `APIClient` exposes a number of predefined methods that you can call
This example uses `get` to perform a GET request on an endpoint.
Other methods include: `post`, `put`, `patch` and `delete`. More
information on these methods is documented in the [Interface](#apiclient-interface).

For a more complex use case example, see: [Extended example](#extended-example)






## Extended Example

```python
from apiclient import (
    APIClient,
    endpoint,
    paginated,
    retry_request,
    HeaderAuthentication,
    JsonResponseHandler,
    JsonRequestFormatter,
)
from apiclient.exceptions import APIClientError

# Define endpoints, using the provided decorator.
@endpoint(base_url="https://jsonplaceholder.typicode.com")
class Endpoint:
    todos = "todos"
    todo = "todos/{id}"


def get_next_page(response):
    return {
        "limit": response["limit"],
        "offset": response["offset"] + response["limit"],
    }


# Extend the client for your API integration.
class JSONPlaceholderClient(APIClient):

    @paginated(by_query_params=get_next_page)
    def get_all_todos(self) -> dict:
        return self.get(Endpoint.todos)

    @retry_request
    def get_todo(self, todo_id: int) -> dict:
        url = Endpoint.todo.format(id=todo_id)
        return self.get(url)


# Initialize the client with the correct authentication method,
# response handler and request formatter.
>>> client = JSONPlaceholderClient(
    authentication_method=HeaderAuthentication(token="<secret_value>"),
    response_handler=JsonResponseHandler,
    request_formatter=JsonRequestFormatter,
)


# Call the client methods.
>>> client.get_all_todos()
[
    {
        'userId': 1,
        'id': 1,
        'title': 'delectus aut autem',
        'completed': False
    },
    ...,
    {
        'userId': 10,
        'id': 200,
        'title': 'ipsam aperiam voluptates qui',
        'completed': False
    }
]


>>> client.get_todo(45)
{
    'userId': 3,
    'id': 45,
    'title': 'velit soluta adipisci molestias reiciendis harum',
    'completed': False
}


# REST APIs correctly adhering to the status codes to provide meaningful
# responses will raise the appropriate exeptions.
>>> client.get_todo(450)
# NotFound: 404 Error: Not Found for url: https://jsonplaceholder.typicode.com/todos/450

>>> try:
...     client.get_todo(450)
... except APIClientError:
...     print("All client exceptions inherit from APIClientError")
"All client exceptions inherit from APIClientError"

```





## Roadmap

1. Enable async support for APIClient.

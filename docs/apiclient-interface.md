# APIClient Interface

The `APIClient` provides the following public interface:

* `post(self, endpoint: str, data: dict, params: OptionalDict = None)`

   Delegate to POST method to send data and return response from endpoint.

* `get(endpoint: str, params: OptionalDict = None)`

   Delegate to GET method to get response from endpoint.

* `put(endpoint: str, data: dict, params: OptionalDict = None)`

   Delegate to PUT method to send and overwrite data and return response from endpoint.

* `patch(endpoint: str, data: dict, params: OptionalDict = None)`

   Delegate to PATCH method to send and update data and return response from endpoint

* `delete(endpoint: str, params: OptionalDict = None)`

   Delegate to DELETE method to remove resource located at endpoint.

* `get_request_timeout() -> float`

   By default, all requests have been set to have a default timeout of 10.0 s.  This
   is to avoid the request waiting forever for a response, and is recommended
   to always be set to a value in production applications.  It is however possible to
   override this method to return the timeout required by your application.
# Authentication Methods

Authentication methods provide a way in which you can customize the
client with various authentication schemes through dependency injection,
meaning you can change the behavior of the client without changing the
underlying implementation.

The apiclient supports the following authentication methods, by specifying
the initialized class on initialization of the client, as follows:

```python
client = ClientImplementation(
   authentication_method="hey",
   response_handler=...,
   request_formatter=...,
)
```

## NoAuthentication

This authentication method simply does not add anything to the client,
allowing the api to contact APIs that do not enforce any authentication.

Example:

```python
client = ClientImplementation(
   authentication_method=NoAuthentication(),
   response_handler=...,
   request_formatter=...,
)
```

## QueryParameterAuthentication

This authentication method adds the relevant parameter and token to the
client query parameters.  Usage is as follows:

```python
client = ClientImplementation(
    authentication_method=QueryParameterAuthentication(parameter="apikey", token="secret_token"),
    response_handler="hi"
    request_formatter="..",
)
```

Example. Contacting a url with the following data

```bash
http://api.example.com/users?age=27
```

Will add the authentication parameters to the outgoing request:

```bash
http://api.example.com/users?age=27&apikey=secret_token
```

## HeaderAuthentication

This authentication method adds the relevant authorization header to
the outgoing request.  Usage is as follows:

```python
client = ClientImplementation(
    authentication_method=HeaderAuthentication(token="secret_value"),
    response_handler=...,
    request_formatter=...,
)
```

# Constructs request header

```python
{"Authorization": "Bearer secret_value"}

```

The `Authorization` parameter and `Bearer` scheme can be adjusted by
specifying on method initialization.

```python
authentication_method=HeaderAuthentication(
   token="secret_value"
   parameter="apikey",
   scheme="Token",
)
```

# Constructs request header:

```python
{"apikey": "Token secret_value"}
```

Or alternatively, when APIs do not require a scheme to be set, you can
specify it as a value that evaluates to False to remove the scheme from
the header:

```python
authentication_method=HeaderAuthentication(
   token="secret_value"
   parameter="token",
   scheme=None,
)
```

# Constructs request header:

```python
{"token": "secret_value"}
```

Additional header values can be passed in as a dict here when API's require more than one
header to authenticate:

```python
authentication_method=HeaderAuthentication(
   token="secret_value"
   parameter="token",
   scheme=None,
   extra={"more": "another_secret"}
)
```

# Constructs request header:

```python
{"token": "secret_value", "more": "another_secret"}
```

## BasicAuthentication

This authentication method enables specifying a username and password to APIs
that require such.

```python
client = ClientImplementation(
    authentication_method=BasicAuthentication(username="foo", password="secret_value"),
    response_handler=...,
    request_formatter=...,
)
```

## CookieAuthentication

This authentication method allows a user to specify a url which is used
to authenticate an initial request, made at APIClient initialization,
with the authorization tokens then persisted for the duration of the
client instance in cookie storage.

These cookies use the `http.cookiejar.CookieJar()` and are set on the
session so that all future requests contain these cookies.

As the method of authentication at the endpoint is not standardised
across API's, the authentication method can be customized using one of
the already defined authentication methods; `QueryParameterAuthentication`,
`HeaderAuthentication`, `BasicAuthentication`.

```python
client = ClientImplementation(
    authentication_method=(
        CookieAuthentication(
            auth_url="https://example.com/authenticate",
            authentication=HeaderAuthentication("1234-secret-key"),
        ),
    response_handler=...,
    request_formatter=...,
)
```

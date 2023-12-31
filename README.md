![test workflow](https://github.com/cogini/tesla_middleware_dynamic_headers/actions/workflows/test.yml/badge.svg)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](CODE_OF_CONDUCT.md)

# Tesla.Middleware.DynamicHeaders

Middleware for the [Tesla](https://hexdocs.pm/tesla/readme.html) HTTP client
that sets value for HTTP headers dynamically at runtime. 

This is most useful to handle secrets such as auth tokens. If you set them at
compile time, then they are hard coded into the release file, which is a
security risk. Similarly, if you build your code in a CI system, then you have
to make the secrets available there. 


## Installation

Add `tesla_middleware_dynamic_headers` to the list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:tesla_middleware_dynamic_headers, "~> 0.7.0"}
  ]
end
```

## Configuration

Add this middleware as a plug in your client.

Add `plug Tesla.Middleware.DynamicHeaders` to the client and
specify the list of dynamic headers. 

The plug takes a single option, either a list of tuples or a function. The
first element of the tuple is the header name. Other values are as follows:

* `{application, key}`: Reads the value from `Application.get_env/2`
* `{application, key, default}`: Reads the value from `Application.get_env/3`
* Function with arity 1: Calls the function with the header name to get the value
* Any other value is passed through as is, same as `Tesla.Middleware.Headers`

If the option is a zero-arity function, it is called to generate a list of
`{header_name, value`} tuples.

## Examples

The following example shows configuration via a list of headers:

```elixir
defmodule FooClient do
  use Tesla

  @app :foo_client

  plug(Tesla.Middleware.BaseUrl, "https://example.com/")

  plug(Tesla.Middleware.DynamicHeaders, [
    {"X-Foo-Token", {@app, :foo_token}},
    {"X-Bar-Token", {@app, :bar_token, "default"}},
    {"Authorization", &get_authorization/1},
    {"content/type", "application/json"}
    ])

  plug(Tesla.Middleware.Logger)

  defp get_authorization(header_name) do
    {header_name, "token: " <> Application.get_env(@app, :auth_token)}
  end
end
```

The following example shows how to use a custom function to generate all the headers:

```elixir
defmodule FooClient do
  use Tesla

  @app :foo_client

  plug Tesla.Middleware.DynamicHeaders, &get_dynamic_headers/0

  defp get_dynamic_headers do
    Application.get_env(@app, :headers)
  end
end
```

The equivalent configuration in `config/test.exs` might look like:

```elixir
config :foo_client,
  foo_token: "footoken",
  bar_token: "bartoken",
  auth_token: "authtoken"

config :foo_client,
  headers: [
    {"Authorization", "token: authtoken"}
  ]
```

In production, you would set environment variables with the tokens, then read them
in `config/runtime.exs`:

```elixir
config :foo_client,
  foo_token: System.get_env("FOO_TOKEN") || raise "missing environment variable FOO_TOKEN"
```

Documentation is here: https://hexdocs.pm/tesla_middleware_dynamic_headers

This project uses the Contributor Covenant version 2.1. Check [CODE_OF_CONDUCT.md](/CODE_OF_CONDUCT.md) for more information.

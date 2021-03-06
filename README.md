# Authtoken

[![Travis CI Build Status](https://travis-ci.org/Brainsware/authtoken.svg?branch=master)](https://travis-ci.org/Brainsware/authtoken)
![Elixir CI](https://github.com/sealas/authtoken/workflows/Elixir%20CI/badge.svg)

Simplified encrypted authentication tokens using JWE.

This package provides you with a simplified headerless and encrypted JWT. It provides you with sane defaults (AES128) and almost no configuration to counteract JWTs overblown standard. See this [blog post](https://sealas.at/blog/2017-12/tokens-cookies-and-sessions-an-auth-story-part-1/) for more information.

Example integration here in [Sealas](https://github.com/Brainsware/sealas)

## Installation

1. Add `authtoken` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:authtoken, "~> 0.3"}
  ]
end
```

2. Configure `authtoken`. The minimum amount of configuration needed is a key for encryption.

```elixir
config :authtoken,
  token_key: <<1, 2, 3, 230, 103, 242, 149, 254, 4, 33, 137, 240, 23, 90, 99, 250>>
```

You can generate this with

```elixir
iex> AuthToken.generate_key()
{:ok, <<1, 2, 3, 230, 103, 242, 149, 254, 4, 33, 137, 240, 23, 90, 99, 250>>}
```

## Usage

Generate a token for your user after successful authentication like this:

```elixir
token_content = %{userid: user.id}

{:ok, token} = AuthToken.generate_token(token_content)
```

then pass it on to your view.

### Get the content / Decrypting

The most likely use-case is going to be you wanting to decrypt a token you have received in the request headers.

```elixir
{:ok, token} = AuthToken.decrypt_token(conn)

user_id = token.userid
```

### Refreshing

By default a token should be refreshed every 30 minutes, using `AuthToken.refresh_token/1`.
You can pass either a decrypted token, or simply the token you get from the client directly.
This will extend the lifetime of the token for another 30 minutes until it expires completely.
You can use this opportunity to check if the user's credentials haven't been revoked in the meantime.

```elixir
case AuthToken.refresh_token(token) do
  {:error, :timedout} ->
    # Redirect to login
  {:error, :stillfresh} ->
    # Do nothing
  {:ok, token} ->
    # Check credentials and send back new token
end
```

### Plug

For verification you can use the plug `AuthToken.Plug.verify_token`.

```elixir
import AuthToken.Plug

pipeline :auth do
  plug :verify_token
end

scope "/protected/route", MyApp do
  pipe_through :auth

  resources "/", DoNastyStuffController
end
```

More detailed documentation can be found here: [https://hexdocs.pm/authtoken](https://hexdocs.pm/authtoken).

## Configuration

More optional configuration options

### timeout (default: 86400)

Denotes the lifetime of a token in seconds. After it expires you need to generate a new one.

### refresh (default: 1800)

The total duration after which the token needs to be refreshd

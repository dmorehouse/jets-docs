---
title: CORS Support
category: routing
order: 5
---

Enabling CORS is simple.  You just set `config.api.cors` in the `config/application.rb` file.  Here's an example:

config/application.rb:

```ruby
Jets.application.configure do
  # ...
  config.api.cors = true
end
```

A `config.api.cors = true` will add a response header with `Access-Control-Allow-Origin='*'`.

### Specific Domain

If you would like more specificity for the `Access-Control-Allow-Origin` header then you can set the domain name like so:

```ruby
Jets.application.configure do
  # ...
  config.api.cors = "*.mydomain.com"
end
```

The example above adds a response header with `Access-Control-Allow-Origin='*.mydomain.com'`.

### Full Customization

If you need full customization of the CORS response headers, you can set `config.api.cors` as a Hash.

```ruby
Jets.application.configure do
  # ...
  config.api.cors = {
    "access-control-allow-origin" => "*.mydomain.com",
    "access-control-allow-credentials" => true,
  }
end
```

If you need to control the extra headers added as part pre-flight OPTIONS request you can set `config.api.cors_preflight`:

```ruby
Jets.application.configure do
  # ...
  config.api.cors_preflight = {
    "access-control-allow-methods" => "DELETE,GET,HEAD,OPTIONS,PATCH,POST,PUT",
    "access-control-allow-headers" => "Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token,X-Amz-User-Agent",
  }
  end
```

## Authorization Type

By default, OPTIONS requests will have an `authorization_type = "NONE"`. This allows libraries and frameworks like AWS Amplify to use this HTTP endpoint to send an unsigned preflight request. For some reason if you want to specify authorization_type for the OPTIONS request, you can do this:

```ruby
Jets.application.configure do
  # ...
  config.api.cors_authorization_type = "CUSTOM" # default is "NONE"
end
```

More info: [Routes Authorization]({% link _docs/routing/authorizers/authorization-types.md %})

## Testing CORS

Here's a curl command example to help test that CORS is working.

    $ curl -H "Origin: http://example.com" \
        -H "Access-Control-Request-Method: POST" \
        -H "Access-Control-Request-Headers: X-Requested-With" \
        -X OPTIONS --verbose \
        https://pfw5gle1d8.execute-api.us-west-2.amazonaws.com/dev/ 2>&1 | grep '< HTTP'
    < HTTP/2 200

You should expect a 200 reponse code.

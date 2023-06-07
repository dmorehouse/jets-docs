---
title: Authorizers and One Method for All Routes
nav_text: One Method
category: authorizers
order: 7
---

When you use [one_apigw_method_for_all_routes]({% link _docs/config/cfn.md %}) and tell Jets to build only one APIGW Methods for all routes, IE:

config/application.rb

```ruby
Jets.application.configure do
  config.cfn.build.routes = "one_apigw_method_for_all_routes" # only works with one_lambda_for_all_controllers
end
```

Authorizers should only be set at the `root` and the `*catchall` route. This is because only these 2 APIGW resources are actually built. So authorizers can only be set at that APIGW Method. If assign an authorizer for any other route, a warning will be printed.

## Example routes.rb

Here's an example of how you set the `authorizer` option for the `root` and `*catchall` route.

config/routes.rb

```ruby
Jets.application.routes.draw do
  root "jets/public#show", authorizer: "main#protect"
  any "*catchall", to: "jets/public#show", authorizer: "main#protect"
end
```

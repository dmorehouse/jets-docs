---
title: CloudFormation Config
nav_text: CloudFormation
category: config
order: 1
---

You can configure how Jets builds the CloudFormation templates with the `cfn.build` settings.

## Lambda Functions

In Jets v5, a single Lambda function is built for all controller actions.  This is the default:

```ruby
Jets.application.configure do
  config.cfn.build.controllers = "one_lambda_for_all_controllers"
end
```

You can change this behavior for:

    one_lambda_for_all_controllers
    one_lambda_per_controller
    one_lambda_per_method

Note: In Jets v4 and below, a Lambda function was built for each controller method. IE: `config.cfn.build.controllers = "one_lambda_per_controller"`

For thoughts on the default change see: [CloudFormation Multiple or Few Lambda Functions Thoughts]({% link _docs/thoughts/cfn.md %}).

## API Gateway

Jets builds API Gateway Resources and Methods for each route defined in your `config/routes.rb`. This is the default:

```ruby
Jets.application.configure do
  config.cfn.build.routes = "one_method_per_route"
end
```

You can tell jets to build a single catchall APIGW Method instead and essentially use APIGW as a proxy. This is achieve with `config.cfn.build.routes = "one_method_for_all_routes"`.

Note: When `one_method_for_all_routes` is enabled, then `config.cfn.build.controllers = "one_lambda_for_all_controllers"`.

{% include config/reference/header.md %}
{% include config/reference/cfn.md %}

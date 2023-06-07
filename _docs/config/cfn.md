---
title: CloudFormation Config
nav_text: CloudFormation
category: config
order: 1
---

The settings covered below configure Jets and control how Jets should build the CloudFormation templates that provision the AWS Cloud resources.

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

Notes:

* For thoughts on the default change see: [CloudFormation Multiple or Few Lambda Functions Thoughts]({% link _docs/thoughts/cfn-many-lambdas.md %}).
* If you're switching from one to many lambda functions and vice-versa, Cloudformation may not be able to deploy your app because it's unable to replace existing function name collisions. You can use `JETS_RESET=1` to deploy one time to allow CloudFormation to take over management of the function names. And then deploy a 2nd time with `JETS_RESET` unset to get back the pretty human-friendly function names. See: [Blue-Green Deployments]({% link _docs/extras/blue-green-deployment.md %}).

## API Gateway

In Jets v5, Jets builds and deploys one catchall API Gateway Resource and Method for all route defined in your `config/routes.rb`. APGIW is essentially use APIGW as a proxy and Jets handles routing. This is the default:

```ruby
Jets.application.configure do
  config.cfn.build.routes = "one_apigw_method_for_all_routes"
end
```

You can tell jets to build a individual APIGW Resources and Methods. This is achieve with `config.cfn.build.routes = "one_apigw_method_per_route"`.

Notes:

* When `one_apigw_method_for_all_routes` is set, Jets automatically sets `config.cfn.build.controllers = "one_lambda_for_all_controllers"` no matter what value you set. This is the only value that makes sense since Jets collapses all the APIGW Methods down to one.
* Authorizers should only be set at the `root` and `catchall` route. Jets only respects them there. See: [Authorizers One Method]({% link _docs/routing/authorizers/one-method.md %}). You'll a warning if you use authorizers on other routes.
* If you're changing the `config.cfn.build.routes` from a previous deployed API. You might have to use `JETS_API_REPLACE=1` to tell Jets to force a full replacement of the API Gateway RestAPI.
* Technically, 2 methods are created since APIGW requires special treatment of the root path Resource. The name `one_apigw_method_for_all_routes` is

{% include config/reference/header.md %}
{% include config/reference/cfn.md %}

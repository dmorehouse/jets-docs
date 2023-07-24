---
title: Associated Resources Extensions
---

You can define your own custom associated resource methods. This helps for shorter and cleaner code. Remember that methods like `cron` and `rate` are just convenience methods that ultimately call the `resource` method. You can extend Jets with custom convenience methods.

## Example Extension

To define a custom extension, you create a module in the `app/extensions` folder.  Here's an example:

app/extensions/iot_extension.rb:

```ruby
module IotExtension
  def thermostat_rule(logical_id, props={})
    defaults = {
      topic_rule_payload: {
        sql: "SELECT * FROM 'TemperatureTopic' WHERE temperature > 60",
        actions: [
          lambda: { function_arn: "!Ref {namespace}LambdaFunction" }
        ],
        rule_disabled: true,
      },
    }
    props = defaults.deep_merge(props)
    resource(logical_id, "AWS::IoT::TopicRule", props)
  end
end
```

Note: Astute users may noticed that underscore for the keys is used. This allows the user to use underscore when using `thermostat_rule`, once `props` reaches the `resource` call, the resource call will internally camelize the keys. If the helper is meant to be user-facing, then underscore is probably what Ruby-ist would use. If you're writing internal Jets code, CamelCalse is used since we're working closely with CloudFormation properties.

After the module is defined, you can use the newly created convenience method like so:

```ruby
class TemperatureJob < ApplicationJob
  thermostat_rule(:room)
  def record
    # custom business logic
  end
end
```

The code above creates an `AWS::IoT::TopicRule` and runs the `record` Lambda function for incoming IoT thermostat data.  You can add your own custom business logic to handle the received data accordingly.

## Three Resource Forms

You might have noticed that the `thermostat_rule extension` used a different form of the `resource` method. There are 3 different forms of the `resource` method. Here are examples of each:

### Resource Long Form

```ruby
def thermostat_rule(logical_id, props={})
  # ...
  resource(
    logical_id : {
      Type: "AWS::IoT::TopicRule",
      Properties: props
    }
  )
end
```

### Resource Medium Form

```ruby
def thermostat_rule(logical_id, props={})
  # ...
  resource(logical_id,
    Type: "AWS::IoT::TopicRule",
    Properties: props
  )
end
```

### Resource Short Form

```ruby
def thermostat_rule(logical_id, props={})
  # ...
  resource(logical_id, "AWS::IoT::TopicRule", props)
end
```

### Which one to use?

You can use any of the resource forms depending on how much customization and control is needed.  It is probably best to try the simplest form first and then go up your way to the long form when needed.


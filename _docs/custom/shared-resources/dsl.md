---
title: Shared Resources DSL
---

As mentioned in [Shared Resources]({% link _docs/custom/shared-resources.md %}), the `sns_topic` is simply a convenience method that calls the `resources` and `output` methods that add sections to the CloudFormation template. Shared Resources inherit from the `Jets::Stack` class.  By inheriting from the `Jets::Stack` class, Shared Resources are provided access to a general CloudFormation template DSL.  Here are the main methods of that DSL:

DSL Method | Description
--- | ---
parameter | Adds a parameter to the template.
resource | Adds a resource to the template.
output | Adds an output to the template.

The main methods correspond to sections of the [CloudFormation anatomy sections](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html).

Each method has long, medium and short forms.  Here are some contrived examples that show their different forms:

## Parameters

```ruby
class ParametersExample < Jets::Stack
  # long form
  parameter(InstanceType: {
    Default: "t2.micro" ,
    Description: "instance type" ,
  })
  # medium form
  parameter :Company, Default: "boltops", Description: "instance type"
  # short form
  parameter :AmiId, "ami-123" # default is ami-123
end
```

## Resources

```ruby
class ResourcesExample < Jets::Stack
  # long form
  resource(SnsTopic: {
    Type: "AWS::SNS::Topic",
    Properties: {
      Description: "my desc",
      DisplayName: "my name",
    }
  })
  # medium form
  resource(:SnsTopic2,
    Type: "AWS::SNS::Topic",
    Properties: {
      DisplayName: "my name 2",
    }
  )
  # short form
  resource(:SnsTopic3, "AWS::SNS::Topic",
    DisplayName: "my name 3",
  )
end
```

## Outputs

```ruby
class OutputsExample < Jets::Stack
  # long form
  output(VpcId: {
    Description: "vpc id",
    Value: ref(:VpcId), # same as: Value: "!Ref VpcId"
  })
  # medium form
  output :StackName, Value: "!Ref AWS::StackName"
  # short form
  output :Elb, "!Ref Elb" # same as
               # output :Elb, Value: "!Ref Elb"
  output :Elb2 # short form, same as:
               # output :Elb2, "!Ref Elb2"

end
```

The DSL provides full access to creating custom CloudFormation stacks and AWS resources.  It is also easy extend the DSL with your own [Shared Resource Extensions]({% link _docs/custom/shared-resources/extensions.md %}). This helps you remove duplication and keep your code concise.

{% include custom/camelcase-note.md %}

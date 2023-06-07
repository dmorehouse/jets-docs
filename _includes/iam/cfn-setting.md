## CloudFormation Controllers Build Setting

Important: In Jets v5, the [config.cfn.build.controllers]({% link _docs/config/cfn.md %}) setting was introduced. Depending on whether or not Jets is configured to deploy `one_lambda_per_method`, `one_lambda_per_controller`, or `one_lambda_for_all_controllers`, you have different level of controls over how fine-grain the IAM Policies can be defined for controllers. Jets will print a warning message when you're using an IAM policy that is not available.

For [jobs]({% link _docs/jobs.md %}), you have the ability to control IAM Policies at the individual Lambda function level because Jets always deploys an distinct Lambda functions for each job method.

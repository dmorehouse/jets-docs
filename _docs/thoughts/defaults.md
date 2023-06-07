---
title: Defaults Are Hard
nav_text: Defaults
desc: It's hard because it's impossible to please everyone.
category: thoughts
order: 3
---

Choosing good default values is hard because it's impossible to please everyone. However, that does not mean we shouldn't try! Especially since it provides the benefits of conventions of configuration. 😄

## The Purist

The Serverless Purist will probably be quick to clamor that AWS Lambda should be used in a micro manner. They say there should be a distinct lambda function for each controller action. They'll find reasons why the Jets single Lambda function default and single route default is an anti-pattern in the Serverless world. They don't account for reality.

## The Realist

The Serverless Realist will sternly say that in the real-world, it's impossible to use distinct Lambda functions for each controller action. They have probably not seen that a Jets app can successfully deploy 500+ functions in minutes. Or they've had bad experiences with CloudFormation rollbacks and given up in tears. Admittedly, I've had plenty of bad experiences with CloudFormation rollbacks myself.

## The Pragmatist

The Serverless Pragmatist will always say there needs to be a compromise. The Pragmatist is sometimes overly balanced and can sometimes have a difficult time acknowledging there are sometimes black-and-white or right-and-wrong answers in life.

## The Experiencist

The Serverless Experiencist is a mixture of all of these people. They have had enough battle scars to know there's some truth to their opinions. They also know that software constantly changes, and what they know today might not be correct tomorrow.

## Why the Current Defaults

The Jets current defaults were chosen after years of toiling with Jets, AWS, CloudFormation, Ruby, Rails, building Applications, and building Cloud infrastructure. It's a gut call on what are reasonable defaults. More concrete details are provided in these other docs pages like [CloudFormation Thoughts]({% link _docs/thoughts/cfn-many-lambdas.md %}).


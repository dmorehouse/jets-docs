---
title: Blue-Green Deployment
category: extras
order: 12
---

## Background

Underneath the hood, Jets uses CloudFormation to deploy Lambda functions and API Gateway resources. During an update operation, CloudFormation creates new resources, makes sure they are first created successfully, and then deletes any old resources. CloudFormation does this to avoid deleting the existing resource and putting us stuck in a terrible state. It is entirely logical.

## Automated Blue-Green Deployment

For some resources, the way CloudFormation updates can sometimes fail and rollback. Notably, changing API Gateway routes can cause a rollback. Jets checks the routes and replaces the API Gateway Rest API entirely when needed to avoid a rollback. Jets essentially performs an automated blue-green of API Gateway in this case. This results in changing the API Gateway DNS endpoint. At the end of `jets deploy` the updated API Gateway endpoint is provided.

If you have configured a [Custom Domain]({% link _docs/routing/custom-domain.md %}) then this custom domain automatically gets updated as part of the automated blue-green deployment.

## Manual Blue-Green Deployment

For the most part, Jets auto blue-green deployments suffice. Manual blue-green deployments are sometimes required, though. For example, [upgrading]({% link _docs/extras/upgrading.md %}) between different versions of Jets can require a blue-green deployment.

This is where Jets and AWS Lambda power shines. We simply create another [extra environment]({% link _docs/env-extra.md %}) and switch to it to do a manual blue-green deployment. Here are the steps:

1. Create another environment by deploying with `JETS_EXTRA`.
2. Test it to your heart's content
3. Switch the API Gateway Custom Domain over to the new stack
4. Delete the old environment

When we create new environments, there will be no CloudFormation update issues because the application is entirely brand new.

## In-Place Blue-Green Deployement: JETS_RESET

For certain cases, CloudFormation may not be unable to deploy the Jets app, and you don't have the time for a blue-green deployment. For example, route changes that result in needed to replace APIGW entirely. If you need to replace the APIGW, you can use `JETS_API_REPLACE=1`.

In other cases, CloudFormation won't be able to deploy the Lambda Function stacks. For example, changing `config.cfn.build.controllers` from `one_lambda_for_all_controllers` to `one_lambda_per_method` and vice-versa. You can:

1. `JETS_RESET=1 jets deploy` to deploy once, allowing CloudFormation to manage the function names. This allows CloudFormation to avoid the existing conflicting function names.
2. Deploy a 2nd time with `jets deploy` without the `JETS_RESET` to get back the pretty human-friendly function names.

The `JETS_RESET=1` approach is a quick way to get around some of the CloudFormation quirks and provides an in-place blue-green deployment. If you want to be extra cautious, using a regular blue-green deployment with `JETS_EXTRA` as described above can be taken.

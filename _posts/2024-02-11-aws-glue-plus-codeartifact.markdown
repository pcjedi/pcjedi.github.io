---
layout: post
---
# How to setup your AWS Glue Job to access your Codeartifact Packages

If [this official documentation](https://aws.amazon.com/de/blogs/big-data/simplify-and-optimize-python-package-management-for-aws-glue-pyspark-jobs-with-aws-codeartifact/) is not applicable to you, maybe because you are not using step functions, then this documentation is for you.

## Goal

The goal is to configure your AWS Glue job in a way, that you can declare your python codeartifact packages as [additional-python-modules](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-glue-arguments.html#additional-python-modules) like [documented for pypi packages](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-python-libraries.html). We will facilitate [python-modules-installer-option](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-glue-arguments.html#python-modules-installer-option) for this purpose. We will need a [AWS Lambda](https://aws.amazon.com/lambda/) function to be called before the AWS Glue job, so that the Lambda function can set these arguments. This is necessary, because the token for the AWS Codeartifact has only a maximum duration of [12 hours](https://docs.aws.amazon.com/codeartifact/latest/ug/tokens-authentication.html).

## End Result

Let me start with the finished result and then break it down. The Lambdafunction should look something like this:

```python
import boto3

job_name="your-glue-job-name-here"
domain="your-codeartifact-domain-here"
repository="your-codeartifact-repository-here"
domain_owner="your-codeartifact-domain_owner-here"


def get_index_url(
    domain: str,
    repository: str,
    repo_format: str,
    domain_owner: str,
    duration_seconds: int = 0,
    prefix: str = "",
    suffix: str = "",
) -> str:
    "get index url with authorization-token from aws codeartifact"
    codeartifact = boto3.client("codeartifact")
    repository_endpoint = codeartifact.get_repository_endpoint(
        domain=domain,
        domainOwner=domain_owner,
        repository=repository,
        format=repo_format,
    ).get("repositoryEndpoint")
    authorization_token = codeartifact.get_authorization_token(
        domain=domain,
        domainOwner=domain_owner,
        durationSeconds=duration_seconds,
    ).get("authorizationToken")
    if repo_format == "pypi":
        suffix = "simple"
    return prefix + repository_endpoint.replace("https://", f"https://aws:{authorization_token}@") + suffix


def  lambda_handler(event, context):
    glue_client = boto3.client("glue")

    default_arguments = glue_client.get_job(JobName=job_name).get("Job", {}).get("DefaultArguments", {})
    overwrite_arguments = {
        "--python-modules-installer-option": get_index_url(
            domain=domain,
            repository=repository,
            repo_format="pypi",
            domain_owner=domain_owner,
            prefix="--index-url=",
        )
    }
    arguments = default_arguments | overwrite_arguments

    return glue_client.start_job_run(JobName=job_name, Arguments=arguments)
```

You can give it more complexity if you have to of course, but the basic stuff for authentication a gluejob for the codeartifact are in here. Let me break some stuff down:

- ```python
    glue_client.get_job(JobName=job_name).get("Job", {}).get("DefaultArguments", {})
    ```

    This will read the arguments of the glue job, especially the given additional python modules will be interesting. If we wouldn't do this here, the job run wouldn't know about them.
- `overwrite_arguments`
    In this dictinary we will place the values that are to be added/edited. it will look something like this:

    ```python
    {"--python-modules-installer-option": "--index-url=https://aws:token@aws.codeartifact.uri"}
    ```

- `get_index_url`
    This function combines the codeartifact endpoint and the codeartifact access token.

## Discussion

You now have a lambda handler, that is able to trigger a glue job with authentication to the codeartifact. Of course, you will need to set the permissions right, as the lambda needs to have privileges to do the stuff it does now. In the logs of the Glue job runs will the codeartifact token be visible. This is not that critical, as it only has a duration of 12h and people/resources who are previliged to see glue job run details are probably also permitted to read your codeartifact. But keep this in mind.

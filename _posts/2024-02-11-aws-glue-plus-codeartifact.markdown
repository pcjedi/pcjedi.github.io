---
layout: post
---
# How to setup your AWS Glue Job to access your Codeartifact Packages

If [this official documentation](https://aws.amazon.com/de/blogs/big-data/simplify-and-optimize-python-package-management-for-aws-glue-pyspark-jobs-with-aws-codeartifact/) is not applicable to you, maybe because you are not using step functions, then this documentation is for you.

## Goal

The goal is to configure your AWS Glue job in a way, that you can declare your python codeartifact packages as [additional-python-modules](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-glue-arguments.html#additional-python-modules) like [documented for pypi packages](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-python-libraries.html). We will faccilitate [python-modules-installer-option](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-glue-arguments.html#python-modules-installer-option) for this purpose. 

{% raw %}

{% endraw %}

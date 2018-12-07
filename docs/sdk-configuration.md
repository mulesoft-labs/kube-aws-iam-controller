# Configure credentials for SDKs

The AWS SDKs does not have complete feature parity and therefore must be
configured slightly different to recieve credentials in Kubernetes.

Below is a support matrix for the different SDKs along with a configuration
guide for those that have support already.

| SDK | Supported | Comment |
| --- | --------- | ------- |
| [Java AWS SDK (JVM)](#java-aws-sdk-jvm) | :heavy_check_mark: | |
| [Python AWS SDK (boto3)](#python-aws-sdk-boto3) | :heavy_check_mark: | |
| [AWS CLI](#aws-cli) | :heavy_check_mark: | |
| [Ruby AWS SDK](#) | :heavy_plus_sign: | Supported but not yet tested ([aws-sdk-ruby/#1820](https://github.com/aws/aws-sdk-ruby/pull/1820)) |
| [Golang AWS SDK](#golang-aws-sdk) | :heavy_check_mark: | |
| [JS AWS SDK](#) | :heavy_multiplication_x: | Not yet supported ([aws-sdk-js/#1923](https://github.com/aws/aws-sdk-js/pull/1923)) |

## Java AWS SDK (JVM)

| SDK | Tested version |
|-----| -------------- |
| [aws-sdk-java](https://github.com/aws/aws-sdk-java) | `>=1.11.394` |

Here's a minimal example of how to configure a deployment so each pod will get
the AWS credentials.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aws-iam-java-example
spec:
  replicas: 1
  selector:
    matchLabels:
      application: aws-iam-java-example
  template:
    metadata:
      labels:
        application: aws-iam-java-example
    spec:
      containers:
      - name: aws-iam-java-example
        image: mikkeloscar/kube-aws-iam-controller-java-example:latest
        env:
        # must be set for the Java AWS SDK/AWS CLI to find the credentials file.
        - name: AWS_CREDENTIAL_PROFILES_FILE
          value: /meta/aws-iam/credentials
        volumeMounts:
        - name: aws-iam-credentials
          mountPath: /meta/aws-iam
          readOnly: true
      volumes:
      - name: aws-iam-credentials
        secret:
          # secret should be named: aws-iam-<name-of-your-aws-iam-role>
          secretName: aws-iam-aws-iam-example
```

It's important that you set the `AWS_CREDENTIALS_PROFILES_FILE` environment
variable as shown in the example as well as mount the secret named
`aws-iam-<name-of-your-iam-role>` into the pod. This secret will be provisioned
by the **kube-aws-iam-controller**.

See full [Java example project](https://github.com/mikkeloscar/kube-aws-iam-controller-java-example).

## Python AWS SDK (boto3)

| SDK | Tested version |
|-----| -------------- |
| [boto3](https://github.com/boto/boto3) | `>=1.9.28` |

Here's a minimal example of how to configure a deployment so each pod will get
the AWS credentials.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aws-iam-python-example
spec:
  replicas: 1
  selector:
    matchLabels:
      application: aws-iam-python-example
  template:
    metadata:
      labels:
        application: aws-iam-python-example
    spec:
      containers:
      - name: aws-iam-python-example
        image: mikkeloscar/kube-aws-iam-controller-python-example:latest
        env:
        # must be set for the AWS SDK/AWS CLI to find the credentials file.
        - name: AWS_SHARED_CREDENTIALS_FILE # used by python SDK
          value: /meta/aws-iam/credentials.process
        - name: AWS_DEFAULT_REGION # adjust to your AWS region
          value: eu-central-1
        volumeMounts:
        - name: aws-iam-credentials
          mountPath: /meta/aws-iam
          readOnly: true
      volumes:
      - name: aws-iam-credentials
        secret:
          # secret should be named: aws-iam-<name-of-your-aws-iam-role>
          secretName: aws-iam-aws-iam-example
```

It's important that you set the `AWS_SHARED_CREDENTIALS_FILE` environment
variable as shown in the example as well as mount the secret named
`aws-iam-<name-of-your-iam-role>` into the pod under `/meta/aws-iam`. This
secret will be provisioned by the **kube-aws-iam-controller**.

Also note that for this to work the docker image you use **MUST** contain the
program `cat`. [`cat` is called by the SDK to read the credentials from a
file](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#sourcing-credentials-from-external-processes).

See full [Python example project](https://github.com/mikkeloscar/kube-aws-iam-controller-python-example).

## AWS CLI

| SDK | Tested version |
|-----| -------------- |
| [aws-cli](https://github.com/aws/aws-cli) | `>=1.16.43` |

Configuration is the same as for the [Python AWS SDK](#python-aws-sdk-boto3).

## Golang AWS SDK

| SDK | Tested version |
|-----| -------------- |
| [aws-sdk-go](https://github.com/aws/aws-sdk-go) | `>=`[89ebd3a5f70a416b84e65ad55d9935b7ba72e2dc](https://github.com/aws/aws-sdk-go/commit/89ebd3a5f70a416b84e65ad55d9935b7ba72e2dc) (No release yet) |

Here's a minimal example of how to configure a deployment so each pod will get
the AWS credentials.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aws-iam-golang-example
spec:
  replicas: 1
  selector:
    matchLabels:
      application: aws-iam-golang-example
  template:
    metadata:
      labels:
        application: aws-iam-golang-example
    spec:
      containers:
      - name: aws-iam-golang-example
        image: mikkeloscar/kube-aws-iam-controller-golang-example:latest
        env:
        # must be set for the AWS SDK/AWS CLI to find the credentials file.
        - name: AWS_SHARED_CREDENTIALS_FILE # used by golang SDK
          value: /meta/aws-iam/credentials.process
        - name: AWS_REGION # adjust to your AWS region
          value: eu-central-1
        volumeMounts:
        - name: aws-iam-credentials
          mountPath: /meta/aws-iam
          readOnly: true
      volumes:
      - name: aws-iam-credentials
        secret:
          # secret should be named: aws-iam-<name-of-your-aws-iam-role>
          secretName: aws-iam-aws-iam-example
```

It's important that you set the `AWS_SHARED_CREDENTIALS_FILE` environment
variable as shown in the example as well as mount the secret named
`aws-iam-<name-of-your-iam-role>` into the pod under `/meta/aws-iam`. This
secret will be provisioned by the **kube-aws-iam-controller**.

Also note that for this to work the docker image you use **MUST** contain the
program `cat`. [`cat` is called by the SDK to read the credentials from a
file](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#sourcing-credentials-from-external-processes).

See full [Golang example project](https://github.com/mikkeloscar/kube-aws-iam-controller-golang-example).
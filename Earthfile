VERSION 0.7
PROJECT applied-knowledge-systems/aws_op_earthly

FROM ubuntu:18.04

ci-pipeline:
  PIPELINE
  TRIGGER push main
  TRIGGER pr main
  BUILD +target
          
target:
  BUILD +run --region="eu-west-2" --args="sts get-caller-identity"

ubuntu:
  FROM ubuntu:18.04
  WORKDIR /workspace
  RUN apt update -y
  RUN apt install -y curl unzip

login-op:
  FROM +ubuntu
  DO +INSTALL_OP_CLI
  DO +INSTALL_CLI
  COPY aws.env . 
  RUN --secret OP_CONNECT_HOST=OP_CONNECT_HOST --secret OP_CONNECT_TOKEN=OP_CONNECT_TOKEN op run --env-file="/workspace/aws.env" -- aws sts get-caller-identity
  RUN --secret OP_CONNECT_HOST=OP_CONNECT_HOST --secret OP_CONNECT_TOKEN=OP_CONNECT_TOKEN op run --env-file="/workspace/aws.env" -- aws s3 ls

run:
  ARG --required region
  ENV AWS_REGION=$region
  ARG --required args
  FROM +login-op
  RUN --no-cache --secret OP_CONNECT_HOST=OP_CONNECT_HOST --secret OP_CONNECT_TOKEN=OP_CONNECT_TOKEN op run --env-file="/workspace/aws.env" -- aws $args --region $region


# Installs the AWS CLI
INSTALL_CLI:
  COMMAND
  ARG TARGETARCH
  IF [ "$TARGETARCH" = "amd64" ]
    ARG aws_architecture="x86_64"
  ELSE IF [ "$TARGETARCH" = "arm64" ]
    ARG aws_architecture="aarch64"
  ELSE
    RUN echo "unknown architecture $TARGETARCH" && exit 1
  END
  RUN curl --location --fail --silent --show-error "https://awscli.amazonaws.com/awscli-exe-linux-$aws_architecture.zip" --output "awscliv2.zip" && \
    unzip -q awscliv2.zip && \
    ./aws/install && \
    rm -rf awscliv2.zip aws


# Installs the 1Password Connect CLI2
INSTALL_OP_CLI:
  COMMAND
  ARG TARGETARCH
  ENV ARCH "$TARGETARCH"
  ENV OP_CLI_VERSION "v2.0.0"

  RUN echo "https://cache.agilebits.com/dist/1P/op2/pkg/${OP_CLI_VERSION}/op_linux_${ARCH}_${OP_CLI_VERSION}.zip"
  RUN curl -sSfo op.zip "https://cache.agilebits.com/dist/1P/op2/pkg/${OP_CLI_VERSION}/op_linux_${ARCH}_${OP_CLI_VERSION}.zip" \
    && unzip -od /usr/local/bin/ op.zip \
    && rm op.zip

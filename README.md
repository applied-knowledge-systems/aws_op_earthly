Demo of pipeline using 1password connect cli to fetch AWS secrets

Assuming OP_CONNECT_HOST and OP_CONNECT_TOKEN in op_args in higher level folder:
Run locally:
```
earthly --secret-file-path ../op_args -i +run --region="eu-west-2" --args="sts get-caller-identity"
```



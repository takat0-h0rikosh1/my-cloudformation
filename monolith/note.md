
```bash
$ aws cloudformation create-stack \
    --stack-name "my-stack" --template-body file://$PWD/main.yaml \
    --parameters ParameterKey=S3BucketNameParam,ParameterValue=my-gomibako
    --capabilities CAPABILITY_NAMED_IAM
    
$ aws cloudformation deploy --template file://$PWD/main.yaml --stack-name my-stack 
```

```bash
$ aws cloudformation describe-stacks --stack-name my-stack
```

```bash
$ aws cloudformation list-stack-resources --stack-name my-stack
```

```bash
$ aws cloudformation package --template $PWD/main.yaml --s3-bucket packaged-cf-template
```

`--no-fail-on-empty-changeset` を指定で変更がなくても終了ステータスを0としてくれる。



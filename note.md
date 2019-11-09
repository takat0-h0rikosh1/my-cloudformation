```sh
$ aws-vault exec me -- \
	aws cloudformation create-stack \
	--stack-name "my-stack" \
	--template-body file://$PWD/temp.yml

{
    "StackId": "arn:aws:cloudformation:ap-northeast-1:593653836732:stack/my-stack/89b14450-02ba-11ea-899b-06fcc76a2472"
}
```

defautl の vpc id の指定がないとダメ？
No default VPC for this user (Service: AmazonEC2; Status Code: 400; Error Code: VPCIdNotSpecified; Request ID: 1cf5aebd-fa77-4cc6-a8db-5a2308b8774f)

Refの説明
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html

Fn::GetAttの説明
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html

AWS CloudFormationを使ってEC2インスタンスを作成する
https://blog.ismg.kdl.co.jp/aws/cloudformation_createinstance

update-stack できなかった。

```sh
$ aws-vault exec me -- \
	aws cloudformation create-stack \
	--stack-name "my-stack" \
	--template-body file://$PWD/temp.yml

An error occurred (ValidationError) when calling the UpdateStack operation: Stack:arn:aws:cloudformation:ap-northeast-1:593653836732:stack/my-stack/89b14450-02ba-11ea-899b-06fcc76a2472 is in ROLLBACK_COMPLETE state and can not be updated.
```

ROLLBACK_COMPLETE となった STACK は削除以外のオペレーションはできない。
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/using-cfn-describing-stacks.html

消そう。

```sh
$ aws-vault exec me -- \
	aws cloudformation delete-stack \
	--stack-name "my-stack"
```

Tags の指定は以下のようにしないとダメ。

```yml
Tags: 
 - 
   Key: "Name"
   Value: "MyVPC"
```

# note

Instance Profile の参考になりそうなテンプレート
https://gist.github.com/sakamaki-kazuyoshi/4aaa4abba47e5a651f32a07656abbf4b#file-sample2-template

Parameterの外出し定義
https://qiita.com/okubot55/items/b18a5dd5166f1ec2696c
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-cloudformation-interface.html

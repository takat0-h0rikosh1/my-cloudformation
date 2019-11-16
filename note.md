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

2019/11/04 ---

Subnet の設定を追加した。

```yml
  MySubnet:
    Type: AWS:EC2:Subnet
    Properties:
      AvailabilityZone: "ap-northeast-1a"
      CidrBlock: "10.0.1.0/24" 
      VpcId: !Ref MyVpc
      Tags: 
        - Key: Name
          Value: MySubnet
```
	
とりあえず、自分のアカウントにデフォルトで一つでもVPCが無いと CloudFormation は動いてくれないみたい？
EC2 Instance の設定に SecutiryGroups と SubnetId を同時に紐付けることはできないらしい。

- インターネットゲートウェイ
- ルートテーブルの設定をやる

2019/11/15 ---

ig, vpc-gateway-attachment, route-table, route, subnet-route-table-association の設定を追加

subnet は private 用に用意することもあるみたいだ。
その場合、route は作成しないらしい。

2019/11/16 ---

ssh するには keypair を作って stack の ec2 の KeyName の指定が必要。
https://docs.aws.amazon.com/cli/latest/reference/ec2/create-key-pair.html

cli でパラメータを指定するとき
--parameters ParameterKey=MyKeyPair,ParameterValue=my-key-pair

```bash
$ aws-vault exec me -n -- \ 
  aws cloudformation update-stack \
  --stack-name "my-stack" \
  --template-body file://$PWD/Main.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=MyKeyPair,ParameterValue=[すでに存在するKeyPair]
```

AWS::IAM::Policy はインラインポリシーを作る。

aws s3 ls が遅い。  
インターネットには接続できているはずなんだが。  
curl http://sample.com/ は成功した。

セキュリティグループの Egress の設定を http に絞っていたことが原因。
aws cli は https で port は 443。

# ベストプラクティス

## クロススタック参照

クロススタック参照を利用して stack 作成時にリソースの export, import できる機能。
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/walkthrough-crossstackref.html

## テンプレートの再利用

パラメータやマッピング、条件セクションを利用して stack 作成時にパラメータをカスタマイズできるようにする。

コピペをせず、出来合いに stack を参照できる。
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/template-anatomy.html

## テンプレートに認証情報を埋め込まない

認証情報は必ず入力パラメータから受け取るようにする。  
その際は、必ず NoEcho オプションをつける。

## テンプレート実行前に検証する

`aws cloudformation validate-template` を使え。

# note

Instance Profile の参考になりそうなテンプレート
https://gist.github.com/sakamaki-kazuyoshi/4aaa4abba47e5a651f32a07656abbf4b#file-sample2-template

Parameterの外出し定義
https://qiita.com/okubot55/items/b18a5dd5166f1ec2696c
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-cloudformation-interface.html

AWS::EC2::Route
-> VPC内のルートテーブルにルートを指定する
-> igw, virtual private gw, nat instance, nat gateway, vpc pairing, network interface, egress only igw のいずれかの指定が必要

AWS::EC2::RouteTable
-> 指定されたVPCのルートテーブルを紐付ける。

AWS::EC2::SubnetRouteTableAssociation
サブネットをルートテーブルに関連付ける。
ルートテーブルは複数のサブネットに関連付けることができる。

`--capabilities CAPABILITY_IAM`

aws-vault exec me -n しないとダメ。

S3, RDS, SES

p4f-ses:
https://console.aws.amazon.com/iam/home?region=ap-northeast-1#/users/p4f-ses

creativedb-bot:
https://console.aws.amazon.com/iam/home?region=ap-northeast-1#/users/creativedb_bot?section=permissions

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:*",
                "route53domains:*",
                "cloudfront:ListDistributions",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticbeanstalk:DescribeEnvironments",
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:GetBucketWebsite",
                "ec2:DescribeVpcs",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeRegions",
                "sns:ListTopics",
                "sns:ListSubscriptionsByTopic",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:GetMetricStatistics"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "apigateway:GET",
            "Resource": "arn:aws:apigateway:*::/domainnames"
        }
    ]
}
```

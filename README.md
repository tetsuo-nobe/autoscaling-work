# EC2 Auto Scaling の環境を作成してみよう

* このワークでは、EC2 AutoScaling  の環境を作成します。

![概要](images/agent.png)

---
## 準備

* インストラクターが指定した環境で AWS マネジメントコンソールにサインインして下さい。
    - **このワークの環境は、ワークを実施する時間帯のみ使用可能です。**
* AWS マネジメントコンソールで、**指定されたリージョン**を選択した状態にしてください。
* ご自分に割り当てられた **2桁の番号**を覚えておいてください。


---
## VPC リソースの作成

1. AWS マネジメントコンソールのページ上部の **検索**で `cfn` と入力して **CloudFormation** のメニューを選択します。
1. **スタックの作成**　から **新しいリソースを使用（標準）** を選択します。
1. **テンプレートの準備**　で **既存のテンプレートを選択** を選択します。
1. **テンプレートソース**　で **Amazon S3 URL** を選択します。
1. **Amazon S3 URL** に下記の URL を入力します。
    - `https://tnobep-work-public.s3.ap-northeast-1.amazonaws.com/autoscaling-work/autoscaling_work_network.yaml`
1. **次へ**　を選択します。
1. **スタック名** に `vpc-stack-<自分の2桁の番号>` を入力します。
1. **パラメータ** の **YOURID** に **自分の2桁の番号** を入力します。 
1. **次へ**　を選択します。
1. **次へ**　を選択します。
1. **送信**　を選択します。
1. スタックのステータスが **CREATE_COMPLETE** になるまで待ちます。
1. **出力**　タブを選択します。
1. **ALBDNSName** の値をメモしておきます。
---
## 起動テンプレートの作成

1. AWS マネジメントコンソールのページ上部の **検索**で `ec2` と入力して **EC2** のメニューを選択します。
1. ページ左側で **インスタンス** の **起動テンプレート** を選択します。
1. **起動テンプレートを作成** を選択します。
1. **起動テンプレート名** に `launch-template-<自分の2桁の番号>` を入力します。
1. **EC2 Auto Scaling で使用できるテンプレートをセットアップする際に役立つガイダンスを提供** にチェックします。
1. **アプリケーションおよび OS イメージ (Amazon マシンイメージ)** で **クイックスタート** を選択します。 
1. **Amazon Linux** を選択します。
1. **インスタンスタイプ** で　**t2.micro** を選択します。
1. **高度な詳細** を展開表示します。
1. ページを下にスクロールして **ユーザーデータ** に下記を入力します。
   
```
#!/bin/bash
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
AZ=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
yum update -y
yum -y install httpd
cat <<EOF >> /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>EC2</title>
</head>
<body>
<p><font size="7" >Welcome to AWS at $(date +%F) ! (from EC2) at ${AZ}</font></p>
</body>
</html>
EOF
chown -R apache:apache /var/www/html
service httpd start
chkconfig httpd on
```

1. **起動テンプレートを作成**　を選択します。
1. **起動テンプレートを表示**　を選択します。

---
## Auto Scaling Group の作成

1. 作成した起動テンプレートの左のチェックボックスをチェックします。
1. **アクション** から **Auto Scaling グループを作成** を選択します。
1. **Auto Scaling グループ名** に `asg-<自分の2桁の番号>` を入力します。   
1. **次へ**　を選択します。
1. **ネットワーク** で **VPC** で **my-VPC-<自分の2桁の番号>**　を選択します。
1. **アベイラビリティーゾーンとサブネット** で下記の **2つのサブネット**を選択します。
    - **PrivateSubnet01-<自分の2桁の番号>**
    - **PrivateSubnet02-<自分の2桁の番号>**
1. **次へ**　を選択します。
1. **既存のロードバランサーにアタッチする** を選択します。
1. **既存のロードバランサーターゲットグループ**
   
---

## CloudWatch ログの削除
1. Amazon CloudWatch のページに切り替えます。
1. **ログ** - **ロググループ** を選択します。
1. /aws/lambda/(関数名) のロググループのチェックボックスをチェックして、**アクション** - **ロググループの削除** を選択します。
1. 確認のダイアログで **削除** を選択します。
   
---
### お疲れさまでした。

* **このワークの環境は、ワークを実施する時間帯のみ使用可能です。**

# EC2 AutoScaling ワーク

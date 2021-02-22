+++
title = "Khởi tạo một CDK project"
date = 2020
weight = 2
chapter = false
pre = "<b>2. </b>"
+++

---
Trong chương này, chúng ta sẽ sử dụng lệnh cdk init để khởi tạo một AWS CDK Python project mới. 
Thông qua đó chúng ta cũng sẽ học được cách sử dụng CDK Toolkit tổng hợp mã lệnh từ ngôn ngữ lập trình để tạo nên một AWS CloudFormation template và cũng học được cách triển khai một ứng dụng cụ thể vào AWS Account của bạn. 

### 1. CDK INIT
- Thực hiện tạo một thư mục mới dùng để chứa CDK project
```python
mkdir cdkworkshop 
cd cdkworkshop
```
- Sau khi đã ở trong thư mục mới, thực hiện tạo mới một CDK project bằng lệnh cdk init
```python
cdk init sample-app --language python
```
Kết quả của lệnh tạo giống như mô tả bên dưới: 
```python
Applying project template sample-app for python
Initializing a new git repository...
Executing Creating virtualenv...

# Welcome to your CDK Python project!

You should explore the contents of this project. It demonstrates a CDK app with an instance of a stack (`CdkWorkshopStack`)
which contains an Amazon SQS queue that is subscribed to an Amazon SNS topic.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

This project is set up like a standard Python project.  The initialization process also creates
a virtualenv within this project, stored under the .env directory.  To create the virtualenv
it assumes that there is a `python3` executable in your path with access to the `venv` package.
If for any reason the automatic creation of the virtualenv fails, you can create the virtualenv
manually once the init process completes.

To manually create a virtualenv on MacOS and Linux:

python3 -m venv .env

After the init process completes and the virtualenv is created, you can use the following
step to activate your virtualenv.

source .env/bin/activate

If you are a Windows platform, you would activate the virtualenv like this:

% .env\Scripts\activate.bat

Once the virtualenv is activated, you can install the required dependencies.

pip install -r requirements.txt

At this point you can now synthesize the CloudFormation template for this code.

cdk synth

You can now begin exploring the source code, contained in the cdk-workshop directory.
There is also a very trivial test included that can be run like this:

pytest

To add additional dependencies, for example other CDK libraries, just add to
your requirements.txt file and rerun the `pip install -r requirements.txt`
command.

## Useful commands

 * `cdk ls`          list all stacks in the app
 * `cdk synth`       emits the synthesized CloudFormation template
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk docs`        open CDK documentation

Enjoy!
```
Như bạn có thể thấy, cửa sổ terminal hiển thị hàng loạt câu lệnh rất hữu ích để chúng ta có thể bắt đầu làm việc với một CDK project. 

### 2. Kích hoạt môi trường ảo hóa Virtualenv
Kịch bản khởi tạo với lệnh cdk init mà chúng ta vừa chạy ở trên không chỉ khởi tạo project, cung cấp hàng loạt mã lệnh hữu ích để bắt đầu làm việc với project, mà nó còn giúp tạo một môi trường ảo hóa nằm bên trong thư mục của cdk project. 
Nếu bạn chưa từng sử dụng Virtualenv trước đây, bạn có thể tìm hiểu thêm về nó ở [đây](https://docs.python.org/3/tutorial/venv.html), nhưng nói một cách ngắn gọn virtualenv cung cấp cho bạn một môi trường khép kín và tách biệt để chạy Python và cài đặt các gói thư viện cần thiết mà không làm ảnh hưởng tới cấu hình Python ở môi trường bên ngoài.
Để tận dụng được ưu điểm của môi trường ảo hóa mà chúng ta đã tạo, thực hiện kích hoạt chúng trên cửa sổ terminal sử dụng lệnh như bên dưới

Với nền tảng Linux hoặc MacOs:
```python
source .env/bin/activate
```

Với nền tảng Windows:
```python
.env\Scripts\activate.bat
```

Khi môi trường ảo hóa đã được kích bạn, ta cần chuyển đổi sang môi trường ảo hóa để cài đặt những python modules cần thiết cho việc xây dựng ứng dụng của chúng ta.
```python
& c:/cdkworkshop/.venv/Scripts/Activate.ps1
pip install -r requirements.txt
```

### 3. Cấu trúc file của một CDK project
Bài lab sử dụng VSCode làm IDE chuẩn để thao tác với ngôn ngữ lập trình python. Trên màn hình Windows Explore, truy cập vào thư mục chứa cdk project C:\cdkworkshop, bấm chuột phải rồi chọn Open with VSCode. 
Một cửa sổ VSCode mở ra với cây thư mục như hình bên dưới: 
![Security Hub](../../../images/1/4.png?width=30pc) 
Trong đó: 
- Thư mục .env - chứa thông tin và môi trường ảo hóa python mà chúng ta đã tạo ở các phần trước. 
- Thư mục cdkworkshop — chứa các Python module.
	- Thư mục: cdkworkshop.egg-info - chứa các thông tin liên quan tới việc đóng gói cdk project 
	- File cdkworkshop_stack.py — chứa cấu trúc tùy chỉnh của CDK stack dùng để xây dựng ứng dụng CDK.
- Thư mục tests — chứa toàn bộ kịch bản kiểm thử trước khi deploy ứng dụng.
	- Thư mục unit — chứa các thành phần của kịch bản kiểm thử.
		- File test_cdkworkshop.py — chứa nội dung của kịch bản test. 
- File app.py — chứa nội dung chính của một ứng dụng mẫu.
- File cdk.json — chứa nội dung cậu hình cho CDK, giúp định nghĩa cách CDK tạo ra cây cấu trúc của ứng dụng CDK. 
- FIle README.md — chứa nội dung hướng dẫn sử dụng và các lệnh hữu ích để bắt đầu làm việc với CDK.
- File requirements.txt — sử dụng pip để cài đặt tất cả các gói phụ thuộc cần thiết cho ứng dụng. Đối với bài lab của chúng ta, nội dung file chỉ chứa -e - có nghĩa các gói phụ thuộc được chỉ rõ trong file setup.py.
- File setup.py — định nghĩa cách mà các gói Python package sẽ được cấu trúc và chỉ ra các gói phụ thuộc đối với ứng dụng. 

Việc triển khai ứng dụng dù đơn giản hay phức tạp thì chủ yếu chúng ta sẽ chỉ làm việc với 2 file quan trọng nhất đó là app.py và file cấu trúc tùy chỉnh của CDK stack - cdkworkshop_stack.py (đối với workshop này)
Nội dung file app.py mô tả hành động khởi tạo thực thể ứng dụng CDK và nạp các đối số vào lớp CdkWorkshopStack nằm trong file cdkworkshop_stack.py. 
```python
#!/usr/bin/env python3

# Thực hiện import toàn bộ thư viện core nằm trong thư viện aws_cdk có đường dẫn .venv\Lib\site-packages\aws_cdk
from aws_cdk import core

# Thực hiện import lớp CdkWorkshopStack nằm trong file cdkworkshop_stack.py thuộc thư mục (hoặc ứng dụng) cdkworkshop
from cdkworkshop.cdkworkshop_stack import CdkWorkshopStack

# Thực hiện khởi tạo thực thể của ứng dụng CDK 
app = core.App()
# Nạp các đối số vào lớp CdkWorkshopStack
CdkWorkshopStack(app, "cdkworkshop", env={'region': 'us-west-2'})
# Chạy lệnh tạo CloudFormation template cho ứng dụng CDK 
app.synth()
```

Nội dung file cdkworkshop_stack.py mô tả đầy đủ thành phần của một ứng dụng CDK bao gồm những gì. Cụ thể trog trường hợp này, ứng dụng của chúng ta tạo ra 2 thực thể từ thư viện CDK đó là 
- SQS Queue (sqs.Queue)
- SNS Topic (sns.Topic)
```python
# Thực hiện import 5 thư viện aws-cdk bao gồm aws_iam, aws_sqs, aws_sns, aws_sns_subscriptions và core
from aws_cdk import (
    aws_iam as iam,
    aws_sqs as sqs,
    aws_sns as sns,
    aws_sns_subscriptions as subs,
    core
)
# Khởi tạo lớp CdkWorkshopStack
class CdkWorkshopStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        queue = sqs.Queue(
            self, "CdkWorkshopQueue",
            visibility_timeout=core.Duration.seconds(300),
        )

        topic = sns.Topic(
            self, "CdkWorkshopTopic"
        )
# Thực hiện cấu hình subcription cho topic, đẩy dữ liệu tới hàng đợi queue
        topic.add_subscription(subs.SqsSubscription(queue))
```
### 4. CDK synth
Các ứng dụng xây dựng bởi AWS CDK thực chất chỉ là một cách định nghĩa cơ sở hạ tầng sử dụng ngôn ngữ dòng lệnh. Khi ứng dụng CDK được thực thi, chúng sẽ tạo ra một AWS CloudFormation template cho mỗi stack được định nghĩa bên trong ứng dụng của bạn. 
Để tổng hợp một ứng dụng CDK, sử dụng lệnh cdk synth. Trường hợp ứng dụng của chúng ta có 2 CDK stack, thì ta sẽ cần liệt kê các stack và chạy cdk synth cho từng stack.
```python
$ cdk ls
cdkworkshop
$
$ cdk synth cdkworkshop
```

Kết quả thu được, ta có một CloudFormation template như bên dưới
```python
Resources:
# Thông tin Queue
  CdkworkshopQueue18864164:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 300
    Metadata:
      aws:cdk:path: cdkworkshop/CdkworkshopQueue/Resource
	  
# Thông tin Chính sách dành cho Queue, cho phép Topic được send message tới Queue
  CdkworkshopQueuePolicy78D5BF45:
    Type: AWS::SQS::QueuePolicy
    Properties:
      PolicyDocument:
        Statement:
          - Action: sqs:SendMessage
            Condition:
              ArnEquals:
                aws:SourceArn:
                  Ref: CdkworkshopTopic58CFDD3D
            Effect: Allow
            Principal:
              Service: sns.amazonaws.com
            Resource:
              Fn::GetAtt:
                - CdkworkshopQueue18864164
                - Arn
        Version: "2012-10-17"
      Queues:
        - Ref: CdkworkshopQueue18864164
    Metadata:
      aws:cdk:path: cdkworkshop/CdkworkshopQueue/Policy/Resource
	  
# Thông tin Subscription giữa Topic và Queue
  CdkworkshopQueuecdkworkshopCdkworkshopTopic7642CC2FCF70B637:
    Type: AWS::SNS::Subscription
    Properties:
      Protocol: sqs
      TopicArn:
        Ref: CdkworkshopTopic58CFDD3D
      Endpoint:
        Fn::GetAtt:
          - CdkworkshopQueue18864164
          - Arn
    Metadata:
      aws:cdk:path: cdkworkshop/CdkworkshopQueue/cdkworkshopCdkworkshopTopic7642CC2F/Resource
	  
# Thông tin Topic
  CdkworkshopTopic58CFDD3D:
    Type: AWS::SNS::Topic
    Metadata:
      aws:cdk:path: cdkworkshop/CdkworkshopTopic/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.18.0,jsii-runtime=Python/3.7.3
```
Như bạn có thể thấy, template bao gồm thông tin của rất nhiều resource như: 
- AWS::SQS::Queue 
- AWS::SNS::Topic 
- AWS::SNS::Subscription - subscription giữa queue và topic
- AWS::SQS::QueuePolicy - chính sách IAM policy cho phép topic được send message tới queue

### 5. CDK Deploy
Okay! Như vậy sau khi kết thúc bước 4, chúng ta đã có CloudFormation template của ứng dụng CDK. Giờ hãy cùng nhau thực hiện triển khai chúng lên AWS Account của bạn. 
Ở lần đầu tiên triển khai một ứng dụng AWS CDK lên môi trường AWS, thì bạn sẽ cần cài đặt “bootstrap stack”. Stack này bao gồm những tài nguyên cần thiết cho hoạt động của CDK toolkit. Ví dụ Stack bao gồm một S3 bucket dùng cho việc lưu trữ các template và các thành phần cần thiết cho quá trình triển khai.
Bạn có thể sử dụng lệnh cdk bootstrap để cài đặt bootstrap stack:
```python
cdk bootstrap

 ⏳  Bootstrapping environment 999999999999/us-east-1...
...
```
Sau khi hoàn tất cài đặt bootstrap stack, ta có thể triển khai ứng dụng CDK bằng lệnh cdk deploy như bên dưới
```python
cdk deploy cdkworkshop

This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬─────────────────────────┬────────┬─────────────────┬───────────────────────────┬─────────────────────────────────────────────────────────┐
│   │ Resource                │ Effect │ Action          │ Principal                 │ Condition                                               │
├───┼─────────────────────────┼────────┼─────────────────┼───────────────────────────┼─────────────────────────────────────────────────────────┤
│ + │ ${CdkworkshopQueue.Arn} │ Allow  │ sqs:SendMessage │ Service:sns.amazonaws.com │ "ArnEquals": {                                          │
│   │                         │        │                 │                           │   "aws:SourceArn": "${CdkworkshopTopic}"                │
│   │                         │        │                 │                           │ }                                                       │
└───┴─────────────────────────┴────────┴─────────────────┴───────────────────────────┴─────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)?
```

Cảnh báo này muốn cho bạn biết rằng việc triển khai ứng dụng tiềm ẩn những rủi ro nhất định. Bởi vì chúng ta cần cho phép Topic được send message tới Queue và chúng ta cũng đã tạo một IAM User và cấp quyền cho User truy cập dịch vụ S3, do vậy ta nhập y để bắt đầu triển khai Stack và khởi tạo các resource. 
Kết quả tiếp theo hiển thị như bên dưới, với ACCOUNT-ID là ID của AWS account của bạn, REGION là ID của Region mà tại đó chúng ta khởi tạo ứng dụng, và STACK-ID là tên định danh duy nhất của Stack:
```python
cdkworkshop: deploying...
cdkworkshop: creating CloudFormation changeset...
 0/6 | 1:31:31 PM | CREATE_IN_PROGRESS   | AWS::CDK::Metadata     | CDKMetadata
 0/6 | 1:31:31 PM | CREATE_IN_PROGRESS   | AWS::SQS::Queue        | CdkworkshopQueue (CdkworkshopQueue18864164)
 0/6 | 1:31:32 PM | CREATE_IN_PROGRESS   | AWS::SNS::Topic        | CdkworkshopTopic (CdkworkshopTopic58CFDD3D)
 0/6 | 1:31:32 PM | CREATE_IN_PROGRESS   | AWS::SQS::Queue        | CdkworkshopQueue (CdkworkshopQueue18864164) Resource creation Initiated
 0/6 | 1:31:32 PM | CREATE_IN_PROGRESS   | AWS::SNS::Topic        | CdkworkshopTopic (CdkworkshopTopic58CFDD3D) Resource creation Initiated
 0/6 | 1:31:33 PM | CREATE_IN_PROGRESS   | AWS::CDK::Metadata     | CDKMetadata Resource creation Initiated
 1/6 | 1:31:33 PM | CREATE_COMPLETE      | AWS::CDK::Metadata     | CDKMetadata
 2/6 | 1:31:33 PM | CREATE_COMPLETE      | AWS::SQS::Queue        | CdkworkshopQueue (CdkworkshopQueue18864164)
 3/6 | 1:31:42 PM | CREATE_COMPLETE      | AWS::SNS::Topic        | CdkworkshopTopic (CdkworkshopTopic58CFDD3D)
 3/6 | 1:31:44 PM | CREATE_IN_PROGRESS   | AWS::SQS::QueuePolicy  | CdkworkshopQueue/Policy (CdkworkshopQueuePolicy78D5BF45)
 3/6 | 1:31:44 PM | CREATE_IN_PROGRESS   | AWS::SNS::Subscription | CdkworkshopQueue/cdkworkshopCdkworkshopTopic7642CC2F (CdkworkshopQueuecdkworkshopCdkworkshopTopic7642CC2FCF70B637)
 3/6 | 1:31:45 PM | CREATE_IN_PROGRESS   | AWS::SQS::QueuePolicy  | CdkworkshopQueue/Policy (CdkworkshopQueuePolicy78D5BF45) Resource creation Initiated
 3/6 | 1:31:45 PM | CREATE_IN_PROGRESS   | AWS::SNS::Subscription | CdkworkshopQueue/cdkworkshopCdkworkshopTopic7642CC2F (CdkworkshopQueuecdkworkshopCdkworkshopTopic7642CC2FCF70B637) Resource creation Initiated
 4/6 | 1:31:45 PM | CREATE_COMPLETE      | AWS::SQS::QueuePolicy  | CdkworkshopQueue/Policy (CdkworkshopQueuePolicy78D5BF45)
 5/6 | 1:31:45 PM | CREATE_COMPLETE      | AWS::SNS::Subscription | CdkworkshopQueue/cdkworkshopCdkworkshopTopic7642CC2F (CdkworkshopQueuecdkworkshopCdkworkshopTopic7642CC2FCF70B637)

 ✅  cdkworkshop

Stack ARN:
arn:aws:cloudformation:us-west-2:************:stack/cdkworkshop/********-****-****-****-************
```

Ứng dụng CDK được triển khai thông qua AWS CloudFormation. Mỗi một CDK Stack sẽ map 1:1 với một CloudFormation Stack. Điều này cũng có nghĩa rằng, bạn có thể sử dụng CloudFormation để quản lý các CDK Stack của bạn. 
Truy cập [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home), bạn sẽ nhìn thấy Stack được triển khai thành công, tương tự như hình bên dưới
![Security Hub](../../../images/1/5.png?width=90pc) 
Lựa chọn cdkworkshop rồi chuyển sang tab Resource để xem các resource đã được tạo ra từ CloudFormation template
![Security Hub](../../../images/1/6.png?width=90pc) 

+++
title = "Ứng dụng CDK đầu tiên - Hello Lambda!"
date = 2020
weight = 3
chapter = false
pre = "<b>3. </b>"
+++

---
Trong chương này, chúng ta sẽ thực sự đi sâu hơn về nội dung code mô tả một ứng dụng CDK. Thay cho những đoạn code SNS/SQS chúng ta sẽ sử dụng Lambda function cùng với API Gateway endpoint dặt ở phía trước để tạo nên ứng dụng CDK đầu tiên. 
Người dùng khi truy cập vào API Gateway endpoint thông qua một URL, sẽ nhận được một lời chào đầy nồng ấm gửi từ Lambda function của chúng ta. 
![Security Hub](../../../images/1/7.png?width=70pc) 
Đầu tiên, hãy cùng dọn dẹp những đoạn code SNS/SQS không còn liên quan tới ứng dụng đầu tiên của chúng ta. 

### 1. Clean up 

The project ban đầu được tạo bởi cdk init sample-app bao gồm một SQS queue và chính sách liên quan tới queue, một SNS topic và subscription của nó. 
Chúng ta sẽ không sử dụng lại chúng trong các phần tiếp theo của workshop này, vì vậy chúng ta sẽ xóa chúng khởi CdkworkshopStack constructor.
Sử dụng VSCode IDE, mở file cdkworkshop/cdkworkshop_stack.py và xóa toàn bộ nội dung không liên quan. Cuối cùng ta thu được nội dung như sau:

```python
from aws_cdk import (
    core,
)


class CdkworkshopStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Nothing here!
```

Sau khi đã chỉnh sửa nội dung của Stack, ta có thể kiểm tra lại sự khác biệt giữa ứng dụng CDK mẫu mà ta deploy ban đầu với nội dung của ứng dụng hiện tại bằng cdk toolkit, cụ thể là lệnh cdk diff. 
Đây là một cách an toàn để kiểm tra điều gì sẽ xảy ra trước khi ta chạy lệnh cdk deploy và đây là một thói quen tốt nên được duy trì. 

```python
cdk diff

Stack cdkworkshop
IAM Statement Changes
┌───┬─────────────────────────────────┬────────┬─────────────────┬───────────────────────────┬─────────────────────────────────────────────────────────────────┐
│   │ Resource                        │ Effect │ Action          │ Principal                 │ Condition                                                       │
├───┼─────────────────────────────────┼────────┼─────────────────┼───────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ - │ ${CdkworkshopQueue18864164.Arn} │ Allow  │ sqs:SendMessage │ Service:sns.amazonaws.com │ "ArnEquals": {                                                  │
│   │                                 │        │                 │                           │   "aws:SourceArn": "${CdkworkshopTopic58CFDD3D}"                │
│   │                                 │        │                 │                           │ }                                                               │
└───┴─────────────────────────────────┴────────┴─────────────────┴───────────────────────────┴─────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[-] AWS::SQS::Queue CdkworkshopQueue18864164 destroy
[-] AWS::SQS::QueuePolicy CdkworkshopQueuePolicy78D5BF45 destroy
[-] AWS::SNS::Subscription CdkworkshopQueuecdkworkshopCdkworkshopTopic7642CC2FCF70B637 destroy
[-] AWS::SNS::Topic CdkworkshopTopic58CFDD3D destroy
```
Và đúng như kì vọng, toàn bộ resource không còn liên quan đều sẽ bị destroy khỏi nội dung của Stack. 
Thực hiện chạy cdk deploy dể áp dụng sự thay đổi trên.
```python
cdk deploy
```

### 2. Hello Lambda

#### Lambda handler code
Chúng ta sẽ bắt đầu với AWS Lambda handler.
- Thực hiện tạo một thư mục đặt tên là lambda nằm trong thư mục root của cây thư mục ứng dụng của bạn.
- Bổ sung một file đặt tên là lambda/hello.py với nội dung nhu sau:

```python
import json

# Hàm bắt sự kiện 
def handler(event, context):
    print('request: {}'.format(json.dumps(event)))
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'text/plain'
        },
        'body': 'Hello, CDK! You have hit {}\n'.format(event['path'])
    }

```
Trên đây chỉ là một Lambda function tương đối đơn giản, nó sẽ trả về nội dung text “Hello, CDK! You’ve hit [url path]" khi người dùng truy cập link endpoint. 
Ngoài ra kết quả trả về cũng bao gồm HTTP status code và thông tin HTTP headers dược tính toán bởi API Gateway.

#### Cài đặt AWS Lambda construct
AWS CDK được cài đặt cùng với bộ thư viện cấu trúc mở rộng AWS Construct Library. Bộ thư viện này được chia thành nhiều module con, và một trong số đó dành cho các dịch vụ của AWS. 
Ví dụ nếu bạn muốn định nghĩa một AWS Lambda function, bạn sẽ cần import thư viện cấu trúc liên quan tới AWS Lambda.
Để có thể tìm hiểu sâu hơn về AWS Construct, bạn có thể truy cập [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html)
![Security Hub](../../../images/1/8.png?width=60pc) 

Okay, giờ chúng ta hãy cùng sử dụng lệnh pip install để bắt đầu cài đặt AWS Lambda module và tất cả các gói phụ thuộc khác vào project:
```python
pip install aws-cdk.aws-lambda
```

#### Thêm AWS Lambda Function vào Stack
Sau khi bộ thư viện đã được cài đặt, quay trở lại file stack cdkworkshop/cdkworkshop_stack.py, thực hiện import thư viện AWS Lambda vừa mới cài đặt, đồng thởi bổ sung thêm đoạn code khởi tạo một thực thể của lambda.Function. 
Nội dung cdkworkshop/cdkworkshop_stack.py lúc này trở thành như sau: 
```python
# Thực hiện import thư viện cdk core và aws_lambda
from aws_cdk import (
    core,
    aws_lambda as _lambda,
)

class CdkworkshopStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Defines an AWS Lambda resource
        my_lambda = _lambda.Function(
            self, 'HelloHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            code=_lambda.Code.asset('lambda'),
            handler='hello.handler',
        )
```
Phân tích: 
- Chúng ta đã import module aws_lambda và gán một tên rút gọn cho nó _lambda
- Function của chúng ta sử dụng bộ python runtime version  3.7 
- Đoạn mã bắt sự kiện Handler được nạp từ thư mục lambda mà chúng ta đã tạo từ trước. Đường dẫn này là tương đối đối với vị trí mà ứng dụng cdk của chúng ta được thực thi (file app.py nằm tại thư mục root)
- Tên của Handler function là hello.handler, trong đó hello là tên của file hello.py và handler là tên của function nằm trong file hello.py

Như bạn có thể thấy Cấu trúc class của cả CdkworkshopStack và lambda.Function (và rất nhiều class khác nữa trong CDK) đều có chung một dạng đối số đầu vào (signature) đó là (scope, id, **kwargs). 
Lý do là bởi tất cả các class này đều là một dạng Cấu trúc hay một dạng Construct. Construct là một khung cấu trúc cơ bản của mọi ứng dụng CDK. 
Chúng giúp thể hiện các thành phần trừu tượng của cloud mà có thể kết hợp lại với nhau để trở thành mức trừu tượng cao hơn thông qua scopes. 
Scopes cũng có thể bao hàm cả construct, và các construct này cũng có thể chứa các construct khác bên trong nó. 

Construct thường được tạo bên trong một scope của một Construct khác và chúng phải có một định danh là duy nhất bên trong scope mà nó được tạo. Do vậy bộ khởi tạo construct (hay còn gọi là constructors) sẽ luôn có các thành phần sau đây:
- scope: là đối số đầu tiên và là nơi construct được tạo ra. Trong hầu hết trường hợp bạn sẽ phải định nghĩa các construct khác bên trong scope của construct hiện tại, điều này có nghĩa ta chỉ cần truyền self cho đối số đầu tiên là đủ.
- id: là đối số thứ 2, định danh cho construct. ID này phải là duy nhất đối với tất cả các construct khác nằm trong cùng một scope. CDK sẽ sủ dụng định dạnh này để tạo ra CloudFormation Logical ID cho mỗi resource được định nghĩa bên trong scope. Tham khảo [CDK user manual](https://docs.aws.amazon.com/cdk/latest/guide/identifiers.html#identifiers_logical_ids) để có thêm thông tin chi tiết về IDs trong CDK.
- kwargs: là đối số cuối cùng (tùy chọn), chúng thường là một tập các đối số khởi tạo và chuyên biệt với từng construct. Ví dụ, với lambda.Function construct thì đối số này là những thông tin như runtime, code and handler. Bạn có thể tìm hiểu thêm về các lựa chọn khác nhau với tính năng auto-complete của IDE hoặc tham khảo tài liệu [online documentation](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-lambda-readme.html).


#### Deploy Ứng dụng
Trước khi thực hiện deploy ứng dụng, việc chúng ta nên làm và làm thường xuyên đó là kiểm tra nhanh sự khác biệt sử dụng lệnh cdk diff
```python
cdk diff cdkworkshop

The cdkworkshop stack uses assets, which are currently not accounted for in the diff output! See https://github.com/awslabs/aws-cdk/issues/395
IAM Statement Changes
┌───┬────────────────────────┬────────┬────────────────────────┬────────────────────────┬───────────┐
│   │ Resource               │ Effect │ Action                 │ Principal              │ Condition │
├───┼────────────────────────┼────────┼────────────────────────┼────────────────────────┼───────────┤
│ + │ ${HelloHandler/Service │ Allow  │ sts:AssumeRole         │ Service:lambda.amazona │           │
│   │ Role.Arn}              │        │                        │ ws.com                 │           │
└───┴────────────────────────┴────────┴────────────────────────┴────────────────────────┴───────────┘
IAM Policy Changes
┌───┬─────────────────────────────┬─────────────────────────────────────────────────────────────────┐
│   │ Resource                    │ Managed Policy ARN                                              │
├───┼─────────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ + │ ${HelloHandler/ServiceRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AWSLambdaBas │
│   │                             │ icExecutionRole                                                 │
└───┴─────────────────────────────┴─────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See http://bit.ly/cdk-2EhF7Np)

Parameters
[+] Parameter HelloHandler/Code/S3Bucket HelloHandlerCodeS3Bucket4359A483: {"Type":"String","Description":"S3 bucket for asset \"hello-cdk-1/HelloHandler/Code\""}
[+] Parameter HelloHandler/Code/S3VersionKey HelloHandlerCodeS3VersionKey07D12610: {"Type":"String","Description":"S3 key for asset version \"hello-cdk-1/HelloHandler/Code\""}
[+] Parameter HelloHandler/Code/ArtifactHash HelloHandlerCodeArtifactHash5DF4E4B6: {"Type":"String","Description":"Artifact hash for asset \"hello-cdk-1/HelloHandler/Code\""}

Resources
[+] AWS::IAM::Role HelloHandler/ServiceRole HelloHandlerServiceRole11EF7C63 
[+] AWS::Lambda::Function HelloHandler HelloHandler2E4FBA4D 
```
Kết quả kiểm tra cho ta thấy, tài nguyên AWS::Lambda::Function sẽ được tạo ra cùng với một loạt CloudFormation parameter được CDK Toolkit sử dụng để truyền tới handler code. 
Sau khi việc kiểm tra hoàn tất, thực hiện deploy ứng dụng cdk deploy. Lệnh không chỉ thực hiện việc triển khai ứng dụng lên AWS Account mà còn thực hiện upload và lưu trữ thư mục lambda từ local lên bootstrap bucket. 

```python
cdk deploy cdkworkshop

```

#### Kiểm thử function của ứng dụng
Truy cập [AWS Lambda Console](https://console.aws.amazon.com/lambda/home#/functions) để bắt đầu kiểm thử function của ứng dụng.
![Security Hub](../../../images/1/9.png?width=90pc) 

Chọn tên của function rồi bấm Test để mở cửa sổ Configure test event:
![Security Hub](../../../images/1/10.png?width=90pc) 

Nội dung Event template list chọn Amazon API Gateway AWS Proxy, rồi nhập tên cho bài kiểm thử bên dưới Event name.
![Security Hub](../../../images/1/11.png?width=90pc) 

Tiếp theo bấm Create, rồi click Test một lần nữa. Chờ cho tới khi quá trình kiểm thử hoàn thành.
Mở rộng phần Details trong bảng Execution result để xem chi tiết và bạn có thể sẽ nhìn một kết quả như mong đợi tương tự bên dưới:
![Security Hub](../../../images/1/12.png?width=90pc) 

### API Gateway
Bước tiếp theo, chúng ta sẽ đi bổ sung API Gateway và đặt nó phía trước Lambda function của chúng ta. API Gateway sẽ phơi ra bên ngoài địa chỉ đầu cuối HTTP công khai để bất cứ ai trên internet đểu cho thể truy cập HTTP client thông qua lệnh curl hoặc qua một web browser.
Lambda proxy được tích hợp với API để mọi request đi tới URL đều được chuyển hướng tới Lambda function, và phản hồi từ Lambda function cũng sẽ tới được với User.

Để cài đặt thư viện cấu trúc API Gateway, ta dùng lệnh pip như sau:

pip install aws-cdk.aws_apigateway

Tiếp theo, bổ sung vào file cdkworkshop_stack.py, thêm định nghĩa API endpoint và liên kết nó với Lambda function:
```python
from aws_cdk import (
    aws_lambda as _lambda,
    aws_apigateway as apigw,
    core,
)

class CdkworkshopStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        my_lambda = _lambda.Function(
            self, 'HelloHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            code=_lambda.Code.asset('lambda'),
            handler='hello.handler',
        )

        apigw.LambdaRestApi(
            self, 'Endpoint',
            handler=my_lambda,
        )
```

Thực hiện chạy lệnh kiểm tra sự khác biệt, có thể thấy các resource sẽ được bổ sung sau khi deploy ứng dụng

```python
cdk diff

The cdkworkshop stack uses assets, which are currently not accounted for in the diff output! See https://github.com/awslabs/aws-cdk/issues/395
IAM Statement Changes
┌───┬────────────────────┬────────┬────────────────────┬────────────────────┬───────────────────────┐
│   │ Resource           │ Effect │ Action             │ Principal          │ Condition             │
├───┼────────────────────┼────────┼────────────────────┼────────────────────┼───────────────────────┤
│ + │ ${Endpoint/CloudWa │ Allow  │ sts:AssumeRole     │ Service:apigateway │                       │
│   │ tchRole.Arn}       │        │                    │ .${AWS::URLSuffix} │                       │
├───┼────────────────────┼────────┼────────────────────┼────────────────────┼───────────────────────┤
│ + │ ${HelloHandler.Arn │ Allow  │ lambda:InvokeFunct │ Service:apigateway │ "ArnLike": {          │
│   │ }                  │        │ ion                │ .amazonaws.com     │   "AWS:SourceArn": "a │
│   │                    │        │                    │                    │ rn:${AWS::Partition}: │
│   │                    │        │                    │                    │ execute-api:us-east-2 │
│   │                    │        │                    │                    │ :${AWS::AccountId}:${ │
│   │                    │        │                    │                    │ Endpoint}/${EndpointD │
│   │                    │        │                    │                    │ eploymentStageprodB78 │
│   │                    │        │                    │                    │ BEEA0}/*/"            │
│   │                    │        │                    │                    │ }                     │
│ + │ ${HelloHandler.Arn │ Allow  │ lambda:InvokeFunct │ Service:apigateway │ "ArnLike": {          │
│   │ }                  │        │ ion                │ .amazonaws.com     │   "AWS:SourceArn": "a │
│   │                    │        │                    │                    │ rn:${AWS::Partition}: │
│   │                    │        │                    │                    │ execute-api:us-east-2 │
│   │                    │        │                    │                    │ :${AWS::AccountId}:${ │
│   │                    │        │                    │                    │ Endpoint}/test-invoke │
│   │                    │        │                    │                    │ -stage/*/"            │
│   │                    │        │                    │                    │ }                     │
│ + │ ${HelloHandler.Arn │ Allow  │ lambda:InvokeFunct │ Service:apigateway │ "ArnLike": {          │
│   │ }                  │        │ ion                │ .amazonaws.com     │   "AWS:SourceArn": "a │
│   │                    │        │                    │                    │ rn:${AWS::Partition}: │
│   │                    │        │                    │                    │ execute-api:us-east-2 │
│   │                    │        │                    │                    │ :${AWS::AccountId}:${ │
│   │                    │        │                    │                    │ Endpoint}/${EndpointD │
│   │                    │        │                    │                    │ eploymentStageprodB78 │
│   │                    │        │                    │                    │ BEEA0}/*/{proxy+}"    │
│   │                    │        │                    │                    │ }                     │
│ + │ ${HelloHandler.Arn │ Allow  │ lambda:InvokeFunct │ Service:apigateway │ "ArnLike": {          │
│   │ }                  │        │ ion                │ .amazonaws.com     │   "AWS:SourceArn": "a │
│   │                    │        │                    │                    │ rn:${AWS::Partition}: │
│   │                    │        │                    │                    │ execute-api:us-east-2 │
│   │                    │        │                    │                    │ :${AWS::AccountId}:${ │
│   │                    │        │                    │                    │ Endpoint}/test-invoke │
│   │                    │        │                    │                    │ -stage/*/{proxy+}"    │
│   │                    │        │                    │                    │ }                     │
└───┴────────────────────┴────────┴────────────────────┴────────────────────┴───────────────────────┘
IAM Policy Changes
┌───┬────────────────────────────┬──────────────────────────────────────────────────────────────────┐
│   │ Resource                   │ Managed Policy ARN                                               │
├───┼────────────────────────────┼──────────────────────────────────────────────────────────────────┤
│ + │ ${Endpoint/CloudWatchRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AmazonAPIGate │
│   │                            │ wayPushToCloudWatchLogs                                          │
└───┴────────────────────────────┴──────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See http://bit.ly/cdk-2EhF7Np)

Resources
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.ANY.. HelloHandlerApiPermissionANYAC4E141E 
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.Test.ANY.. HelloHandlerApiPermissionTestANYDDD56D72 
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.ANY..{proxy+} HelloHandlerApiPermissionANYproxy90E90CD6 
[+] AWS::Lambda::Permission HelloHandler/ApiPermission.Test.ANY..{proxy+} HelloHandlerApiPermissionTestANYproxy9803526C 
[+] AWS::ApiGateway::RestApi Endpoint EndpointEEF1FD8F 
[+] AWS::ApiGateway::Deployment Endpoint/Deployment EndpointDeployment318525DAb462c597ccb914d9fc1c10f664ed81ca 
[+] AWS::ApiGateway::Stage Endpoint/DeploymentStage.prod EndpointDeploymentStageprodB78BEEA0 
[+] AWS::IAM::Role Endpoint/CloudWatchRole EndpointCloudWatchRoleC3C64E0F 
[+] AWS::ApiGateway::Account Endpoint/Account EndpointAccountB8304247 
[+] AWS::ApiGateway::Resource Endpoint/Default/{proxy+} Endpointproxy39E2174E 
[+] AWS::ApiGateway::Method Endpoint/Default/{proxy+}/ANY EndpointproxyANYC09721C5 
[+] AWS::ApiGateway::Method Endpoint/Default/ANY EndpointANY485C938B 

Outputs
[+] Output Endpoint/Endpoint Endpoint8024A810: {"Value":{"Fn::Join":["",["https://",{"Ref":"EndpointEEF1FD8F"},".execute-api.us-east-2.",{"Ref":"AWS::URLSuffix"},"/",{"Ref":"EndpointDeploymentStageprodB78BEEA0"},"/"]]}}
```

Sau khi hoàn tất việc kiểm tra, thực hiện deploy ứng dụng 
```python
cdk deploy
```
Kết quả thu được đường dẫn là API Gateway endpoint URL tương tự như bên dưới

```python
CdkWorkshopStack.Endpoint8024A810 = https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

Thực hiện kiểm tra kết quả đầu ra với lệnh curl 
```python
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```
Bạn sẽ nhận được output 
```python
Hello, CDK! You've hit /
```
Hoặc kiểm tra sử dụng web browser
![Security Hub](../../../images/1/13.png?width=90pc) 

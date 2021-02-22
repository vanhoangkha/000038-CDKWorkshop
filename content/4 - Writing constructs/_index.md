+++
title = "Cách Xây dựng một Construct"
date = 2020
weight = 4
chapter = false
pre = "<b>4. </b>"
+++

---
Trong chương này, chúng ta sẽ định nghĩa một Construct mới có tên là HitCounter. Cấu trúc mới có thể gắn với bất cứ Lambda function để chúng được sử dụng như một API Gateway backend,  và chúng sẽ đếm số lượng request được gửi tới API Gateway Endpoint URL rồi lưu thông tin đó vào bảng DynamoDB.
![Security Hub](../../../images/1/14.png?width=70pc) 

### Định nghĩa HitCounter API
Tạo file cdkworkshop/hitcounter.py với nội dung như bên dưới: 
```python
from aws_cdk import (
    aws_lambda as _lambda,
    core
)

class HitCounter(core.Construct):

    def __init__(self, scope: core.Construct, id: str, downstream: _lambda.IFunction, **kwargs):
        super().__init__(scope, id, **kwargs)

        # TODO
```
Phân tích: 
- Chúng ta đã khởi tạo một construct class mới có tên là HitCounter
- Các đối số của constructor vẫn bao gồm scope, id và kwargs, và ta truyền chúng tới lớp cơ sở cdk.construct
- Lớp HitCounter có thêm một đối số có tên downstream thuộc _lambda.IFunction - đối số này gắn với Lambda function mà chúng ta mới tạo ở phía trên.

### Hit counter Lambda handler
Để Lambda function có thể xử lý sự kiện, đếm số lần request và lưu vào bảng DynamoDB thì ứng dụng phải được cài đặt AWS DynamoDB module. Sử dụng lệnh pip như bên dưới: 
```python
pip install aws-cdk.aws_dynamodb
```
Tiếp theo, tạo file lambda/hitcount.py bên trong thư mục lambda với nội dụng như sau:
```python
import json
import os

import boto3
# Tương tự như import, ở đây ta khai báo thực thể đại diện cho các resource như dynamodb và lambda service
ddb = boto3.resource('dynamodb')
table = ddb.Table(os.environ['HITS_TABLE_NAME'])
_lambda = boto3.client('lambda')


def handler(event, context):
    print('request: {}'.format(json.dumps(event)))
    table.update_item(
        Key={'path': event['path']},
        UpdateExpression='ADD hits :incr',
        ExpressionAttributeValues={':incr': 1}
    )

    resp = _lambda.invoke(
        FunctionName=os.environ['DOWNSTREAM_FUNCTION_NAME'],
        Payload=json.dumps(event),
    )

    body = resp['Payload'].read()

    print('downstream response: {}'.format(body))
    return json.loads(body)
```
Phân tích: 
- Bảng lưu thông tin số lần request gửi tới URL có tên là HITS_TABLE_NAME
- Hàm bắt sự kiện Handler, mỗi lần có request đi tới URL thì sẽ cập nhật giá trị của trường hits lên 1
- Hàm trả về thông tin phản hồi có tên DOWNSTREAM_FUNCTION_NAME

### Định nghĩa các Resource
Trở lại file cdkworkshop/hitcounter.py mà ta đã tạo ở đầu chương, bổ sung nội dung tạo thêm Lambda function để xử lý bắt sự kiện và bảng DynamoDB lưu số lần request. 
```python
from aws_cdk import (
    aws_lambda as _lambda,
    aws_dynamodb as ddb,
    core,
)

class HitCounter(core.Construct):

    @property
    def handler(self):
        return self._handler    

    def __init__(self, scope: core.Construct, id: str, downstream: _lambda.IFunction, **kwargs):
        super().__init__(scope, id, **kwargs)

        table = ddb.Table(
            self, 'Hits',
            partition_key={'name': 'path', 'type': ddb.AttributeType.STRING}
        )

        self._handler = _lambda.Function(
            self, 'HitCountHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            handler='hitcount.handler',
            code=_lambda.Code.asset('lambda'),
            environment={
                'DOWNSTREAM_FUNCTION_NAME': downstream.function_name,
                'HITS_TABLE_NAME': table.table_name,
            }
        )
```
Phân tích:
- Chúng ta đã định nghĩa một bảng DynamoDB với giá trị path của URL làm partition key (mọi bảng DynamoDB đều phải có một partition key).
- Chúng ta đã định nghĩa một Lambda function xử lý sự kiện sử dụng lambda/hitcount.handler, trong đó lambdal là tên thư mục, hitcount là rút gọn của tên file hitcount.py, còn lại handler là tên hàm xử lý bên trong file hitcount.py.
- Chúng ta đã buộc cái biến môi trường của Lambda với function_name and table_name tương ứng với các resources.
Các thuộc tính function_name và table_name là những giá trị mà chỉ được phân giải khi chúng ta deploy ứng dụng (lưu ý rằng chúng ta không cấu hình tên cho table/function, tất cả đều là logical IDs). 
Điều này có nghĩa là nếu bạn cố gắng in giá trị của chúng ra trong quá trình tổng hợp, bạn sẽ chỉ nhận được một "TOKEN", 

### Sử dụng HitCounter
Như vậy function HitCounter của chúng ta đã sẵn sàng, giờ chúng ta sẽ đưa chúng vào ứng dụng của chúng ta. Mở lại file cdkworkshop_stack.py và sửa lại nội dung theo như bên dưới: 
```python
from aws_cdk import (
    core,
    aws_lambda as _lambda,
    aws_apigateway as apigw,
)

from hitcounter import HitCounter


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

        hello_with_counter = HitCounter(
            self, 'HelloHitCounter',
            downstream=my_lambda,
        )

        apigw.LambdaRestApi(
            self, 'Endpoint',
            handler=hello_with_counter.handler,
        )
```
Nội dung cơ bản có thể được hiểu là bất cứ khi nào có request tới Endpoint URL, API Gateway sẽ chuyển request tới hàm bắt sự kiện hit counter handler. 
Tại đây nó sẽ log lại thông tin request và chuyển tới my_lambda function. Cuối cùng nội dung phản hồi sẽ được chuyển tiếp theo thứ tự ngược lại tới User.

Sau khi đã chỉnh sửa lại nội dung cho file cdkworkshop_stack.py, thực hiện deploy
```python
cdk deploy
```
Ta thu được Endpoint URL như sau: 
```python
cdkworkshop.Endpoint8024A810 = https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```
Mặc dù việc triển khai không xảy ra lỗi, nhưng khi thử kiểm tra truy cập tới URL sử dụng lệnh curl, ta sẽ gặp phải lỗi
```python
curl -i https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/

HTTP/1.1 502 Bad Gateway
...

{"message": "Internal server error"}
```
Hãy cùng tìm hiểu xem điều gì đã xảy ra và cách khắc phục chúng với CloudWatch nhé.

### CloudWatch Logs
Việc đầu tiên chúng ta cần làm khi gặp bất cứ vấn đề gì về truy cập đó là kiểm tra logs của CloudWatch, tìm kiếm những dòng nhật kí liên quan tới hit counter.
Còn một vài công cụ khác cũng có thể giúp bạn kiểm tra thông tin về như SAM CLI hay awslogs. Trong phạm vi workshop này, chúng tôi sẽ chỉ cho các bạn cách để tìm kiếm thông tin xoay quanh lỗi xảy đến với ứng dụng của bạn với AWS CloudWatch.
- Truy cập [AWS Lambda console](https://console.aws.amazon.com/lambda/home) (đảm bảo bạn đang ở đúng region).
- Chọn HitCounter Lambda function (tên function có chứa chuỗi HelloHitCounterHitCountHandler): 
![Security Hub](../../../images/1/15.png?width=90pc) 
- Chuyển sang tab Monitoring
![Security Hub](../../../images/1/16.png?width=60pc) 
- Bấm vào View Logs in CloudWatch để mở cửa sổ AWS CloudWatch console. 
![Security Hub](../../../images/1/17.png?width=90pc) 
- Chọn log group gấn với thời điểm hiện tại nhất rồi tiếp kiếm message có chứa cụm “errorMessage”. Ví dụ như dưới đây: 
```python
{
    "errorMessage": "User: arn:aws:sts::585695036304:assumed-role/hello-cdk-1-HelloHitCounterHitCounterHandlerS-TU5M09L1UBID/hello-cdk-1-HelloHitCounterHitCounterHandlerD-144HVUNEWRWEO is not authorized to perform: dynamodb:UpdateItem on resource: arn:aws:dynamodb:us-east-1:585695036304:table/hello-cdk-1-HelloHitCounterHits7AAEBF80-1DZVT3W84LJKB",
    "errorType": "AccessDeniedException",
    "stackTrace": [
        "Request.extractError (/var/runtime/node_modules/aws-sdk/lib/protocol/json.js:48:27)",
        "Request.callListeners (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:105:20)",
        "Request.emit (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:77:10)",
        "Request.emit (/var/runtime/node_modules/aws-sdk/lib/request.js:683:14)",
        "Request.transition (/var/runtime/node_modules/aws-sdk/lib/request.js:22:10)",
        "AcceptorStateMachine.runTo (/var/runtime/node_modules/aws-sdk/lib/state_machine.js:14:12)",
        "/var/runtime/node_modules/aws-sdk/lib/state_machine.js:26:10",
        "Request.<anonymous> (/var/runtime/node_modules/aws-sdk/lib/request.js:38:9)",
        "Request.<anonymous> (/var/runtime/node_modules/aws-sdk/lib/request.js:685:12)",
        "Request.callListeners (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:115:18)"
    ]
}
```
Lỗi này cho ta biết được rằng, có vẻ như Lambda function không thể ghi dữ liệu vào bảng DynamoDB, do vậy ta cần phải bổ sung permission cho phép Lambda được phép đọc ghi vào bảng DynamoDB.

### Cấp quyền thao tác DynamoDB cho Lambda function
Truy cập lại file hitcounter.py và chỉnh sửa lại nội dung theo như bên dưới đây: 
```python
from aws_cdk import (
    aws_lambda as _lambda,
    aws_dynamodb as ddb,
    core,
)

class HitCounter(core.Construct):

    @property
    def handler(self):
        return self._handler    

    def __init__(self, scope: core.Construct, id: str, downstream: _lambda.IFunction, **kwargs):
        super().__init__(scope, id, **kwargs)

        table = ddb.Table(
            self, 'Hits',
            partition_key={'name': 'path', 'type': ddb.AttributeType.STRING}
        )

        self._handler = _lambda.Function(
            self, 'HitCountHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            handler='hitcount.handler',
            code=_lambda.Code.asset('lambda'),
            environment={
                'DOWNSTREAM_FUNCTION_NAME': downstream.function_name,
                'HITS_TABLE_NAME': table.table_name,
            }
        )
# Cấp quyền đọc ghi bảng dữ liệu cho Lambda function
        table.grant_read_write_data(self.handler)
```

Sau khi đã cấp quyền đọc ghi DynamoDB table cho Lambda function, thực hiện deploy lại stack  
```python
cdk deploy
```
Thực hiện kiểm tra truy cập dịch vụ với lệnh curl
```php
curl -i https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/

HTTP/1.1 502 Bad Gateway
...

{"message": "Internal server error"}
```
Tuy nhiên như ta thấy, vẫn có lỗi ở đâu khiến cho ứng dụng vẫn chưa hoạt động được. 
Thực hiện lại các bước kiểm tra logs với CloudWatch như đã làm ở phía trên
```python
{
    "errorMessage": "User: arn:aws:sts::585695036304:assumed-role/hello-cdk-1-HelloHitCounterHitCounterHandlerS-TU5M09L1UBID/hello-cdk-1-HelloHitCounterHitCounterHandlerD-144HVUNEWRWEO is not authorized to perform: lambda:InvokeFunction on resource: arn:aws:lambda:us-east-1:585695036304:function:hello-cdk-1-HelloHandler2E4FBA4D-149MVAO4969O7",
    "errorType": "AccessDeniedException",
    "stackTrace": [
        "Object.extractError (/var/runtime/node_modules/aws-sdk/lib/protocol/json.js:48:27)",
        "Request.extractError (/var/runtime/node_modules/aws-sdk/lib/protocol/rest_json.js:52:8)",
        "Request.callListeners (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:105:20)",
        "Request.emit (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:77:10)",
        "Request.emit (/var/runtime/node_modules/aws-sdk/lib/request.js:683:14)",
        "Request.transition (/var/runtime/node_modules/aws-sdk/lib/request.js:22:10)",
        "AcceptorStateMachine.runTo (/var/runtime/node_modules/aws-sdk/lib/state_machine.js:14:12)",
        "/var/runtime/node_modules/aws-sdk/lib/state_machine.js:26:10",
        "Request.<anonymous> (/var/runtime/node_modules/aws-sdk/lib/request.js:38:9)",
        "Request.<anonymous> (/var/runtime/node_modules/aws-sdk/lib/request.js:685:12)"
    ]
}
```
Có thể thấy rằng nội dung lỗi đã thay đổi, cụ thể ở 
```python
User: <VERY-LONG-STRING> is not authorized to perform: lambda:InvokeFunction on resource: <VERY-LONG-STRING>"
```
Có vẻ như HitCounter function của chúng ta đã có quyền ghi vào database của DynamoDB, chúng ta có thể kiểm chứng điều này bằng cách truy cập [DynamoDB console](https://console.aws.amazon.com/dynamodb/home)
![Security Hub](../../../images/1/18.png?width=50pc) 
Tuy nhiên HitCounter function vẫn chưa có quyền gọi tới downstream function. Để cấp quyền cho thao tác này, thực hiện chỉnh sửa nội dung file hitcounter.py theo như bên dưới đây
```python
from aws_cdk import (
    aws_lambda as _lambda,
    aws_dynamodb as ddb,
    core,
)

class HitCounter(core.Construct):

    @property
    def handler(self):
        return self._handler    

    def __init__(self, scope: core.Construct, id: str, downstream: _lambda.IFunction, **kwargs):
        super().__init__(scope, id, **kwargs)

        table = ddb.Table(
            self, 'Hits',
            partition_key={'name': 'path', 'type': ddb.AttributeType.STRING}
        )

        self._handler = _lambda.Function(
            self, 'HitCountHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            handler='hitcount.handler',
            code=_lambda.Code.asset('lambda'),
            environment={
                'DOWNSTREAM_FUNCTION_NAME': downstream.function_name,
                'HITS_TABLE_NAME': table.table_name,
            }
        )

        table.grant_read_write_data(self.handler)
        downstream.grant_invoke(self.handler)
```

Sau đó thực hiện kiểm tra lại nội dung đã thay đổi so với lần deploy trước sử dụng lệnh cdk diff
```python
cdk diff
Stack cdkworkshop
The cdkworkshop stack uses assets, which are currently not accounted for in the diff output! See https://github.com/awslabs/aws-cdk/issues/395
IAM Statement Changes
┌───┬────────────────────────┬────────┬────────────────────────┬────────────────────────┬───────────┐
│   │ Resource               │ Effect │ Action                 │ Principal              │ Condition │
├───┼────────────────────────┼────────┼────────────────────────┼────────────────────────┼───────────┤
│ + │ ${HelloHandler.Arn}    │ Allow  │ lambda:InvokeFunction  │ AWS:${HelloHitCounter/ │           │
│   │                        │        │                        │ HitCounterHandler/Serv │           │
│   │                        │        │                        │ iceRole}               │           │
└───┴────────────────────────┴────────┴────────────────────────┴────────────────────────┴───────────┘
(NOTE: There may be security-related changes not in this list. See http://bit.ly/cdk-2EhF7Np)

Resources
[~] AWS::IAM::Policy HelloHitCounter/HitCounterHandler/ServiceRole/DefaultPolicy HelloHitCounterHitCounterHandlerServiceRoleDefaultPolicy1487A60A 
 └─ [~] PolicyDocument
     └─ [~] .Statement:
         └─ @@ -24,5 +24,15 @@
            [ ]         "Ref": "AWS::NoValue"
            [ ]       }
            [ ]     ]
            [+]   },
            [+]   {
            [+]     "Action": "lambda:InvokeFunction",
            [+]     "Effect": "Allow",
            [+]     "Resource": {
            [+]       "Fn::GetAtt": [
            [+]         "HelloHandler2E4FBA4D",
            [+]         "Arn"
            [+]       ]
            [+]     }
            [ ]   }
            [ ] ]
```
Nhìn vào phần Resource có thể thấy IAM policy mới đã được add vào role của HitCounter function
Thực hiện deploy lại Stack 
```python
cdk deploy
```
Rồi kiểm tra lại khả năng truy cập dịch vụ bằng lệnh curl 
```php
curl -i https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/

HTTP/1.1 200 OK
...

Hello, CDK! You've hit /
```
Và tuyệt vời! Dịch vụ đã cho phép truy cập và trả về nội dung như ta mong muốn!

### Kiểm thử bộ đếm của HitCounter
Trên máy Client, thực hiện gửi một loạt request tới Endpoint URL với nhiều đường dẫn khác nhau sử dụng lệnh curl hoặc web browser
```python
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hello
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hello/world
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hello/world
```
Truy cập [DynamoDB console](https://console.aws.amazon.com/dynamodb/home), chọn Tables, rồi chọn tên bảng bắt đầu bằng cdkworkshop-HelloHitCounterHits. 
Chuyển sang tab Items, ta có thể thấy số lượng hits ứng với mỗi path như hình bên dưới
![Security Hub](../../../images/1/19.png?width=50pc) 
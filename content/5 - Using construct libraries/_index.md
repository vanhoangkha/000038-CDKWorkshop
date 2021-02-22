+++
title = "Sử dụng thư viện Construct"
date = 2020
weight = 5
chapter = false
pre = "<b>5. </b>"
+++

---
Trong chương này, chúng ta sẽ đi tìm hiểu cách ứng dụng sử dụng thư viện construct có sẵn là như thế nào. Để dễ hình dung hơn, ta sẽ thực hiện import một thư viện construct có tên [cdk-dynamo-table-viewer](https://www.npmjs.com/package/cdk-dynamo-table-viewer) vào ứng dụng cdkworkshop của chúng ta và cài đặt nó lên bảng Hit Counter được tạo ở phần trước. 
![Security Hub](../../../images/1/20.png?width=90pc) 
Để table viewer có thể tích hợp được với ứng dụng bạn cần phải cài đặt python module tương ứng với nó. Sử dụng lệnh pip dể cài đặt như sau: 
```python
pip install cdk-dynamo-table-viewer

Installing collected packages: cdk-dynamo-table-viewer
Successfully installed cdk-dynamo-table-viewer-3.0.3
```
Trở lại file cdkworkshop_stack.py, rồi thực hiện bổ sung construct TableViewer. Lúc này đoạn nội dung stack sẽ tương tự bên dưới: 
```python
from aws_cdk import (
    core,
    aws_lambda as _lambda,
    aws_apigateway as apigw,
)
# Bổ sung thư viện construct TableViewer
from cdk_dynamo_table_viewer import TableViewer
from hitcounter import HitCounter


class CdkworkshopStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Defines an AWS Lambda resource
        hello = _lambda.Function(
            self, 'HelloHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            code=_lambda.Code.asset('lambda'),
            handler='hello.handler',
        )

        hello_with_counter = HitCounter(
            self, 'HelloHitCounter',
            downstream=hello,
        )

        apigw.LambdaRestApi(
            self, 'Endpoint',
            handler=hello_with_counter.handler,
        )
# Gọi tới construct TableViewer. Tên bảng tạm thời bỏ trống. 
        TableViewer(
            self, 'ViewHitCounter',
            title='Hello Hits',
            table=??????
        )  
```

Như vậy TableViewer yêu cầu bạn phải xác định giá trị cho thuộc tính table để truyền vào construct làm đối số. Bằng cách nào đó chúng ta phải truy cập được DynamoDB table đặt sau HitCounter function.
Tuy nhiên với API hiện tại của HitCounter thì điều này là không thể, do vậy chúng ta phải chỉnh sửa lại hitcounter.py để thông tin DynamoDB table có thể trở nên công khai. Nội dung file hitcounter.py sau khi chỉnh sửa như sau: 
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
		
# Định nghĩa thêm hàm trả về thông tin của bảng 
    @property
    def table(self):
        return self._table

    def __init__(self, scope: core.Construct, id: str, downstream: _lambda.IFunction, **kwargs):
        super().__init__(scope, id, **kwargs)

        self._table = ddb.Table(
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
                'HITS_TABLE_NAME': self._table.table_name,
            }
        )

        self._table.grant_read_write_data(self.handler)
        downstream.grant_invoke(self.handler)
```
Đồng thời, trên file cấu hình stack thực hiện nạp đối số tương ứng chứa tên của bảng Hit Counter. 
```python
from aws_cdk import (
    core,
    aws_lambda as _lambda,
    aws_apigateway as apigw,
)

from cdk_dynamo_table_viewer import TableViewer
from hitcounter import HitCounter


class CdkworkshopStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Defines an AWS Lambda resource
        hello = _lambda.Function(
            self, 'HelloHandler',
            runtime=_lambda.Runtime.PYTHON_3_7,
            code=_lambda.Code.asset('lambda'),
            handler='hello.handler',
        )

        hello_with_counter = HitCounter(
            self, 'HelloHitCounter',
            downstream=hello,
        )

        apigw.LambdaRestApi(
            self, 'Endpoint',
            handler=hello_with_counter.handler,
        )
# Truyền đối số tương ứng với tên bảng, gán cho thuộc tính table
        TableViewer(
            self, 'ViewHitCounter',
            title='Hello Hits',
            table=hello_with_counter.table,
        )  
```

Sau khi hoàn tất việc bổ sung construct TableViewer vào ứng dụng, thực hiện kiểm tra khác biệt trước khi deploy ứng dụng:
```python
cdk diff
Resources
[+] AWS::IAM::Role ViewHitCounter/Rendered/ServiceRole ViewHitCounterRenderedServiceRole254DB4EA 
[+] AWS::IAM::Policy ViewHitCounter/Rendered/ServiceRole/DefaultPolicy ViewHitCounterRenderedServiceRoleDefaultPolicy9ADB8C83 
[+] AWS::Lambda::Function ViewHitCounter/Rendered ViewHitCounterRendered9C783E45 
[+] AWS::Lambda::Permission ViewHitCounter/Rendered/ApiPermission.ANY.. ViewHitCounterRenderedApiPermissionANY72263B1A 
[+] AWS::Lambda::Permission ViewHitCounter/Rendered/ApiPermission.Test.ANY.. ViewHitCounterRenderedApiPermissionTestANYA4794B81 
[+] AWS::Lambda::Permission ViewHitCounter/Rendered/ApiPermission.ANY..{proxy+} ViewHitCounterRenderedApiPermissionANYproxy42B9E676 
[+] AWS::Lambda::Permission ViewHitCounter/Rendered/ApiPermission.Test.ANY..{proxy+} ViewHitCounterRenderedApiPermissionTestANYproxy104CA88E 
[+] AWS::ApiGateway::RestApi ViewHitCounter/ViewerEndpoint ViewHitCounterViewerEndpoint5A0EF326 
[+] AWS::ApiGateway::Deployment ViewHitCounter/ViewerEndpoint/Deployment ViewHitCounterViewerEndpointDeployment1CE7C5761d44312e8424c23ba090a70e0962c36f 
[+] AWS::ApiGateway::Stage ViewHitCounter/ViewerEndpoint/DeploymentStage.prod ViewHitCounterViewerEndpointDeploymentStageprodF3901FC7 
[+] AWS::IAM::Role ViewHitCounter/ViewerEndpoint/CloudWatchRole ViewHitCounterViewerEndpointCloudWatchRole87B94D6A 
[+] AWS::ApiGateway::Account ViewHitCounter/ViewerEndpoint/Account ViewHitCounterViewerEndpointAccount0B75E76A 
[+] AWS::ApiGateway::Resource ViewHitCounter/ViewerEndpoint/Default/{proxy+} ViewHitCounterViewerEndpointproxy2F4C239F 
[+] AWS::ApiGateway::Method ViewHitCounter/ViewerEndpoint/Default/{proxy+}/ANY ViewHitCounterViewerEndpointproxyANYFF4B8F5B 
[+] AWS::ApiGateway::Method ViewHitCounter/ViewerEndpoint/Default/ANY ViewHitCounterViewerEndpointANY66F2285B 
```
Quan sát kĩ ta sẽ thấy TableViewer đã thêm vào hàng loạt Resource mới bao gồm các API Gateway Endpoint, một Lambda function, các permission và role mới. 
Nó cho thấy một tín hiệu tốt và sẵn sàng cho việc deploy ứng dụng. Sử dụng lệnh cdk deploy để triển khai ứng dụng lên AWS Account của bạn: 
```python
cdk deploy
...
cdkworkshop.ViewHitCounterViewerEndpointCA1B1E4B = https://6i4udz9wb2.execute-api.us-east-2.amazonaws.com/prod/
```
Thực hiện truy cập URL bằng web browser, bạn sẽ thấy một bảng tương tự như bên dưới: 
![Security Hub](../../../images/1/21.png?width=50pc) 

Khi ta tạo thêm một vài request tới URL, bạn sẽ thấy giá trị của cột hits sẽ tăng lên theo thời gian thực. Bạn có thể sử dụng lệnh curl để tự mình trải nghiệm ứng dụng khá thú vị này. 
```python
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hoooot
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hoooot
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hoooot
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hoooot
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/hit1
```
+++
title = "AWS CDK Intro Workshop"
date = 2020
weight = 1
chapter = false
+++

# AWS CDK INTRO WORKSHOP
---
AWS CDK là một framework phát triển phần mềm mới của AWS, hỗ trợ các developer trong việc sử dụng ngôn ngữ lập trình yêu thích đối với họ để xây dựng cơ sở hạ tầng trên Cloud một cách dễ dàng và sau đó triển khai chúng bằng AWS CloudFormation.  

Ở workshop lần này tôi và các bạn sẽ cùng nhau học cách thiết lập môi trường phát triển và tìm hiểu một chút về cách làm việc với Bộ công cụ CDK để triển khai ứng dụng lên môi trường AWS.  

Sau đó chúng ta sẽ bắt tay vào viết một ứng dụng đơn giản đầu tiên, sử dụng một chút hàm Lambda kiểu “Hello, world” đặt sau một API Gateway để người dùng có thể gọi tới chúng qua phương thức HTTP.

Phấn tiếp theo, chúng ta sẽ đi tìm hiểu về một khái niệm của CDK mà sẽ được nhắc tới rất nhiều sau này đó là CDK Construct. Các Construct cho phép bạn gộp một loạt cơ sở hạ tầng thành các thành phần có thể tái sử dụng mà bất kỳ ai cũng có thể đưa chúng vào ứng dụng của riêng họ. 

Đồng thời chúng tôi cũng sẽ hướng dẫn bạn cách để tự viết một Contruct cho riêng mình.

Phần cuối cùng, chúng tôi sẽ chỉ cho bạn cách sử dụng một Contruct từ thư viện đã được đóng gói sẵn để xây dựng stack ứng dụng của bạn.

Kết thúc AWS CDK Intro workshop, bạn sẽ có thể: 
- Tạo một ứng dụng CDK hoàn chỉnh
- Định nghĩa cơ sở hạ tầng cho ứng dụng sử dụng thư viện AWS Construct Library
- Triển khai các ứng dụng CDK
- Định nghĩa các Construct do chính bạn định nghĩa và hướng dẫn bạn cách tái sử dụng chúng
- Sử dụng các Contruct được chia sẻ bởi cộng đồng

#### Tài liệu

1. Tài liệu nguồn chính thức [AWS CDK Intro Workshop](https://cdkworkshop.com/)
2. Tài liệu tham khảo [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html) 
3. Tài liệu tham khảo [AWS Lambda](https://aws.amazon.com/vi/lambda/)
4. Tài liệu tham khảo [AWS CloudFormation](https://aws.amazon.com/vi/cloudformation/)


#### Nội dung

Nội dung bài lab gồm 5 phần:

1. [Chuẩn bị](http://localhost:1313/1-preparation/)
2. [Khởi tạo một CDK Project](http://localhost:1313/2-new-project/)
3. [Ứng dụng CDK đầu tiên - Hello Lambda!](http://localhost:1313/3-hello-cdk/)
4. [Cách Xây dựng một Construct](http://localhost:1313/4-writing-constructs/)
5. [Sử dụng thư viện Construct](http://localhost:1313/5-using-construct-libraries/)




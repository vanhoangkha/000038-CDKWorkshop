+++
title = "Chuẩn bị"
date = 2020
weight = 1
chapter = false
pre = "<b>1. </b>"
+++

---
Để đảm bảo follow up workshop một cách chính xác nhất, ta cần chuẩn bị những nội dung sau đây

### 1. AWS CLI

AWS CLI cho phép bạn tương tác với các dịch vụ AWS từ một phiên terminal. Để tránh xảy ra bug trong quá trình thao tác, chúng tôi khuyên bạn nên cài đặt phiên bản AWS CLI mới nhất. 

- Windows: [MSI installer](https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html#install-msi-on-windows)
- Linux, macOS hoặc Unix: [Bundled installer](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)

Truy cập trang [AWS Command Line Interface installation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) để có thêm thông tin chi tiết.


### 2. AWS Account & User
Để triển khai được ứng dụng yêu cầu bạn phải có một AWS Account với User có quyền truy cập và sử dụng toàn bộ dịch vụ AWS - tương đương với quyền quán trị của Administrator.  
Nếu bạn chưa có AWS Account, thực hiện tạo tài khoản [ở đây](https://portal.aws.amazon.com/billing/signup)  

Để tạo một User có quyền quản trị, bạn có thể tham khảo các bước như bên dưới:

- Bước 1: Đăng nhập vào AWS account
- Bước 2: Đi tới AWS IAM console rồi bấm Create User để tạo User mới.
- Bước 3: Nhập tên cho User (e.g. cdk-workshop) và nhớ chọn Programmatic access
![Security Hub](../../../images/1/1.png?width=80pc) 
- Bước 4: Bấm Next: Permissions để chuyển sang bước tiếp theo
- Bước 5: Bấm Attach existing policies directly rồi chọn AdministratorAccess
![Security Hub](../../../images/1/2.png?width=80pc) 
- Bước 6: Bấm Next: Review
- Bước 7: Bấm Create User
- Bước 8: Tại bước này, trên màn hình sẽ hiển thị cho biết thông tin Access key ID và bạn sẽ có lựa chọn để bấm xem Secret access key. Lưu lại 2 thông tin này hoặc giữ nguyên cửa sổ trình duyệt đang mở.
![Security Hub](../../../images/1/3.png?width=80pc) 

Sau khi đã có thông tin User với quyền truy cập và sử dụng toàn bộ dịch vụ AWS, ta thực hiện cấu hình Credential cho phép AWS CLI thao tác với các dịch vụ AWS thông qua giao diện dòng lệnh.  
Để làm được điều này, truy cập cửa sổ terminal, rồi nhập lệnh aws configure. Tiếp tục nhập access key ID và secret key và chọn default region. Tốt nhất nên chọn region nào đó mà chưa được deploy bất cứ ứng dụng nào cả. 
```python
aws configure
AWS Access Key ID [None]: <type key ID here>
AWS Secret Access Key [None]: <type access key>
Default region name [None]: <choose region (e.g. "us-east-1", "eu-west-1")>
Default output format [None]: <leave blank>
```
### 3. Node.js
AWS CDK sử dụng Node.js, phiên bản >= 10.3.0
Để cài đặt Node.js truy cập trang chủ Node.js [tại đây](https://nodejs.org/en/).
Nếu máy tính của bạn đã được cài đặt Node.js, thực hiện kiểm tra version bằng lệnh:

```python
node --version
```
### 4. IDE - Công cụ làm việc với Ngôn ngữ Lập trình
Một trong những điểm nổi bật nhất của AWS CDK đó là bạn có thể tận dụng môi trường làm việc ưa thích của bạn với kinh nghiệm sử dụng giày dạn sẽ rất tốt cho việc khám phá hàng trăm Dịch vụ và Tính năng khác nhau của AWS.
Chúng tôi cực kỳ kỳ khuyến khích các bạn nên sử dụng một IDE mà có khả năng hỗ trợ code-completion (hoàn thiện mã lệnh) và syntax highlighting (đánh dấu từ khóa mã lệnh).  
Bạn có thể cân nhắc sử dụng một trong các IDE bên dưới: 
- [VSCode](https://code.visualstudio.com/)(recommended)]
- [AWS Cloud9](https://aws.amazon.com/cloud9/)
- [Atom](https://atom.io/) with the [atom-typescript](https://atom.io/packages/atom-typescript) plugin
- [vim](https://www.vim.org/) with [tsuquyomi](https://github.com/Quramy/tsuquyomi)
- [WebStorm](https://www.jetbrains.com/help/webstorm/typescript-support.html)
- [Emacs](https://www.gnu.org/software/emacs/) with the [tide](https://github.com/ananthakumaran/tide) mode
- [PyCharm](https://www.jetbrains.com/pycharm/download/)

### 5. AWS CDK Toolkit
AWS CDK Toolkit là bộ công cụ command-line cho phép bạn có thể làm việc với các ứng dụng CDK.
Để cài đặt Toolkit, mở một cửa sổ terminal session với quyền quản trị rồi chạy lệnh:

```python
npm install -g aws-cdk
```
Sau khi cài đặt xong, thực hiện kiểm tra toolkit version bằng lệnh
```python
$ cdk --version
1.21.1 (build 842cc5f)
```

---
title: "Worklog Tuần 6"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---


### Mục tiêu tuần 6:

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu Dịch vụ bảo mật trên AWS  <br>&emsp; + Share Responsibility Model <br>&emsp; + Amazon Identity and access management <br>&emsp; + Amazon Cognito <br>&emsp; + AWS Organization <br>&emsp; + AWS Identity Center <br>&emsp; + Amazon Key Management Service <br>&emsp; + AWS Security Hub <br>&emsp; + Hands-on and Additional research | 13/10/2025   | 13/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Tìm hiểu Security Standards <br> - **Thực hành:** <br>&emsp; + Kích hoạt Security Hub <br>&emsp; + Điểm từng bộ tiêu chuẩn <br>&emsp; + Dọn dẹp tài nguyên <br> - **Thực hành:** Tối ưu hóa chi phí EC2 với Lambda <br>&emsp; + Các bước chuẩn bị: <br>&emsp; 1. Tạo VPC <br>&emsp; 2. Tạo Security group <br>&emsp; 3. Tạo EC2 <br>&emsp; 4. Cài đặt Web-Hooks đến Slack <br>&emsp; + Tạo Tag cho Instance <br>&emsp; + Tạo Role cho Lambda <br>&emsp; + Tạo Lambda Function <br>&emsp; 1. Tạo Lambda Function thực hiện chức năng Stop instances <br>&emsp; 2. Tạo Lambda Function thực hiện chức năng Start instances <br>&emsp; + Kiểm tra kết quả <br>&emsp; + Dọn dẹp tài nguyên| 14/10/2025   | 14/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Tìm hiểu AWS Resource Groups <br> - **Thực hành:** <br>&emsp; + Tạo EC2 Instance có tag <br>&emsp; + Quản lý Tags trong AWS Resources <br>&emsp; + Lọc tài nguyên theo tag <br>&emsp; + Sử dụng tag bằng CLI <br>&emsp; + Tạo một Resource Group <br>&emsp; + Dọn dẹp tài nguyên| 15/10/2025   | 15/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - **Thực hành:** <br>&emsp; + Tạo IAM User <br>&emsp; + Tạo IAM Policy <br>&emsp; + Tạo IAM Role <br>&emsp; + Chuyển Role <br>&emsp; + Tiến hành truy cập EC2 console ở AWS Region - Tokyo <br>&emsp; + Tiến hành truy cập EC2 console ở AWS Region - North Virginia <br>&emsp; + Tiến hành tạo EC2 instance khi không có và có Tags thỏa điều kiện <br>&emsp; + Chỉnh sửa Resource Tag trên EC2 Instance <br>&emsp; + Kiểm tra chính sách <br>&emsp; + Don dẹp tài nguyên | 16/10/2025   | 16/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - Tìm hiểu IAM Permission Boundary là gì <br> - **Thực hành:** Giới hạn quyền của user với IAM Permission Bourdary <br>&emsp; + Tạo Policy Giới hạn <br>&emsp; + Tạo IAM User Giới Hạn <br>&emsp; + Kiểm tra IAM User Giới <br>&emsp; + Hạn Dọn dẹp tài nguyên| 17/10/2025   | 17/10/2025      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 6:
* Hiểu về các dịch vụ bảo mật trên AWS
  * Share Responsibility Model
  * Amazon Identity and access management
  * Amazon Cognito
  * AWS Organization
  * ...

* Nắm khái niệm Authentication/Authorization qua Cognito.
* Nắm kiến thức của IAM, Security Hub, Organizations.

* Sử dụng tagging và resource groups để quản lý tài nguyên.

* Tối ưu chi phí bằng automation stop/start EC2.
* Biết cách sử dụng IAM Permission Boundary để giới hạn quyền của User



---
title: "Blog 1"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# AWS Lambda giới thiệu mức giá theo tầng cho nhật ký Amazon CloudWatch và các đích ghi nhật ký bổ sung

*bởi Shridhar Pandey và Matthew Barker | 01 tháng 5 năm 2025 | [[Thông báo](https://aws.amazon.com/blogs/compute/category/post-types/announcements/)], [[AWS Lambda](https://aws.amazon.com/blogs/compute/category/compute/aws-lambda/)], [[Intermediate (200)](https://aws.amazon.com/blogs/compute/category/learning-levels/intermediate-200/)], [[Serverless](https://aws.amazon.com/blogs/compute/category/serverless/)]*

Việc ghi nhật ký hiệu quả là một phần quan trọng của chiến lược quan sát khi xây dựng các ứng dụng không có máy chủ bằng **AWS Lambda**.

Lambda tự động thu thập và gửi nhật ký đến **Amazon CloudWatch Logs**.Điều này cho phép bạn tập trung vào việc xây dựng logic ứng dụng thay vì thiết lập cơ sở hạ tầng ghi nhật ký và cho phép người vận hành khắc phục sự cố lỗi và hiệu suất dễ dàng hơn.

Vào ngày 1 tháng 5 năm 2025, AWS đã công bố những thay đổi đối với tính năng ghi nhật ký Lambda, giúp giảm chi phí ghi nhật ký Lambda CloudWatch và giúp việc sử dụng nhiều công cụ giám sát dễ dàng và tiết kiệm chi phí hơn. Nhật ký Lambda hiện có sẵn với mức giá theo tầng dựa trên khối lượng khi sử dụng các lớp nhật ký CloudWatch Logs Standard và Infrequent Access. Khi tạo nhật ký Lambda ở quy mô lớn, bạn có thể mong đợi chi phí giảm ngay lập tức theo mô hình định giá mới này. Ngoài Nhật ký CloudWatch, Lambda hiện cũng hỗ trợ **Amazon S3** và **Amazon Data Firehose** làm đích bổ sung cho nhật ký Lambda. Nhật ký Lambda được gửi đến S3 và Firehose cũng có sẵn với mức giá theo tầng dựa trên khối lượng.

Bài đăng trên blog này đề cập đến một số cải tiến ghi nhật ký Lambda gần đây và mô tả cách thay đổi này mang lại trải nghiệm ghi nhật ký đơn giản hơn, tiết kiệm chi phí hơn cho Lambda.

## Tổng quan

Việc ghi nhật ký cung cấp cho các nhà phát triển và vận hành viên dữ liệu giá trị để gỡ lỗi và xử lý sự cố hành vi ứng dụng, các vấn đề về hiệu suất và các lỗi tiềm ẩn. Điều này càng trở nên quan trọng hơn đối với các ứng dụng không máy chủ được xây dựng bằng Lambda do tính chất tạm thời và không trạng thái của **Lambda execution environment**. Tích hợp sẵn của Lambda với CloudWatch Logs đảm bảo rằng nhật ký cho mọi lệnh gọi hàm đều có sẵn để phân tích. Dữ liệu nhật ký được thu thập bao gồm nhật ký ứng dụng do mã hàm Lambda của bạn tạo ra và nhật ký hệ thống do dịch vụ Lambda tạo ra khi chạy mã hàm. CloudWatch Logs cho phép bạn tìm kiếm, lọc và phân tích dữ liệu nhật ký để khắc phục sự cố, theo dõi số liệu và thiết lập cảnh báo.

Yêu cầu ghi nhật ký ngày càng phát triển khi các ứng dụng không máy chủ ngày càng phức tạp và mở rộng quy mô, đôi khi bao gồm hàng trăm hoặc hàng nghìn hàm Lambda, tạo ra khối lượng nhật ký đáng kể. Các tổ chức cần các giải pháp ghi nhật ký tinh vi có thể xử lý quy mô này mà vẫn tiết kiệm chi phí. Một số tình huống, chẳng hạn như giám sát các giao dịch kinh doanh quan trọng, đòi hỏi phân tích nhật ký theo thời gian thực, trong khi những tình huống khác tập trung vào phân tích pháp y sau sự kiện. Nhật ký gỡ lỗi từ môi trường phát triển và môi trường dàn dựng thường cần độ chi tiết cao, trong khi bạn có thể muốn độ chi tiết thấp hơn trong nhật ký sản xuất để cải thiện tỷ lệ tín hiệu trên nhiễu.


### Các cải tiến gần đây của Lambda logging

Trong những năm gần đây, Lambda và CloudWatch Logs đã mở rộng khả năng ghi nhật ký của Lambda để đáp ứng nhu cầu ngày càng tăng của các ứng dụng không máy chủ. Các khả năng này cung cấp thông tin chi tiết sâu hơn, khả năng kiểm soát tốt hơn và các giải pháp tiết kiệm chi phí hơn để thu thập, xử lý và sử dụng nhật ký, từ đó nâng cao trải nghiệm quan sát không máy chủ. **Các kiểm soát ghi nhật ký nâng cao của Lambda** (Lambda advanced logging controls)cho phép các nhà phát triển kiểm soát việc tạo và nội dung nhật ký. Các điều khiển này cho phép bạn thu thập nhật ký Lambda ở định dạng có cấu trúc JSON. Bạn không cần phải sử dụng thư viện ghi nhật ký và tùy chỉnh các mức nhật ký (INFO, DEBUG, WARN, ERROR) riêng biệt cho nhật ký ứng dụng và nhật ký hệ thống. Điều này giúp giảm chi phí ghi nhật ký bằng cách đảm bảo chỉ tạo các nhật ký cần thiết trong khi vẫn duy trì khả năng hiển thị phù hợp trên các môi trường khác nhau. Ví dụ: bạn có thể thiết lập mức ghi nhật ký DEBUG chi tiết trong môi trường phát triển đồng thời giới hạn mức ghi nhật ký sản xuất ở mức ERROR để cải thiện tỷ lệ tín hiệu trên nhiễu và kiểm soát chi phí.

**Infrequent Access log class** dành cho CloudWatch Logs đã giới thiệu một giải pháp tiết kiệm chi phí cho các nhật ký cần lưu giữ nhưng ít được truy cập. Lớp Infrequent Access có giá nhập dữ liệu (ingestion) trên mỗi GB thấp hơn 50% so với lớp nhật ký Standard. Bộ tính năng được thiết kế riêng này cho phép bạn giảm chi phí ghi nhật ký trong khi vẫn duy trì quyền truy cập vào dữ liệu lịch sử cho mục đích tuân thủ, kiểm toán hoặc phân tích pháp y.

**CloudWatch Logs Live Tail** là một tính năng phân tích và truyền phát nhật ký tương tác theo thời gian thực. Live Tail đơn giản hóa quy trình gỡ lỗi và giám sát; cho phép bạn quan sát đầu ra nhật ký khi các hàm thực thi mà không cần rời khỏi bảng điều khiển Lambda. Điều này giúp dễ dàng xác định và chẩn đoán sự cố trong quá trình phát triển và xử lý sự cố. Logs Live Tail cũng có sẵn trong **Visual Code IDE**.


## Mức giá theo bậc cho Lambda logging trong CloudWatch Logs

Bắt đầu từ hôm nay, nhật ký Lambda được gửi đến CloudWatch Logs sẽ được phân loại là Vended Logs (Nhật ký do dịch vụ cung cấp), tức là nhật ký từ các dịch vụ AWS cụ thể có sẵn theo mức giá theo khối lượng. Điều này thay thế mô hình giá cố định trước đây khi sử dụng lớp nhật ký Chuẩn của CloudWatch Logs. Ví dụ: tại Khu vực AWS Đông Hoa Kỳ (Bắc Virginia), bạn bị tính phí 0,50 đô la cho mỗi GB khi sử dụng lớp nhật ký Chuẩn cho nhật ký Lambda của mình. Theo mô hình giá mới, bạn bị tính phí khi gửi nhật ký Lambda đến CloudWatch Logs bắt đầu từ 0,50 đô la cho mỗi GB cho lần sử dụng đầu tiên. Khi khối lượng nhật ký tăng lên, giá cho mỗi GB sẽ tự động giảm qua nhiều tầng, xuống mức thấp nhất là 0,05 đô la cho mỗi GB ở tầng thấp nhất. Thay đổi giá này được tự động áp dụng cho tất cả nhật ký Lambda được gửi đến CloudWatch Logs, không yêu cầu bạn phải thay đổi mã hoặc cấu hình.


| Dữ liệu đã được nhập | Nhật ký CloudWatch Tiêu chuẩn | CloudWatch Logs Infrequent Access |
| :------------------- | :--------------------------- | :------------------------------- |
| **10 TB đầu tiên/tháng** | $0.50/GB | $0.25/GB |
| **20 TB tiếp theo/tháng** | $0.25/GB | $0.15/GB |
| **20 TB tiếp theo/tháng** | $0.10/GB | $0.075/GB |
| **hơn 50 TB/tháng** | $0.05/GB | $0.05/GB |

*Bảng 1: Giá theo bậc cho nhật ký Lambda trong CloudWatch Logs tại Khu vực Đông Hoa Kỳ (Bắc Virginia) *

Khi tạo nhật ký Lambda ở quy mô lớn, bạn sẽ thấy chi phí giảm ngay lập tức theo mô hình giá mới này. Ví dụ: nếu bạn tạo 60 TB nhật ký Lambda hàng tháng trong CloudWatch Logs, chi phí sẽ giảm 58% (từ 30.000 đô la xuống còn 12.500 đô la). Các bậc giá sẽ tăng theo khối lượng ghi nhật ký của bạn, đảm bảo lợi ích về chi phí sẽ tăng lên khi ứng dụng của bạn phát triển. Điều này cho phép bạn duy trì các hoạt động ghi nhật ký toàn diện mà trước đây có thể rất tốn kém. Định giá theo bậc của Vended logs được áp dụng cho tất cả Vended logs được nạp vào CloudWatch chứ không phải theo bậc cho mỗi dịch vụ.
Khi thu thập các nhật ký bán sẵn khác, chẳng hạn như **Amazon Virtual Private Cloud flow logs** và  **Amazon Route 53 resolver query logs**, bạn sẽ thấy mức chiết khấu lớn hơn khi phân tầng được áp dụng ở khối lượng thu thập nhật ký hợp nhất.

## Điểm đến ghi nhật ký Lambda mới: Amazon S3 và Amazon Data Firehose

Bắt đầu từ hôm nay, ngoài CloudWatch Logs, Lambda cũng hỗ trợ Amazon S3 và Amazon Data Firehose làm đích cho nhật ký Lambda. Khi sử dụng S3 hoặc Firehose làm đích, chi phí ghi nhật ký bắt đầu từ 0,25 đô la Mỹ/GB. Mức giá theo bậc cũng được áp dụng, với mức giá giảm xuống còn 0,05 đô la Mỹ/GB ở bậc thấp nhất. Mức giá theo bậc này cũng được áp dụng cho khối lượng nhật ký được hợp nhất.

| Dữ liệu đã được nhập | Nhật ký CloudWatch Tiêu chuẩn | CloudWatch ghi lại các lần truy cập không thường xuyên |
| :------------------- | :--------------------------- | :---------------------------------------------------- |
| **10 TB đầu tiên/tháng** | $0.50/GB | $0.25/GB |
| **20 TB tiếp theo/tháng** | $0.25/GB | $0.15/GB |
| **20 TB tiếp theo/tháng** | $0.10/GB | $0.075/GB |
| **hơn 50 TB/tháng** | $0.05/GB | $0.05/GB |

*Bảng 2: Giá theo từng bậc cho dịch vụ giao nhật ký Lambda tới Amazon S3 và Amazon Data Firehose tại Khu vực Đông Hoa Kỳ (Bắc Virginia) *

Việc phân phối trực tiếp nhật ký Lambda đến S3 mang lại sự linh hoạt hơn trong việc quản lý nhật ký. Hỗ trợ Firehose giúp đơn giản hóa việc phân phối nhật ký Lambda đến các đích bổ sung như **Amazon OpenSearch Service**, HTTP endpoints, và **các nhà cung cấp khả năng quan sát (observability providers) bên thứ ba**. Điều này phù hợp với mô hình phân phối nhật ký đã được thiết lập, được sử dụng với các dịch vụ điện toán AWS khác như **Amazon Elastic Container Service (Amazon ECS)** and **Amazon Elastic Compute Cloud (Amazon EC2)**.

Khả năng mới này mang lại lợi ích đáng kể về chi phí và hợp lý hóa việc phân phối nhật ký đến các đích ghi nhật ký bổ sung, giúp sử dụng dễ dàng hơn nhiều công cụ giám sát (bao gồm cả CloudWatch) khi xây dựng các ứng dụng không có máy chủ bằng Lambda.


### Thiết lập đích ghi nhật ký Lambda mới

[cite_start]Tất cả các hàm Lambda mới và hiện có đều có CloudWatch Logs làm đích ghi nhật ký mặc định, với S3 và Firehose là các lựa chọn thay thế[cite: 1]. [cite_start]Khi bạn chọn S3 hoặc Firehose, Lambda sẽ gửi nhật ký thông qua một **lớp nhật ký phân phối (delivery log class)** mới của CloudWatch Logs[cite: 1].

Các bước thiết lập cơ bản trong bảng điều khiển Lambda:

Tất cả các hàm Lambda mới và hiện có đều có CloudWatch Logs làm đích ghi nhật ký mặc định, với S3 và Firehose là các lựa chọn thay thế. Khi bạn chọn S3 hoặc Firehose làm đích ghi nhật ký,Lambda sẽ gửi nhật ký đến đích đã chọn thông qua (via) một lớp nhật ký phân phối (delivery log class) mới của CloudWatch Logs. Lớp nhật ký này cho phép định tuyến hiệu quả nhưng không hỗ trợ các tính năng lớp nhật ký CloudWatch Logs Standard, chẳng hạn như Logs Insights và Live Tail.

Để thiết lập S3 hoặc Firehose làm đích cho nhật ký Lambda của bạn trong bảng điều khiển Lambda:

1. Điều hướng đến Lambda console,và chọn hoặc tạo một hàm để thiết lập đích ghi nhật ký S3 hoặc Firehose.
2. Trong tab Configuration, chọn Monitoring and operations tools ở ngăn bên trái.
3. Chọn Edit trong Logging configuration. Thao tác này sẽ mở trang Edit logging configuration
4. Trong phần "Log destination", chọn Amazon S3 hoặc Amazon Data Firehose. Amazon CloudWatch Logs là lựa chọn mặc định.
5. Trong CloudWatch delivery log group, chọn Create new log group hoặc Existing log group.
6. Để tạo nhóm nhật ký phân phối mới để gửi nhật ký đến S3, hãy nhập tên nhóm nhật ký và chỉ định vùng lưu trữ S3 đích. Cung cấp vai trò AWS Identity and Access Management (IAM)cho CloudWatch Logs để phân phối nhật ký đến S3. 
7. Để sử dụng nhóm nhật ký phân phối hiện có, hãy chọn một nhóm từ nhóm Delivery log group. Delivery log group được chọn phải có đích được cấu hình (S3 hoặc Firehose) và khớp với đích bạn đã chọn.

Các điều khiển ghi nhật ký nâng cao cũng khả dụng cho các đích S3 và Firehose. Các điều khiển này bao gồm lựa chọn định dạng có cấu trúc JSON và bộ lọc cấp độ nhật ký cho cả nhật ký ứng dụng và nhật ký hệ thống. Điều này cung cấp cho bạn các điều khiển quản lý nhật ký nâng cao để tìm kiếm, lọc và phân tích dễ dàng hơn. Bạn cũng có thể sử dụng AWS Command Line Interface (AWS CLI) và infrastructure as code (IaC) tools như AWS CloudFormation and AWS Cloud Development Kit (AWS CDK) để thiết lập việc phân phối nhật ký Lambda đến S3 và Firehose.


## Thực hành tốt nhất

Để tận dụng tối đa những thay đổi được công bố hôm nay, hãy đảm bảo chiến lược ghi nhật ký của bạn phù hợp chặt chẽ với yêu cầu khối lượng công việc. Ví dụ: hãy cân nhắc gửi các nhật ký sản xuất quan trọng đến CloudWatch Logs để tận dụng các tính năng phân tích và cảnh báo theo thời gian thực tiên tiến. Giờ đây, bạn sẽ tự động được hưởng chiết khấu theo khối lượng thông qua giá theo bậc trong CloudWatch Logs cho các tình huống ghi nhật ký khối lượng lớn. Đối với các nhật ký cần lưu giữ lâu dài để phân tích lịch sử, bạn có thể sử dụng các lớp lưu trữ của S3 để giảm chi phí hơn nữa. Khi sử dụng các công cụ giám sát hiện có hoặc của bên thứ ba, tích hợp trực tiếp thông qua Firehose giúp loại bỏ nhu cầu về các giải pháp chuyển tiếp tùy chỉnh và các chi phí liên quan.


Tối ưu hóa chi phí ghi nhật ký không chỉ giới hạn ở việc lựa chọn đích đến. Thường xuyên theo dõi khối lượng nhật ký để hiểu tác động của các bậc giá. Triển khai các chính sách lưu giữ phù hợp để ngăn chặn việc lưu trữ nhật ký cũ không cần thiết và việc lấy mẫu nhật ký cho các nhật ký gỡ lỗi khối lượng lớn. Cân nhắc sử dụng các chiến lược ghi nhật ký khác nhau trên các môi trường phát triển, dàn dựng và sản xuất để cân bằng nhu cầu quan sát với hiệu quả chi phí.


## Kết luận

Mức giá theo từng bậc cho nhật ký Lambda trong CloudWatch Logs và hỗ trợ S3 và Firehose khi có thêm các đích ghi nhật ký, giúp cải thiện khả năng quan sát ứng dụng Lambda. Giờ đây, bạn có thể quản lý chi phí ghi nhật ký ở quy mô lớn và mở rộng các giải pháp giám sát Lambda thông qua các tích hợp tiết kiệm chi phí và dễ cấu hình. Cho dù bạn đang xây dựng các ứng dụng không máy chủ mới hay tối ưu hóa các ứng dụng hiện có, những cải tiến này giúp bạn triển khai các chiến lược ghi nhật ký toàn diện, có khả năng mở rộng hiệu quả về chi phí theo khối lượng công việc của bạn.

Các tính năng mới được công bố hôm nay có sẵn tại tất cả các AWS Regions Hỗ trợ cấu hình phân phối nhật ký đến S3 và Firehose trong bảng điều khiển Lambda hiện có sẵn tại các Khu vực Đông Hoa Kỳ (Ohio), Đông Hoa Kỳ (Bắc Virginia), Tây Hoa Kỳ (Oregon) và Châu Âu (Ireland), cùng các Khu vực bổ sung sẽ sớm được ra mắt. Xem lại Lambda documentation và CloudWatch Logs documentation để tìm hiểu thêm về các tính năng này và cách sử dụng chúng. Xem lại CloudWatch pricing page để tìm hiểu thêm về cách tính giá của các tính năng này.
Để biết thêm tài nguyên học tập về không máy chủ, hãy truy cập Serverless Land.

---
title: "Blog 3"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Các chỉ số AWS KMS trong CloudWatch giúp bạn theo dõi và hiểu rõ hơn về mức độ sử dụng khóa KMS

Bởi Norman Li and Haiyu Zhen | ngày 17 tháng 3 năm 2025 | trong [AWS Key Management Service](https://aws.amazon.com/blogs/security/category/security-identity-compliance/aws-key-management-service/), [Best Practices](https://aws.amazon.com/blogs/security/category/post-types/best-practices/), [Intermediate (200)](https://aws.amazon.com/blogs/security/category/learning-levels/intermediate-200/), [Security, Identity, & Compliance](https://aws.amazon.com/blogs/security/category/security-identity-compliance/), [Technical How-to](https://aws.amazon.com/blogs/security/category/post-types/technical-how-to/) |

[AWS Key Management Service (AWS KMS)](https://aws.amazon.com/kms/) hân hạnh ra mắt tính năng lọc cấp độ khóa cho việc sử dụng API AWS KMS trong số liệu của [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), cung cấp khả năng hiển thị nâng cao để giúp khách hàng cải thiện hiệu quả hoạt động và hỗ trợ quản lý rủi ro bảo mật và tuân thủ.

AWS KMS hiện đang công bố số liệu sử dụng API AWS KMS ở cấp độ tài khoản [với Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Service-Quota-Integration.html), cho phép bạn theo dõi và quản lý việc sử dụng API. Tuy nhiên, nếu bạn đang sử dụng nhiều khóa KMS, việc xác định chính xác khóa nào đang sử dụng nhiều nhất hạn ngạch về tần suất yêu cầu (request rate quota) hoặc gây ra chi phí API đáng kể sẽ trở nên khó khăn. Ví dụ: nếu bạn có hơn 10 khóa KMS đang hoạt động trong tài khoản, trước khi tính năng này ra mắt, bạn sẽ cần xây dựng một giải pháp tùy chỉnh dựa trên CloudTrail và Amazon Athena để xác định khóa nào đang chiếm phần lớn mức sử dụng và chi phí API. Với các chỉ số CloudWatch mới có trong không gian tên (namespace) AWS/KMS trong CloudWatch, bạn có thể theo dõi, tìm hiểu và đặt cảnh báo về mức sử dụng API chi tiết ở cấp độ khóa KMS riêng lẻ mà không cần xây dựng giải pháp tùy chỉnh tốn kém.

Bài viết trên blog này khám phá một số trường hợp sử dụng giúp bạn tận dụng tốt hơn các số liệu CloudWatch mới được giới thiệu này để quản lý việc sử dụng và chi phí API AWS KMS. Các trường hợp sử dụng bao gồm việc xem và hiểu rõ việc sử dụng API của bạn ở cấp độ chính, cũng như tạo cảnh báo CloudWatch để phát hiện việc sử dụng vượt mức ngoài ý muốn.

## Tổng quan về số liệu CloudWatch mới cho khóa KMS

Với số liệu CloudWatch cho khóa KMS, giờ đây bạn có thể thực hiện các thao tác sau:

1.  Xem mức sử dụng API cho một khóa KMS cụ thể, được lọc theo từng thao tác API (ví dụ:Encrypt, Decrypt, or GenerateDataKey).
2.  Xem mức sử dụng tổng hợp trên các thao tác mã hóa cho một khóa KMS nhất định.
3.  Thiết lập cảnh báo nếu một khóa KMS cụ thể vượt quá ngưỡng quy định trên một thao tác API đơn lẻ hoặc một tập hợp các thao tác API.

Phương pháp tiếp cận hợp lý này cho phép bạn nhanh chóng theo dõi, hiểu và khắc phục sự cố các mẫu sử dụng API của khóa KMS mà không cần phải thực hiện quy trình nhiều bước như trước đây. Hãy cùng tìm hiểu chi tiết cách sử dụng các số liệu sử dụng API cấp độ khóa này trong hai ví dụ thực tế.

### Ví dụ 1: Cách xác định các khóa KMS tiêu thụ nhiều hạn ngạch sử dụng API nhất hoặc đóng góp nhiều phí API nhất

Khi vượt quá hạn mức yêu cầu API KMS của AWS, bạn có thể xem mức sử dụng API KMS của AWS trong [Service Quotas console.](https://console.aws.amazon.com/servicequotas) Tuy nhiên, bạn vẫn có thể gặp khó khăn khi xác định các khóa KMS chiếm nhiều hạn mức yêu cầu nhất. Khi nhận được phí API KMS của AWS vượt quá dự kiến, bạn có thể kiểm tra mức sử dụng chi tiết trong từng Khu vực AWS trong Cost Explorer, nhưng bạn không thể dễ dàng tìm thấy các khóa KMS có mức phí API cao nhất. Quá trình này càng trở nên khó khăn hơn khi bạn quản lý một số lượng lớn khóa KMS.

Với các chỉ số CloudWatch về mức sử dụng API cấp độ khóa, bạn có thể sử dụng tùy chọn truy vấn chỉ số (metric query) nâng cao để truy vấn dữ liệu CloudWatch Metrics Insights bằng phương ngữ SQL thân thiện với người dùng để xác định các khóa KMS chiếm phần lớn hạn ngạch sử dụng API hoặc đóng góp nhiều phí API nhất.

### Walkthrough

Để sử dụng Amazon CloudWatch Metrics Insights nhằm xác định 20 khóa KMS hàng đầu có mức sử dụng API mã hóa nhiều nhất trong 3 giờ qua, hãy hoàn thành các bước sau:

1.  Mở [CloudWatch console](https://console.aws.amazon.com/cloudwatch/).
2.  Trong ngăn điều hướng, chọn **Metrics** (Số liệu), rồi chọn **All metrics** (Tất cả số liệu).
3.  Chọn tab **Multi source query** (Truy vấn đa nguồn).
4.  Đối với nguồn dữ liệu, chọn **CloudWatch Metrics Insights** (Thông tin chi tiết về số liệu CloudWatch).
5.  Bạn có thể nhập truy vấn ví dụ sau trong chế độ xem Editor:

    Lưu ý: Trong chế độ xem Builder, các tùy chọn không gian tên số liệu, tên số liệu, lọc theo, nhóm theo, sắp xếp theo và giới hạn sẽ được hiển thị. Trong chế độ xem Editor, các tùy chọn tương tự như trong chế độ xem Builder được hiển thị ở định dạng truy vấn.

    | |
    | :---: |
    | |
    | SELECT SUM(SuccessfulRequest) |
    | FROM SCHEMA(\"AWS/KMS\", KeyArn, Operation) |
    | GROUP BY KeyArn |
    | ORDER BY MAX () DESC |
    | LIMIT 20 |
    | |

6.  Chọn **Run** (Chạy) trong chế độ xem Editor hoặc **Graph query** (Truy vấn đồ thị) trong chế độ xem Builder.

## Ví dụ 2: Cách thiết lập cảnh báo chi tiết mới về việc sử dụng API AWS KMS ngoài ý muốn

Việc chạy các quy trình xử lý dữ liệu lớn đọc các tệp [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) được mã hóa bằng khóa KMS là một tình huống phổ biến đối với các dự án phân tích, báo cáo kinh doanh hoặc học máy. Thông thường, các quy trình này chỉ đọc một số lượng tệp giới hạn từ S3 trong mỗi lần gọi. Tuy nhiên, các quy trình được cấu hình sai có thể vô tình đọc một số lượng lớn tệp S3, dẫn đến việc vượt quá hạn ngạch tần suất yêu cầu (request rate quotas) API AWS KMS của bạn hoặc phát sinh các khoản phí không mong muốn do mức sử dụng API AWS KMS tăng vọt (spiky). Trước đây, để giải quyết vấn đề này, bạn sẽ phải xây dựng một hệ thống cảnh báo tùy chỉnh bằng cách làm theo các bước sau: 1) gửi các sự kiện AWS CloudTrail do AWS KMS tạo ra đến Amazon CloudWatch Logs; 2) viết các truy vấn trong Amazon CloudWatch Logs Insights để theo dõi mức sử dụng yêu cầu API của bạn; và 3) bật phát hiện bất thường trên biểu thức toán học CloudWatch Log Insights tương ứng.

Giờ đây, với số liệu CloudWatch về mức sử dụng API ở cấp độ khóa, bạn có thể trực tiếp bật phát hiện bất thường trên các số liệu này để thiết lập cảnh báo cho các mẫu sử dụng API AWS KMS bất thường. Điều này cung cấp một phương pháp hợp lý và hiệu quả hơn để giám sát và phát hiện các quy trình làm việc tiềm ẩn rủi ro. Bằng cách sử dụng các số liệu CloudWatch và khả năng phát hiện bất thường này, bạn có thể chủ động xác định và xử lý các sự gia tăng ngoài ý muốn trong việc sử dụng API AWS KMS, giúp tránh các khoản phí bất ngờ hoặc gián đoạn dịch vụ trong quy trình phân tích, báo cáo hoặc học máy của bạn.

### Walkthrough

Hãy xem xét một tình huống trong đó bạn có một quy trình phân tích chạy thường xuyên, sử dụng **Decrypt** AWS KMS API trên khóa KMS để giải mã và đọc dữ liệu từ S3. Bạn muốn bật tính năng phát hiện bất thường trên khóa KMS để kích hoạt báo động khi **Decrypt** gọi đến khóa KMS cụ thể phát hiện một xu hướng hoặc mẫu hình rõ ràng. Để thực hiện việc này, hãy hoàn thành các bước sau:

1.  Mở [CloudWatch console](https://console.aws.amazon.com/cloudwatch/).
2.  Trong ngăn điều hướng, chọn **Metrics** (Số liệu), rồi chọn **All metrics** (Tất cả số liệu).
3.  Chọn **KMS**, rồi chọn **KeyArn**, **Operation**.
4.  Trong thanh tìm kiếm, nhập Tên tài nguyên Amazon (**ARN**) của khóa, rồi chọn **Search** (Tìm kiếm). Chọn số liệu CloudWatch mà bạn muốn bật phát hiện bất thường.
5.  Điều hướng đến **Graphed metrics** (Số liệu biểu đồ), và sử dụng danh sách thả xuống **Statistic** (Thống kê) và **Period** (Kỳ hạn), chọn số liệu thống kê và period (Kỳ hạn) mà bạn muốn theo dõi. Sau đó, bạn có thể bật phát hiện bất thường bằng cách chọn biểu tượng Pulse.
6.  Bạn có thể [điều chỉnh phát hiện bất thường](https://aws.amazon.com/blogs/mt/operationalizing-cloudwatch-anomaly-detection/) bằng cách đặt độ nhạy để điều chỉnh băng thông, nếu cần.

## Conclusion

Bài đăng trên blog này đã nêu bật khả năng lọc cấp khóa mới được giới thiệu cho việc sử dụng API AWS KMS trong CloudWatch. Chúng tôi đã trình bày hai trường hợp sử dụng thực tế để minh họa cách bạn có thể sử dụng các số liệu CloudWatch mới. Các trường hợp sử dụng này bao gồm cải thiện khả năng hiển thị hoạt động, thiết lập cảnh báo chủ động về các bất thường trong mô hình sử dụng API KMS và có khả năng theo dõi việc sử dụng khóa chi tiết cho mục đích tuân thủ.

Nếu bạn có phản hồi về bài viết này, vui lòng gửi ý kiến trong phần Bình luận bên dưới. Nếu bạn có thắc mắc về bài viết này, vui lòng tạo một chủ đề mới trong [AWS Key Management Service re:Post](https://forums.aws.amazon.com/forum.jspa?forumID=182).

-----

| |
| :---: |
| **Norman Li**Norman là Giám đốc Phát triển Phần mềm cho AWS KMS. Trong vai trò này, Norman lãnh đạo việc phát triển các tính năng hiển thị (visibility features), cũng như các sáng kiến mở rộng quy mô nội bộ. Ngoài giờ làm việc, Norman thích dành thời gian ở những ngọn núi tuyệt đẹp vùng Tây Bắc Thái Bình Dương. |

| |
| :---: |
| **Haiyu Zhen**Haiyu là Kỹ sư Phát triển Phần mềm Cao cấp cho AWS KMS. Cô chuyên xây dựng các hệ thống phân tán quy mô lớn, bảo mật và đam mê tăng cường bảo mật ứng dụng cloud-native mà không làm ảnh hưởng đến hiệu suất. |

TAGS: [AWS Key Management Service](https://aws.amazon.com/blogs/security/tag/aws-key-management-service/), [AWS Key Management Service (KMS)](https://aws.amazon.com/blogs/security/tag/aws-key-management-service-kms/), [Security Blog](https://aws.amazon.com/blogs/security/tag/security-blog/)
---
title: "Các bài blogs đã dịch"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


Tại đây sẽ là phần liệt kê, giới thiệu các blogs mà các bạn đã dịch. Ví dụ:

###  [Blog 1 - AWS Lambda giới thiệu mức giá theo tầng cho nhật ký Amazon CloudWatch và các đích ghi nhật ký bổ sung](3.1-Blog1/)
Blog này giới thiệu các cập nhật mới của AWS Lambda liên quan đến ghi nhật ký (logging), giúp các ứng dụng serverless tối ưu chi phí và linh hoạt hơn khi quản lý log. Bài viết giải thích việc Lambda giờ đây áp dụng định giá theo bậc (tiered pricing) cho CloudWatch Logs, giúp chi phí ghi nhật ký giảm mạnh khi khối lượng log tăng cao. Bên cạnh CloudWatch Logs, Lambda cũng hỗ trợ Amazon S3 và Amazon Data Firehose làm đích ghi nhật ký mới, tạo sự linh hoạt trong lưu trữ, phân tích và tích hợp với các nền tảng quan sát khác. Blog cũng đề cập đến các cải tiến như advanced logging controls, CloudWatch Logs Infrequent Access, và Live Tail để tăng khả năng quan sát. Cuối cùng, bài viết hướng dẫn cách cấu hình log destination và đưa ra các thực hành tối ưu chi phí ghi nhật ký trong môi trường serverless.
###  [Blog 2 - Hướng dẫn triển khai AWS DMS: Xây dựng khả năng di chuyển cơ sở dữ liệu linh hoạt thông qua thử nghiệm, giám sát và SOP](3.2-Blog2/)
Blog này là một hướng dẫn triển khai AWS Database Migration Service (AWS DMS), tập trung vào cách xây dựng một quy trình di chuyển cơ sở dữ liệu linh hoạt, an toàn và có thể lặp lại. Nội dung nhấn mạnh vào việc thử nghiệm trước khi migration, giám sát trong quá trình thực thi, và xây dựng SOP (Standard Operating Procedures) để đảm bảo việc di chuyển dữ liệu diễn ra trơn tru, giảm rủi ro và dễ bảo trì trong môi trường AWS.
###  [Blog 3 - Các chỉ số AWS KMS trong CloudWatch giúp bạn theo dõi và hiểu rõ hơn về mức độ sử dụng khóa KMS](3.3-Blog3/)
Blog này nói về tính năng mới của AWS KMS cho phép theo dõi mức độ sử dụng API ở cấp độ từng khóa thông qua Amazon CloudWatch, giúp người dùng dễ dàng nhận biết khóa nào đang được sử dụng nhiều nhất, khóa nào gây tốn chi phí hoặc có nguy cơ vượt hạn ngạch. Trước đây, việc này khá phức tạp và cần xây dựng giải pháp tùy chỉnh bằng CloudTrail và Athena, nhưng giờ đây CloudWatch đã cung cấp số liệu chi tiết cho từng khóa, giúp việc giám sát trở nên đơn giản và trực quan hơn. Bài viết cũng minh họa cách sử dụng các chỉ số mới này để truy vấn và tìm ra những khóa KMS tiêu thụ API nhiều nhất, cũng như thiết lập cảnh báo tự động dựa trên phát hiện bất thường để kịp thời xử lý khi khối lượng request tăng đột biến do lỗi cấu hình hoặc quy trình hoạt động. Nhờ đó, người dùng có thể nâng cao hiệu quả vận hành, giảm rủi ro bảo mật và tối ưu chi phí khi sử dụng AWS KMS.

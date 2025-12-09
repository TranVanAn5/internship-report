---
title: "Blog 2"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---


# Hướng dẫn triển khai AWS DMS: Xây dựng khả năng di chuyển cơ sở dữ liệu linh hoạt thông qua thử nghiệm, giám sát và SOP

Bởi Sushant Deshmukh, Alex Anto Kizhakeyyepunnil Joy, and Sanyam Jain | Ngày 5 tháng 5 năm 2025 | trong [Nâng cao (300)](https://aws.amazon.com/blogs/database/category/learning-levels/advanced-300/), [Dịch vụ di chuyển cơ sở dữ liệu AWS (AWS DMS)](https://aws.amazon.com/blogs/database/category/database/aws-database-migration-service/), [AWS Well-Architected](https://aws.amazon.com/blogs/database/category/aws-well-architected/),[Thực hành tốt nhất](https://aws.amazon.com/blogs/database/category/post-types/best-practices/) |

[Dịch vụ di chuyển cơ sở dữ liệu AWS](https://aws.amazon.com/dms/) (AWS DMS) đơn giản hóa việc di chuyển và sao chép cơ sở dữ liệu, cung cấp giải pháp được quản lý cho khách hàng. Quan sát của chúng tôi qua nhiều lần triển khai doanh nghiệp cho thấy việc đầu tư thời gian vào việc lập kế hoạch di chuyển cơ sở dữ liệu chủ động mang lại lợi ích đáng kể. Các tổ chức áp dụng chiến lược thiết lập toàn diện luôn gặp ít gián đoạn hơn và đạt được kết quả di chuyển tốt hơn.

Trong bài viết này, chúng tôi trình bày các biện pháp chủ động để tối ưu hóa việc triển khai AWS DMS ngay từ giai đoạn thiết lập ban đầu. Bằng cách sử dụng kế hoạch chiến lược và tầm nhìn kiến trúc, các tổ chức có thể nâng cao độ tin cậy của hệ thống sao chép, cải thiện hiệu suất và tránh những cạm bẫy thường gặp.

Chúng tôi khám phá các chiến lược và phương pháp hay nhất trong các lĩnh vực chính sau:

1.  Lập kế hoạch và chạy thử nghiệm chứng minh khái niệm (PoC)
2.  Triển khai kiểm tra lỗi hệ thống
3.  Xây dựng quy trình vận hành tiêu chuẩn (SOP)
4.  Giám sát và cảnh báo
5.  Áp dụng các nguyên tắc của [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)

## Lập kế hoạch và thực hiện PoC

Việc thực hiện PoC giúp phát hiện và khắc phục sớm các vấn đề về môi trường. Nó cũng giúp tạo ra thông tin mà bạn có thể sử dụng để ước tính tổng thời gian di chuyển và cam kết tài nguyên của mình.

Sau đây là các bước cơ bản để thực hiện PoC thành công:

1.  Lên kế hoạch và triển khai môi trường thử nghiệm với các instances sao chép, tasks và endpoints AWS DMS phù hợp. Để biết thêm thông tin về việc lập kế hoạch và cung cấp tài nguyên, bạn có thể tham khảo [Thực hành tốt nhất cho Dịch vụ di chuyển cơ sở dữ liệu AWS (AWS DMS).](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html)
2.  Sử dụng khối lượng công việc tương tự như trong môi trường sản xuất. Điều bắt buộc là phải mô phỏng môi trường sản xuất của bạn càng sát càng tốt để tăng khả năng gặp phải nhiều vấn đề khác nhau.
3.  Thực hiện thử nghiệm lỗi dựa trên các tình huống được thảo luận trong bảng ở phần tiếp theo.
4.  Theo dõi việc sử dụng tài nguyên và các điểm nghẽn xảy ra trong PoC và xem xét lại kế hoạch và triển khai tài nguyên cho phù hợp.
5.  Ghi lại các quan sát của bạn và thực hiện đánh giá di chuyển bằng cách so sánh với kết quả kinh doanh. Việc này bao gồm đánh giá thời gian phục hồi sau di chuyển và Thỏa thuận Mức Dịch vụ (SLA) của ứng dụng cho cả hoạt động di chuyển và hoạt động kinh doanh đang diễn ra. Nếu các yêu cầu di chuyển và vận hành này không được đáp ứng, hãy xem xét lại giai đoạn lập kế hoạch để đảm bảo phù hợp với nhu cầu kinh doanh của bạn.

## Thực hiện thử nghiệm lỗi có hệ thống

Tất cả các hệ thống, bất kể độ bền bỉ ra sao, đều có thể gặp sự cố và thời gian ngừng hoạt động. Đối với các tổ chức vận hành khối lượng công việc quan trọng, việc lập kế hoạch chủ động là điều cần thiết để duy trì tính liên tục của hoạt động kinh doanh và đáp ứng các Thỏa thuận Dịch vụ (SLA). Phần này cung cấp một khuôn khổ chiến lược để phát triển các Quy trình Vận hành Chuẩn (SOP) thiết lập các giao thức phục hồi rõ ràng và giảm thiểu tác động vận hành trong trường hợp gián đoạn hệ thống.

Khi triển khai AWS DMS, việc hiểu rõ các điểm lỗi tiềm ẩn trở nên vô cùng quan trọng để xây dựng các hệ thống linh hoạt. Bảng sau đây phác thảo các tình huống lỗi thường gặp trong các hệ thống sao chép AWS DMS, làm nền tảng cho chiến lược kiểm thử của bạn. Mặc dù bảng này khá toàn diện, chúng tôi khuyến khích bạn mở rộng các tình huống này dựa trên kiến trúc, yêu cầu tuân thủ và mục tiêu kinh doanh cụ thể của mình để có thể bao quát toàn diện các chế độ lỗi tiềm ẩn trong môi trường của bạn.

| Điểm thất bại | Các tình huống có khả năng ngừng hoạt động | Kiểm tra | Chiến lược giảm thiểu tiềm năng |
| :---: | :---: | :---: | :---: |
| Cơ sở dữ liệu nguồn và đích | Nút thắt hiệu suất trên máy chủ cơ sở dữ liệu như CPU cao, hạn chế bộ nhớ. | Tạo áp lực tải lớn lên hệ thống bằng công cụ đánh giá chuẩn như [sysbench](https://aws.amazon.com/blogs/database/running-sysbench-on-rds-mysql-rds-mariadb-and-amazon-aurora-mysql-via-ssl-tls/) để mô phỏng tải trọng cao trên máy chủ cơ sở dữ liệu. | Bạn có thể cung cấp một nút cơ sở dữ liệu chỉ đọc cho các công cụ mà AWS DMS hỗ trợ, sử dụng bản sao đọc làm nguồn. Để biết thêm thông tin, hãy tham khảo mục [Nguồn di chuyển dữ liệu](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.html). Bạn cũng có thể mở rộng tài nguyên cơ sở dữ liệu và tối ưu hóa các tham số cơ sở dữ liệu. |
| | Các vấn đề truy cập dữ liệu do không đủ quyền | Dùng tài khoản database cho AWS DMS nhưng tài khoản này không được cấp đủ quyền. | Tạo người dùng cơ sở dữ liệu theo nguyên tắc least-privilege. Tham khảo [tài liệu](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.html) điểm cuối nguồn AWS DMS tương ứng để biết các đặc quyền bắt buộc của DMS cho từng công cụ cơ sở dữ liệu. |
| | Failover cơ sở dữ liệu (nếu sử dụng thiết lập chính hoặc dự phòng) | Thực hiện chuyển đổi dự phòng cơ sở dữ liệu từ node chính sang node phụ. | Trong trường hợp AWS DMS có thể cố kết nối lại vào primary cũ sau quá trình failover, hành vi này phụ thuộc vào Thời gian Sống (TTL). Bạn sẽ cần khởi động lại task sau khi TTL được làm mới. Tham khảo [Tại sao kết nối của tôi lại được chuyển hướng đến phiên bản trình đọc khi tôi cố kết nối với điểm cuối trình ghi Amazon Aurora của mình?](https://repost.aws/knowledge-center/aurora-mysql-redirected-endpoint-writer) |
| | Tắt cơ sở dữ liệu HOẶC xảy ra sự cố | Dừng cơ sở dữ liệu đang sao chép DMS. | Ghi lại toàn bộ hành vi của task DMS nhằm phục vụ việc xây dựng SOP, sau đó tiếp tục task khi sự cố cơ sở dữ liệu đã được khắc phục. |
|  | Không có sẵn nhật ký giao dịch | Xóa nhật ký có thời gian lưu giữ ngắn hơn khi task ngoại tuyến hoặc đang chạy chậm. | Ghi lại các quan sát của task DMS để tạo SOP và tiếp tục task sau khi tạo nhật ký giao dịch hoặc thực hiện tải đầy đủ mới nếu nhật ký không khả dụng. |
|  | Thực hiện các thay đổi về cấu trúc như lược đồ, bảng, chỉ mục, phân vùng và kiểu dữ liệu | Chạy các câu lệnh ngôn ngữ định nghĩa dữ liệu (DDL) khác nhau để sửa đổi bảng có liên quan. | Xem danh sách các [câu lệnh DDL được hỗ trợ](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Introduction.SupportedDDL.html) and [cài đặt task.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TaskSettings.DDLHandling.html) |
| Lỗi mạng (áp dụng cho nguồn và đích) | Các vấn đề về kết nối bao gồm lỗi mạng, DNS và SSL | Xóa IP nguồn khỏi nhóm bảo mật AWS DMS HOẶC sửa đổi iptables; Xóa chứng chỉ khỏi điểm cuối DMS; Sửa đổi giá trị MTU (đơn vị truyền tối đa). | Tham khảo cách khắc phục sự cố [kết nối điểm cuối AWS DMS](https://repost.aws/knowledge-center/dms-endpoint-connectivity-failures) và [sự cố khi kết nối với Amazon RDS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Troubleshooting.html#CHAP_Troubleshooting.General.RDSConnection) |
|  | Mất gói tin | Sử dụng lệnh kiểm soát lưu lượng (tc) trên hệ thống Linux HOẶC sử dụng AWS Fault Injection Simulator (FIS). | Tham khảo phần [Khắc phục sự cố mạng trong quá trình di chuyển cơ sở dữ liệu bằng AMI hỗ trợ chẩn đoán AWS DMS](https://aws.amazon.com/blogs/database/troubleshoot-networking-issues-during-database-migration-with-the-aws-dms-diagnostic-support-ami/) và [Làm việc với AMI hỗ trợ chẩn đoán AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_SupportAmi.html) |
| Lỗi AWS DMS | lỗi Khởi động lại phiên bản sao chép Single-AZ | Khởi động lại phiên bản sao chép AWS DMS. | DMS sẽ tự động tiếp tục các task sau khi khởi động lại phiên bản sao chép. |
|  | Khởi động lại phiên bản Multi-AZ Replication với khả năng chuyển đổi dự phòng trong quá trình sao chép đang diễn ra | Khởi động lại phiên bản sao chép AWS DMS, chọn tùy chọn “Khởi động lại với chế độ dự phòng đã lên kế hoạch?”. | DMS sẽ tự động tiếp tục các task sau khi chuyển đổi dự phòng Multi-AZ của phiên bản sao chép. |
|  | Bộ nhớ EBS đầy | Cho phép ghi nhật ký gỡ lỗi chi tiết cho nhiều thành phần nhật ký dẫn đến đầy bộ nhớ do AWS DMS logs. | Thiết lập cảnh báo khi dung lượng lưu trữ đạt 80% và mở rộng dung lượng lưu trữ được liên kết với phiên bản sao chép DMS. Để biết thêm thông tin, hãy tham khảo [Tại sao phiên bản DB sao chép AWS DMS của tôi lại ở trạng thái đầy dung lượng lưu trữ?](https://repost.aws/knowledge-center/dms-replication-instance-storage-full) |
|  | Áp dụng các thay đổi trong cửa sổ bảo trì | Sửa đổi cấu hình cho phiên bản sao chép DMS của bạn dẫn đến thời gian ngừng hoạt động và chọn tùy chọn “Áp dụng trong cửa sổ bảo trì theo lịch trình tiếp theo”. | DMS tự động tiếp tục các task sau khi bảo trì. |
| | Tranh chấp tài nguyên trên phiên bản sao chép (CPU cao, tranh chấp bộ nhớ, IOPS cao hơn mức cơ bản) | Tạo nhiều task DMS có giá trị cao cho MaxFullLoadSubTasks trên một phiên bản sao chép DMS nhỏ. | Thiết lập giám sát và cảnh báo trên các số liệu quan trọng của CloudWatch, như đã thảo luận trong phần Giám sát và cảnh báo. Mở rộng lớp phiên bản hoặc bạn có thể di chuyển các task sang một phiên bản sao chép mới. |
|  | Nâng cấp phiên bản sao chép DMS | Nâng cấp phiên bản sao lưu DMS. DMS không còn hỗ trợ các phiên bản DMS cũ, do đó người dùng cần nâng cấp phiên bản sao lưu. Để biết thêm thông tin, vui lòng tham khảo [Ghi chú phát hành AWS DMS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_ReleaseNotes.html) | Để giảm thiểu thời gian ngừng hoạt động liên quan đến hoạt động này, chúng tôi khuyên bạn nên thực hiện một PoC kỹ lưỡng. Sau khi kiểm tra PoC, bạn có thể lên kế hoạch tạo các phiên bản sao chép mới chạy trên phiên bản DMS mới nhất và di chuyển tất cả các task của mình vào các giờ cao điểm thấp khi độ trễ thu thập dữ liệu thay đổi (CDC) bằng 0 hoặc tối thiểu. Để biết thêm thông tin, hãy tham khảo [Di chuyển một nhiệm vụ.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.Moving.html) Bạn cũng có thể tham khảo [Thực hiện nâng cấp song song trong AWS DMS bằng cách di chuyển các task để giảm thiểu tác động đến hoạt động kinh doanh.](https://aws.amazon.com/blogs/database/perform-a-side-by-side-upgrade-in-aws-dms-by-moving-tasks-to-minimize-business-impact/) |
| Các vấn đề về dữ liệu | Sao chép dữ liệu | Chạy một task chỉ tải đầy đủ hai lần, lần đầu tiên dừng task ở giữa và lần thứ hai chạy task với cấu hình "DO NOTHING" cho chế độ chuẩn bị bảng Mục tiêu. | Sử dụng xác thực DMS cho các công cụ cơ sở dữ liệu được hỗ trợ. Nếu xác thực báo cáo bất kỳ sự không khớp nào, bạn cần điều tra dựa trên lỗi chính xác. Để giảm thiểu sự cố, bạn có thể thực hiện backfill bằng cách tạo task tải đầy đủ hoặc tải lại bảng (nếu có) cho các bảng cụ thể, sau đó tạo task sao chép liên tục. |
|  | Mất dữ liệu | Tạo các kích hoạt trên mục tiêu để xóa hoặc cắt bớt các bản ghi ngẫu nhiên. | Chúng tôi khuyên bạn nên sử dụng xác thực DMS để tránh những vấn đề này. Bạn có thể thực hiện tải lại bảng hoặc task, hoặc tạo một task tải đầy đủ mới và thay đổi task thu thập dữ liệu để thực hiện tải dữ liệu mới cho (các) bảng bị ảnh hưởng.. |
|  | Lỗi bảng | Có được khóa truy cập độc quyền trên các bảng trước khi bắt đầu task DMS HOẶC sử dụng các kiểu dữ liệu không được hỗ trợ. | Sự cố này có thể do tính năng hoặc cấu hình không được hỗ trợ của DMS. Cần phải điều tra dựa trên lỗi chính xác. Để biết thêm thông tin, vui lòng tham khảo [Tại sao task AWS DMS của tôi lại ở trạng thái lỗi?](https://repost.aws/knowledge-center/dms-task-error-status) |
| Các vấn đề về độ trễ | Tích lũy tệp hoán đổi trên phiên bản sao chép | Bắt đầu các giao dịch dài hạn với số lượng thay đổi lớn và theo dõi. số liệu CloudWatch [CDCChangesDiskSource](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html). | Theo dõi các số liệu CDCChangesDiskSource và CDCChangesDiskTarget. Tham khảo bài viết trong Trung tâm Kiến thức này để biết cách tạo SOP: [Tệp hoán đổi là gì và tại sao các tệp này lại chiếm dung lượng trên phiên bản AWS DMS của tôi?](https://repost.aws/knowledge-center/dms-swap-files-consuming-space) |
|  | Lỗi BatchApply | Xóa một bản ghi trên mục tiêu và cập nhật bản ghi đó trên nguồn bằng cách sử dụng BatchApply trên task. | Thiết lập cảnh báo trên nhật ký DMS CloudWatch cho thao tác áp dụng hàng loạt không thành công, vui lòng tham khảo phần Giám sát và cảnh báo để biết hướng dẫn chi tiết. Để khắc phục sự cố và tạo SOP, vui lòng tham khảo bài viết này trong Trung tâm Kiến thức: [Làm thế nào để tôi có thể khắc phục sự cố tại sao Amazon Redshift chuyển sang chế độ từng cái một?](https://repost.aws/knowledge-center/dms-task-redshift-bulk-operation) |
| Các vấn đề xác thực dữ liệu | Nguồn bị thiếu | Những điều này có thể được mô phỏng do thiếu dữ liệu về nguồn và mục tiêu. | Xem lại trường hợp sử dụng được hỗ trợ và các hạn chế với [Xác thực dữ liệu AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) và tham khảo bài viết sau trong Trung tâm kiến thức để biết thêm thông tin: [Tại sao quá trình xác thực task AWS DMS của tôi không thành công hoặc tại sao quá trình xác thực không tiến triển?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |
|  | Mục tiêu bị mất | Những điều này có thể được mô phỏng do thiếu dữ liệu về nguồn và mục tiêu. | Xem lại trường hợp sử dụng được hỗ trợ và các hạn chế với [Xác thực dữ liệu AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) và tham khảo bài viết sau trong Trung tâm kiến thức để biết thêm thông tin: [Tại sao quá trình xác thực task AWS DMS của tôi không thành công hoặc tại sao quá trình xác thực không tiến triển?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |
|  | Sự khác biệt kỷ lục | Bạn có thể tạo các lược đồ bảng khác nhau trong nguồn và đích để mô phỏng tình huống này. | Xem lại trường hợp sử dụng được hỗ trợ và các hạn chế với [Xác thực dữ liệu AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) và tham khảo bài viết sau trong Trung tâm kiến thức để biết thêm thông tin: [Tại sao quá trình xác thực task AWS DMS của tôi không thành công hoặc tại sao quá trình xác thực không tiến triển?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |
|  | Không tìm thấy khóa chính/khóa duy nhất đủ điều kiện | Xác thực yêu cầu phải có khóa chính hoặc khóa duy nhất trong bảng. LOB và một số kiểu dữ liệu khác không được hỗ trợ với xác thực DMS. Để biết thêm chi tiết, hãy tham khảo [hạn chế xác thực.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html#CHAP_Validating.Limitations) | Xem lại trường hợp sử dụng được hỗ trợ và các hạn chế với [Xác thực dữ liệu AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html) và tham khảo bài viết sau trong Trung tâm kiến thức để biết thêm thông tin: [Tại sao quá trình xác thực task AWS DMS của tôi không thành công hoặc tại sao quá trình xác thực không tiến triển?](https://repost.aws/knowledge-center/dms-task-validation-failed-stuck) |

Bằng cách kiểm tra một cách có hệ thống các kịch bản này và ghi lại kết quả, các tổ chức có thể phát triển các quy trình phục hồi mạnh mẽ, giải quyết cả các chế độ lỗi phổ biến và lỗi đặc thù. Cách tiếp cận chủ động này không chỉ giúp duy trì độ tin cậy của hệ thống mà còn cung cấp cho các nhóm vận hành các giao thức rõ ràng để giải quyết các vấn đề khi chúng phát sinh.

## Phát triển các quy trình vận hành tiêu chuẩn (SOP)

Trong các tình huống kiểm thử lỗi, hãy ghi chép cẩn thận tác động của từng sự cố lên hệ thống sao chép của bạn. Tài liệu này tạo nền tảng cho việc tạo ra các SOP tùy chỉnh mà nhóm của bạn có thể dựa vào khi quản lý việc triển khai AWS DMS. Các chiến lược giảm thiểu được nêu trong khuôn khổ kiểm thử lỗi của chúng tôi đóng vai trò là điểm khởi đầu tuyệt vời để phát triển các quy trình này.

Các SOP ban đầu của bạn sẽ xuất hiện trong giai đoạn đầu của quá trình kiểm thử PoC. Các quy trình này nên được coi là tài liệu sống, đòi hỏi phải cập nhật và tinh chỉnh thường xuyên khi bạn có thêm kinh nghiệm vận hành và gặp phải các tình huống mới. Bản chất năng động của việc di chuyển cơ sở dữ liệu đồng nghĩa với việc các SOP của bạn sẽ phát triển cùng với sự hiểu biết của bạn về hành vi hệ thống và các thách thức mới nổi.

Để được hướng dẫn thêm về cách xử lý các tình huống di chuyển phức tạp, chúng tôi khuyên bạn nên xem lại loạt bài viết gồm ba phần trên blog của chúng tôi về gỡ lỗi di chuyển AWS DMS. Các tài nguyên này cung cấp những hiểu biết quý giá có thể giúp bạn phát triển các phương pháp tiếp cận có hệ thống để khắc phục sự cố, ngay cả đối với các tình huống không được đề cập trong các SOP hiện có của bạn. Bạn có thể tìm thấy các hướng dẫn chi tiết này tại:

1.  [Gỡ lỗi quá trình di chuyển AWS DMS của bạn: Cần làm gì khi có sự cố xảy ra (Phần 1)](https://aws.amazon.com/blogs/database/debugging-your-aws-dms-migrations-what-to-do-when-things-go-wrong-part-1/)
2.  [Gỡ lỗi quá trình di chuyển AWS DMS của bạn: Cần làm gì khi có sự cố xảy ra (Phần 2)](https://aws.amazon.com/blogs/database/debugging-your-aws-dms-migrations-what-to-do-when-things-go-wrong-part-2/)
3.  [Gỡ lỗi quá trình di chuyển AWS DMS của bạn: Cần làm gì khi có sự cố xảy ra (Phần 3)](https://aws.amazon.com/blogs/database/debugging-your-aws-dms-migrations-what-to-do-when-things-go-wrong-part-3/)

Bằng cách ghi chép và kiểm tra các quy trình này, các tổ chức có thể đo lường và xác thực chính xác khả năng đáp ứng SLA của hệ thống sao chép, đặc biệt là trong các trường hợp sự cố nghiêm trọng. Cách tiếp cận chủ động này giúp xác định các điểm nghẽn tiềm ẩn và các lĩnh vực cần cải thiện trong chiến lược phục hồi sau thảm họa, từ đó củng cố khả năng phục hồi và độ tin cậy của kiến trúc sao chép dữ liệu.

Khi thiết kế chiến lược sao chép dữ liệu bằng AWS DMS, điều quan trọng là phải thiết lập các kế hoạch dự phòng toàn diện cho các tình huống liên quan đến việc dịch vụ không khả dụng hoặc sự khác biệt về dữ liệu. Việc đánh giá kỹ lưỡng RTO và RPO của doanh nghiệp sẽ thúc đẩy việc phát triển các SOP. Việc lập kế hoạch chiến lược này không chỉ thúc đẩy tính liên tục của hoạt động kinh doanh mà còn cung cấp những hiểu biết có giá trị về các chỉ số hiệu suất thực tế của hệ thống sao chép trong các tình huống sự cố.

## Giám sát và cảnh báo

Để tối đa hóa hiệu quả của AWS DMS, cần có một phương pháp tiếp cận chiến lược đối với khả năng giám sát và báo cáo. Một khuôn khổ giám sát mạnh mẽ là điều cần thiết để duy trì hoạt động sao chép liền mạch và thúc đẩy tính toàn vẹn dữ liệu trong suốt quá trình di chuyển.

Việc cấu hình các cảnh báo phù hợp trong quá trình thiết lập ban đầu sẽ cung cấp khả năng hiển thị theo thời gian thực đối với các task sao chép và cho phép phản hồi nhanh chóng với các bất thường. Các khả năng giám sát này hoạt động như một hệ thống cảnh báo sớm, giúp duy trì tình trạng hoạt động và hiệu quả của cơ sở hạ tầng di chuyển cơ sở dữ liệu của bạn.

Việc triển khai giám sát và cảnh báo chủ động giúp nâng cao độ tin cậy vận hành, đồng thời cung cấp thông tin chi tiết về việc sử dụng tài nguyên và các mô hình hiệu suất. Phương pháp tiếp cận có hệ thống này cho phép đưa ra quyết định dựa trên dữ liệu và duy trì hiệu suất sao chép tối ưu trong suốt vòng đời di chuyển.

AWS DMS cung cấp các tính năng giám sát sau:

1.  [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) metrics – These metrics are automatically populated by AWS DMS for users to get insights into resource utilization and related metrics for individual task and at replication instance level. For a list of all the metrics available with AWS DMS, refer to [Số liệu của Dịch vụ di chuyển cơ sở dữ liệu AWS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.Metrics)
2.  CloudWatch logs và Time Travel logs của AWS DMS – AWS DMS tạo nhật ký lỗi và điền vào chúng dựa trên mức ghi nhật ký do người dùng thiết lập cho từng thành phần. Để biết thêm thông tin, hãy tham khảo [Xem và quản lý nhật ký task AWS DMS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.ManagingLogs) Khi nhật ký CloudWatch được bật, AWS DMS theo mặc định sẽ bật [ghi nhật ký ngữ cảnh](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#datarep_Monitoring_ContextLogging). Ngoài ra, DMS còn có tính năng ghi nhật ký Du hành Thời gian để hỗ trợ gỡ lỗi các task sao chép. Để biết thêm thông tin về ghi nhật ký Du hành Thời gian, vui lòng tham khảo phần [Cài đặt task Du hành Thời gian](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TaskSettings.TimeTravel.html). Để biết các phương pháp hay nhất khi sử dụng nhật ký Du hành thời gian, hãy tham khảo [Khắc phục sự cố task sao chép bằng Time Travel.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html#CHAP_BestPractices.TimeTravel)
3.  Trạng thái task và bảng – AWS DMS cung cấp bảng thông tin gần như theo thời gian thực để báo cáo trạng thái của task và bảng. Để biết danh sách chi tiết về trạng thái task, vui lòng tham khảo [Trạng thái nhiệm vụ](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Tasks.Status). Để biết trạng thái bảng, hãy tham khảo [Trạng thái bảng trong quá trình thực hiện nhiệm vụ.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Tasks.CustomizingTasks.TableState)
4.  [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) logs– AWS DMS được tích hợp với AWS CloudTrail, một dịch vụ cung cấp bản ghi các hành động được thực hiện bởi người dùng, vai trò hoặc dịch vụ AWS trong AWS DMS. CloudTrail ghi lại tất cả các lệnh gọi API cho AWS DMS dưới dạng sự kiện, bao gồm các lệnh gọi từ bảng điều khiển AWS DMS và từ các lệnh gọi mã đến các hoạt động API của AWS DMS. Để biết thêm thông tin về cách thiết lập CloudTrail, vui lòng tham khảo [Hướng dẫn sử dụng AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/).
5.  Monitoring dashboard – Bảng điều khiển giám sát nâng cao cung cấp khả năng hiển thị toàn diện các số liệu quan trọng liên quan đến task giám sát và phiên bản sao chép của bạn; lọc, tổng hợp và trực quan hóa các số liệu cho các tài nguyên cụ thể bạn muốn theo dõi. Bảng điều khiển trực tiếp xuất bản các số liệu CloudWatch hiện có để giám sát hiệu suất tài nguyên mà không thay đổi thời gian lấy mẫu điểm dữ liệu. Để biết thêm thông tin, vui lòng tham khảo [Tổng quan về bảng điều khiển giám sát nâng cao](https://docs.aws.amazon.com/dms/latest/userguide/enhanced-monitoring-dashboard.html#overview-enhanced-monitoring-dashboard).

Chúng tôi khuyên bạn nên thiết lập cảnh báo CloudWatch trên các số liệu quan trọng và nhật ký sự kiện để chủ động xác định các vấn đề tiềm ẩn trước khi chúng leo thang thành gián đoạn toàn hệ thống. Mặc dù phương pháp giám sát cơ bản này chỉ là điểm khởi đầu, nhưng việc mở rộng chiến lược giám sát dựa trên các yêu cầu cụ thể về trường hợp sử dụng và mục tiêu kinh doanh của bạn là rất quan trọng.

| Loại số liệu | Tên số liệu | Biện pháp khắc phục |
| :---: | :---: | :---: |
| Số liệu máy chủ | Sử dụng CPU | Bạn nên thiết lập cảnh báo dựa trên các số liệu này để cảnh báo người vận hành vì tranh chấp tài nguyên sẽ ảnh hưởng đến hiệu suất task DMS của bạn. Dựa trên giới hạn tài nguyên trên máy chủ, bạn cần nâng cấp lớp phiên bản DMS nếu có tranh chấp CPU và bộ nhớ, hoặc tăng dung lượng lưu trữ nếu dung lượng lưu trữ thấp hoặc IOPS cơ bản đang bị giới hạn. Để biết thêm thông tin về cách chọn phiên bản sao chép phù hợp, bạn có thể tham khảo bài viết [Chọn kích thước tốt nhất cho phiên bản sao chép.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.SizingReplicationInstance.html). |
|  | Bộ nhớ trống |  |
| | SwapUsage |  |
|  | FreeStorageSpace |  |
|  | WriteIOPS |  |
|  | ReadIOPS |  |
| Số liệu nhiệm vụ sao chép | CDCLatencySource | Dựa trên các yêu cầu SLA, bạn có thể thiết lập ngưỡng cảnh báo cho các số liệu về độ trễ. Trong DMS, độ trễ có thể do nhiều nguyên nhân gây ra. Để khắc phục sự cố và tạo SOP, bạn có thể tham khảo [Khắc phục sự cố độ trễ trong Dịch vụ di chuyển cơ sở dữ liệu AWS. (AWS DMS)](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Troubleshooting_Latency.html) |
|  | CDCLatencyTarget |  |
| Sự kiện DMS cho phiên bản sao chép | Thay đổi cấu hình | Mỗi danh mục này là một danh mục với các sự kiện khác nhau được liên kết. Bạn có thể thiết lập thông báo về các sự kiện cụ thể dựa trên nhu cầu của mình. Để biết danh sách chi tiết và mô tả về các sự kiện này, vui lòng tham khảo danh mục [sự kiện AWS DMS và thông báo sự kiện cho thông báo SNS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Events.html#USER_Events.Messages). |
| | Sáng tạo |  |
|  | Xóa |  |
|  | Bảo trì |  |
|  | Lưu trữ thấp |  |
|  | Chuyển đổi dự phòng |  |
|  | Lỗi |  |
| Sự kiện DMS cho các task sao chép | Lỗi | Mỗi danh mục này là một danh mục với các sự kiện khác nhau được liên kết. Bạn có thể thiết lập thông báo về các sự kiện cụ thể dựa trên nhu cầu của mình. Để biết danh sách chi tiết và mô tả về các sự kiện này, vui lòng tham khảo danh mục [sự kiện AWS DMS và thông báo sự kiện cho thông báo SNS.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Events.html#USER_Events.Messages). |
| | Thay đổi trạng thái |  |
|  | Sáng tạo |  |
|  | Xóa | |

Để biết danh sách đầy đủ các số liệu có sẵn, bạn có thể tham khảo Hướng dẫn sử dụng AWS DMS về số liệu của [AWS Database Migration Service](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Monitoring.html#CHAP_Monitoring.Metrics).

Bạn có thể sử dụng [Amazon EventBridge](https://aws.amazon.com/eventbridge/) để cung cấp thông báo khi sự kiện AWS DMS xảy ra hoặc sử dụng [Dịch vụ thông báo đơn giản của Amazon](https://aws.amazon.com/sns/) (Amazon SNS) để tạo cảnh báo cho các sự kiện quan trọng. Để biết thêm thông tin về sự kiện EventBridge trong DMS, hãy tham khảo [Hướng dẫn sử dụng sự kiện EventBridge.](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_EventBridge.html) Để biết thêm thông tin về cách sử dụng Amazon SNS với AWS DMS, hãy tham khảo [Hướng dẫn sử dụng sự kiện Amazon SNS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Events.html).

Ngoài việc thiết lập cảnh báo CloudWatch, bạn có thể tạo cảnh báo tùy chỉnh dựa trên nhật ký lỗi CloudWatch của AWS DMS bằng bộ lọc số liệu. Để biết hướng dẫn chi tiết từng bước về cách triển khai các cảnh báo tùy chỉnh này, hãy tham khảo bài đăng trên blog có tiêu đề [\"Gửi cảnh báo về lỗi AWS DMS tùy chỉnh từ Nhật ký Amazon CLoudWatch\"](https://aws.amazon.com/blogs/database/send-alerts-on-custom-aws-dms-errors-from-amazon-cloudwatch-logs/). Tài nguyên này cung cấp hướng dẫn toàn diện để nâng cao khả năng giám sát lỗi tùy chỉnh của bạn.

## Áp dụng các nguyên tắc của AWS Well-Architected Framework

Khung Kiến trúc Tốt của AWS giúp bạn hiểu rõ ưu và nhược điểm của các quyết định khi xây dựng hệ thống trên đám mây. Sáu trụ cột của khung hướng dẫn bạn các phương pháp kiến trúc tốt nhất để thiết kế và vận hành các hệ thống đáng tin cậy, an toàn, hiệu quả, tiết kiệm chi phí và bền vững.

Khi sử dụng [AWS Well-Architected Tool](https://aws.amazon.com/well-architected-tool/), có sẵn miễn phí trong [AWS Management Console](https://console.aws.amazon.com/wellarchitected),bạn có thể xem xét khối lượng công việc của mình dựa trên các phương pháp hay nhất này bằng cách trả lời một bộ câu hỏi cho từng trụ cột.

Để biết thêm hướng dẫn chuyên môn và các phương pháp hay nhất cho kiến trúc đám mây của bạn—triển khai kiến trúc tham khảo, sơ đồ và sách trắng—hãy tham khảo [AWS Architecture Center](https://aws.amazon.com/architecture/)

## Kết luận

Trong bài viết này, chúng tôi đã trình bày một khuôn khổ toàn diện để xây dựng các triển khai AWS DMS linh hoạt. Hiệu quả của hướng dẫn này liên quan trực tiếp đến mức độ sâu sắc của việc triển khai và khả năng thích ứng với môi trường cụ thể của bạn. Chúng tôi đặc biệt khuyến khích các tổ chức xem xét kỹ lưỡng từng phần của hướng dẫn này và sử dụng nó làm nền tảng để phát triển một chiến lược di chuyển tùy chỉnh phù hợp với trường hợp sử dụng riêng của bạn.

Bằng cách đánh giá cẩn thận và kết hợp các khuyến nghị này vào quy trình lập kế hoạch di chuyển, bạn có thể phát triển một phương pháp tiếp cận toàn diện và đáng tin cậy để sử dụng AWS DMS, tạo điều kiện cho sự thành công lâu dài trong các chiến lược di chuyển dữ liệu của bạn.

Để được hỗ trợ và tài nguyên bổ sung, hãy truy cập [AWS DMS documentation](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html) và tham gia với [AWS Support](https://aws.amazon.com/contact-us/).

## Giới thiệu về tác giả

**Sanyam Jain** là Kỹ sư Cơ sở dữ liệu của nhóm Dịch vụ Di chuyển Cơ sở dữ liệu AWS (AWS DMS). Anh hợp tác chặt chẽ với khách hàng, cung cấp hướng dẫn kỹ thuật để di chuyển khối lượng công việc tại chỗ lên Đám mây AWS. Ngoài ra, anh còn đóng vai trò then chốt trong việc nâng cao chất lượng và chức năng của các sản phẩm di chuyển dữ liệu AWS.

**Sushant Deshmukh** là Kiến trúc sư Giải pháp Đối tác Cấp cao, làm việc với các Nhà tích hợp Hệ thống Toàn cầu. Anh đam mê thiết kế các kiến trúc có tính khả dụng cao, khả năng mở rộng, linh hoạt và bảo mật trên AWS. Anh hỗ trợ Khách hàng và Đối tác AWS di chuyển và hiện đại hóa thành công các ứng dụng của họ lên Đám mây AWS. Ngoài công việc, anh thích đi du lịch khám phá những địa điểm và nền ẩm thực mới, chơi bóng chuyền và dành thời gian chất lượng cho gia đình và bạn bè.

**Alex Anto** là Kiến trúc sư Giải pháp Chuyên gia Di chuyển Dữ liệu thuộc nhóm Amazon Database Migration Accelerator tại Amazon Web Services. Anh làm việc với tư cách là Cố vấn Amazon DMA, hỗ trợ khách hàng AWS di chuyển dữ liệu tại chỗ sang các giải pháp cơ sở dữ liệu AWS Cloud.
---
title: "Event 2"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---


# Bài thu hoạch “BUILDING AGENTIC AI”

### Mục Đích Của Sự Kiện

- Tối ưu context với Amazon Bedrock
- Xây dựng các AI agent automation với Amazon Bedrock thông qua các kỹ thuật thực hành và các trường hợp sử dụng trong thế giới thực

### Danh Sách Diễn Giả

- **Nguyen Gia Hung** - Head of Solutions Architect, AWS
- **Kien Nguyen** - Solutions Architect, AWS
- **Viet Pham** - Founder & CEO, Diagflow
- **Kha Van** - Community Leader, AWS
- **Thang Ton** - Co-Founder & COO, Cloud Thinker
- **Henry Bui** - Head of Engineering, Cloud Thinker

### Nội Dung Nổi Bật

#### Các kỹ thuật tối ưu hóa chi phí và hiệu suất cho các hệ thống AI agent

- Thời gian release sản phẩm lâu → Mất doanh thu/bỏ lỡ cơ hội
- Hoạt động kém hiệu quả → Mất năng suất, tốn kém chi phí
- Không tuân thủ các quy định về bảo mật → Mất an ninh, uy tín

#### Bốn kỹ thuật “Quick Wins” để tối ưu hóa

#### Prompt Caching (Lưu trữ bộ nhớ đệm cho Prompt)
Đây là kỹ thuật quan trọng nhất được đề cập, có thể giảm 70-90% chi phí và tăng tốc độ xử lý: Cấu trúc Context: Context window được chia thành 3 phần: (1) System & Tool Schema, (2) Conversation History (Lịch sử hội thoại), và (3) Objective Prompt (Mục tiêu hiện tại). Sai lầm thường gặp: Đa số mọi người chỉ cache phần System Prompt và Tool. Tuy nhiên, phần Conversation History mới là phần tốn kém nhất (có thể chiếm 80-90% chi phí) và thường bị bỏ qua. Chiến lược đúng: Cần đặt “checkpoint” sao cho toàn bộ lịch sử hội thoại cũng được cache. Mặc dù lần chạy đầu tiên (cache write) tốn thêm 25% chi phí, nhưng các lần sau sẽ tiết kiệm được 90%

#### Context Compaction (Nén ngữ cảnh - Summarization)

Cloud Thinker đã chỉ ra 1 kỹ thuật tóm tắt (Summarization) thông minh để tránh mất cache Cách cũ: Tạo một agent mới để tóm tắt hội thoại cũ. Cách này làm mất toàn bộ cache trước đó và thường làm giảm chất lượng (performance degradation). Cách mới (Cloudthinker technique): Giữ nguyên agent và lịch sử hội thoại (để tận dụng cache đã có), chỉ thay đổi phần “Objective Prompt” thành một task mới là “hãy tóm tắt đoạn hội thoại này”. Cách này giúp tận dụng cache hit, giảm chi phí tóm tắt từ ví dụ 0.3 đô xuống còn 0.03 đô (giảm ~90%) và cải thiện chất lượng đầu ra

#### Tool Consolidation (Hợp nhất công cụ) 

Vấn đề với các giao thức mới như MCP (Model Context Protocol) là việc đưa quá nhiều công cụ (ví dụ 50 tool) vào context sẽ làm tràn bộ nhớ (context flooding). Giải pháp: Thay vì đưa toàn bộ schema (cấu trúc dữ liệu) phức tạp vào prompt, hãy sử dụng một “dictionary” đơn giản và gộp các instruction (hướng dẫn) lại. Just-in-time Instruction: Agent có thể sử dụng một lệnh đặc biệt (get instruction) để lấy hướng dẫn chi tiết về cách dùng công cụ chỉ khi cần thiết. Điều này giúp giảm số lượng token phải nạp vào input liên tục

#### Parallel Tool Calling (Gọi công cụ song song)

Thay vì chạy tuần tự (sequential) như mô hình ReAct cũ (năm 2022), các model hiện đại cho phép chạy song song nhiều tool cùng lúc để tiết kiệm thời gian. Tuy nhiên, tính năng này thường không bật mặc định; lập trình viên cần thêm các câu lệnh instruction cụ thể (ví dụ: “maximize efficiency”) để ép model chạy song song

### Những Gì Học Được

#### Chiến lược quản lý chi phí

- **Input cost:** chiếm phần lớn chi phí vận hành AI agents chạy loop. Tức là mỗi vòng, Agent phải nạp lại toàn bộ context (Conversation history, system prompt, memory) Nên loop Agent chạy càng lâu thì sẽ càng đốt tiền -> History càng dài thì input tokens mỗi vòng càng tăng.
- Solution: dùng Prompt Caching và checkpointing để giảm thiểu chi phí này. Giảm cost lên đến 80-90%.

#### Kỹ thuật “Tóm tắt” (Summarization) thông minh

Summarization: Giữ nguyên agent hiện tại và lịch sử hội thoại để tận dụng cache đã có, và giữ được context quality tốt hơn. Với kỹ thuật này, chi phí tóm tắt giảm từ 0.3 đô xuống còn 0.03 đô (giảm ~90%) và chất lượng đầu ra được cải thiện.

#### Thiết kế công cụ (Tool Design): Tránh “ngập lụt” ngữ cảnh
- Tool Design:
   - Vấn đề: Quá nhiều tools được đưa vào (ví d: MCP với 50 tools) sẽ làm tràn ngữ cảnh (context flooding).
   - Giải pháp: Tạo 1 lệnh đặc biệt để Agent tự gọi đến lấy instruction chi tiết khi cần, thay vì nhồi nhét toàn bộ schema vào prompt.
   - Giúp context gọn nhẹ, giảm token usage và tăng hiệu suất.

#### Tối ưu hiệu suất: Bắt buộc chạy song song (Parallel Tool Calling)
- Maximize Efficiency: thêm hướng dẫn cụ thể vào prompt để ép model Implement các tác vụ song song thay vì tuần tự.


### Trải nghiệm trong event

Tham gia workshop “Building Agentic AI” là một trải nghiệm rất thú vị, nâng cao kiến thức về Agentic AI. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả có chuyên môn cao
- Các diễn giả đến từ AWS, Cloudthinker, Diaflow đã chia sẻ best practices trong thiết kế ứng dụng hiện đại.


#### Một số hình ảnh khi tham gia sự kiện
![event2](/images/5.12-event.jpg)


---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
Tại phần này, chúng tôi trình bày chi tiết bản đề xuất kỹ thuật và quá trình triển khai hệ thống Pedix.

# Pedix - Trợ lý Hướng dẫn Chăm sóc Sức khỏe Nhi khoa Agentic RAG
## Hệ thống Agentic RAG Phân tầng Độ tuổi Level 4 Triển khai trên AWS

### 1. Tóm tắt điều hành
**Pedix** là ứng dụng web trợ lý sức khỏe AI được triển khai hoàn chỉnh trên đám mây AWS, thiết kế chuyên biệt cho phụ huynh và người chăm sóc trẻ em trong độ tuổi **từ 0 đến 5 tuổi (nhi khoa under-5)**. Khác với các công cụ tra cứu triệu chứng thông thường hoặc chatbot trả lời tĩnh, Pedix vận hành quy trình suy luận **Level 4 Agentic Retrieval-Augmented Generation (RAG)** phân tầng theo độ tuổi, kết hợp sức mạnh của Amazon Bedrock (Claude Haiku), cơ sở dữ liệu vector Qdrant và 12 dịch vụ đám mây AWS. Hệ thống đưa ra khuyến nghị lộ trình chăm sóc y tế, phân loại mức độ khẩn cấp (theo chuẩn ESI v4) và danh mục chuẩn bị trước khi khám bệnh dựa trên hướng dẫn lâm sàng chuẩn mực từ WHO và NICE. Toàn bộ kiến trúc được tối ưu hóa cực hạn để tận dụng AWS Free Tier, giúp chi phí vận hành chỉ ở mức ~$72.7/tháng.

### 2. Tuyên bố vấn đề
#### Vấn đề hiện tại
Các triệu chứng ở trẻ nhỏ (như sốt hoặc khó thở) đòi hỏi phản ứng lâm sàng hoàn toàn khác nhau tùy thuộc vào độ tuổi chính xác: Sốt 38°C ở trẻ sơ sinh 2 tuần tuổi là một tình trạng cấp cứu y tế cần đưa đến phòng cấp cứu nhi khoa ngay lập tức, trong khi cùng mức nhiệt độ đó ở trẻ 3 tuổi thường có thể theo dõi và xử lý tại nhà. Các ứng dụng kiểm tra triệu chứng hiện nay thường sử dụng mô hình thích ứng từ người lớn hoặc pipeline cố định, không áp dụng được ngưỡng phân tầng độ tuổi, dẫn đến việc đánh giá quá thấp nguy cơ nguy hiểm hoặc gây ra các chuyến đi cấp cứu không cần thiết.

#### Giải pháp
Pedix giới thiệu kiến trúc **Agentic RAG Phân tầng Độ tuổi Level 4**:
- **Giai đoạn 0 (Safety Screen):** Bộ lọc an toàn định tính kiểm tra các dấu hiệu cấp cứu đe dọa tính mạng trong <10ms trước khi gọi LLM.
- **Giai đoạn 1 (Query Analysis):** Ưu tiên xác định chính xác tuổi của trẻ và ánh xạ vào các nhóm tuổi.
- **Giai đoạn 2 (Retrieval):** Lọc metadata `age_group` trực tiếp trên Qdrant trước khi tìm kiếm vector tương đồng.
- **Giai đoạn 3 & 4 (Reasoning & Reflection):** Suy luận lộ trình chăm sóc dựa trên ESI v4 và tự đánh giá độ hoàn thiện (vòng lặp reflection).
- **Giai đoạn 5 (Output):** Tạo câu trả lời an tâm, minh bạch cho phụ huynh kèm nguồn trích dẫn.

#### Lợi ích và Hoàn vốn Đầu tư (ROI)
- **An toàn Lâm sàng & Minh bạch:** Vết suy luận (Reasoning Trace) hiển thị trực quan cùng nguồn trích dẫn WHO/NICE giúp phụ huynh tin tưởng, xóa bỏ rào cản "hộp đen AI".
- **Quản trị Chi phí trên AWS:** Tối ưu hóa hạ tầng production chạy trên EC2 (t3.micro), Qdrant Container, DynamoDB On-Demand và API Gateway VPC Link với chi phí **~$72.7/tháng**.
- **Khả năng Mở rộng:** Quản trị viên có thể cập nhật tài liệu y khoa vào hệ thống vector mà không cần phải redeploy lại toàn bộ backend.

### 3. Kiến trúc Giải pháp Chi tiết
Pedix vận hành trên hạ tầng AWS chuẩn Production áp dụng kiến trúc mạng Zero-Trust đặt tại Region `ap-southeast-1` (Singapore):

![Kiến trúc AWS Pedix](/images/2-Proposal/pedix_architecture.png)

#### Phân lập Mạng Zero-Trust & Các Dịch Vụ AWS
- **Amazon S3 + CloudFront:** Lưu trữ giao diện React (Vite). Bucket S3 (`pedix-frontend-prod`) được đóng hoàn toàn khỏi mạng internet công cộng (Block All Public Access) và chỉ phân phối thông qua CloudFront Origin Access Control (OAC).
- **Amazon API Gateway (Regional REST):** Chịu trách nhiệm xác thực token bằng Cognito JWT Authorizer và tạo đường hầm (tunnel) vào VPC nội bộ thông qua VPC Link V2 (`fzvy02`). Nó hỗ trợ native Server-Sent Events (SSE) để truyền dữ liệu thời gian thực cho UI.
- **Application Load Balancer (Internal ALB):** Hoạt động dưới dạng Internal Scheme (không có IP Public), kết nối an toàn từ API Gateway tới EC2 backend.
- **Amazon EC2 (t3.micro + 2GB Swap):** Chạy FastAPI server trên IP nội bộ (`172.31.42.140`). Áp dụng kỹ thuật Security Group Chaining, port 8000 của EC2 chỉ chấp nhận traffic từ ALB, loại bỏ 100% rủi ro truy cập trái phép từ bên ngoài.
- **Qdrant Vector DB (Docker v1.10.1):** Database vector chuyên dụng chứa dữ liệu guideline lâm sàng. Container được cấu hình bind trực tiếp vào `127.0.0.1:6333` nhằm cô lập tuyệt đối khỏi các EC2 khác trong cùng VPC.
- **Amazon Bedrock (Claude Haiku):** Xử lý mọi logic suy luận, reflection và tổng hợp bằng model `global.anthropic.claude-haiku-4-5-20251001-v1:0`.
- **Amazon DynamoDB:** Bốn bảng On-Demand (không tính phí duy trì) lưu trữ sessions, profiles của trẻ, nhật ký phân tích và metadata tài liệu.
- **Amazon Cognito + AWS Lambda:** Quản trị danh tính. Lambda Post-Confirmation tự động gán user sau khi đăng ký vào group `pedix-users`.

### 4. Triển khai Kỹ thuật
#### Các giai đoạn triển khai
1. **Nghiên cứu & Mô hình hóa (Tháng 1):** Thu thập tài liệu WHO & NICE CG160, xây dựng bộ lọc dấu hiệu cấp cứu Stage 0 và ma trận phân tầng độ tuổi.
2. **Thiết kế Kiến trúc & Tối ưu Chi phí (Tháng 2):** Thay thế OpenSearch Serverless bằng Qdrant Docker trên EC2 để tiết kiệm chi phí nền tảng ~$60/tháng. Phân bổ 2GB EBS Swap để khắc phục lỗi OOM (Out-Of-Memory) của EC2 khi xử lý các chunk văn bản lớn.
3. **Phát triển Backend & SSE Streaming (Tháng 2-3):** Khắc phục lỗi timeout của CloudFront/ALB bằng cách tích hợp heartbeat frame ngầm định 5 giây/lần. Cấu hình FastAPI worker xuống `--workers 1` để đồng bộ state khi xử lý SSE request bất đồng bộ.
4. **Triển khai Hệ thống Zero-Trust (Tháng 3):** Build infrastructure thông qua AWS CLI/Console, thiết lập Security Group Chaining và VPC Link chuẩn xác.

### 5. Lộ trình & Mốc triển khai
- **Tháng 5/2026:** Thu thập yêu cầu, định hình kiến trúc AWS Cloud, nghiên cứu WHO guideline.
- **Tháng 6/2026:** Code FastAPI backend, tích hợp Qdrant, viết prompt tool-use cho Bedrock, thiết kế DynamoDB và giao diện React.
- **Tháng 7 & 8/2026:** Triển khai đám mây AWS toàn diện, xử lý VPC Link, tối ưu SSE heartbeat, thiết lập phân lập mạng Zero-Trust, kiểm thử End-to-End.

### 6. Ước tính Ngân sách Production (~100 người dùng)
Hệ thống được thiết kế tinh giản tối đa, tận dụng AWS Free Tier (Region Singapore):

| Dịch vụ | Cấu hình | Chi phí ước tính / Tháng |
|---|---|---|
| EC2 (t3.micro) | Linux Ubuntu 2 GiB RAM + 2 GiB Swap | ~$9.50 |
| Public IPv4 | IPv4 tĩnh gán cho EC2 | $3.65 |
| Ổ cứng EBS | 30 GiB gp3 storage | ~$3.00 |
| Internal ALB | HTTP listener + Target group | ~$24.24 |
| VPC Link V2 | REST API Private Integration | $18.25 |
| Bedrock Inference | Claude Haiku (~800 truy vấn) | ~$14.00 |
| DynamoDB | On-Demand (4 bảng dưới 25GB) | <$0.10 |
| CloudFront & S3 | Băng thông CDN & lưu trữ file tĩnh | ~$0.05 |
| Cognito & CloudWatch | Trong hạn mức Free Tier | $0.00 |
| **Tổng cộng** | | **~$72.7 / tháng** |

### 7. Đánh giá Rủi ro
- **Bỏ sót Dấu hiệu Cấp cứu:** Khắc phục bằng bộ lọc Stage 0 định tính (<10ms) chuyển hướng thẳng tới hướng dẫn cấp cứu đối với các trường hợp nguy cơ cao.
- **Timeout SSE trên ALB / API Gateway:** Khắc phục bằng cơ chế phát heartbeat giữ kết nối sống sót trong lúc Bedrock suy luận logic phức tạp.
- **Rò rỉ Bảo mật & Bỏ qua Xác thực:** Khắc phục triệt để thông qua cơ chế phòng thủ chiều sâu (Defense-in-depth): xác thực JWT bằng API Gateway, xác minh JWKS nội bộ của FastAPI và khóa truy cập bằng Security Group Chaining.

### 8. Kết quả Kỳ vọng
- **Hệ thống Agentic RAG Level 4 thực tế:** Ứng dụng trợ lý nhi khoa thông minh, hoạt động trực tiếp trên đám mây AWS.
- **AI Y tế Minh bạch:** Giao diện truyền dẫn vết suy luận SSE trực quan, cho phép người dùng giám sát toàn bộ quá trình ra quyết định của bot.
- **Kiến trúc Cloud Hoàn chỉnh:** Mô hình hệ thống an toàn tuyệt đối, phân tách ranh giới mạng nghiêm ngặt và tối ưu vượt trội về chi phí cho doanh nghiệp.
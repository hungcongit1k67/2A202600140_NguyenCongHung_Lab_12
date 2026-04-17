# Day 12 Lab - Final Report

**Họ và tên:** Nguyễn Công Hùng  
**MSSV:** 2A202600140  

---

## Part 1: Localhost vs Production (12-Factor App)

### Exercise 1.1: Anti-patterns Found in the Basic Version

Sau khi phân tích phiên bản cơ bản, em xác định được các anti-patterns chính như sau:

1. **Hardcoded Secrets**  
   API key được khai báo trực tiếp trong source code, làm tăng nguy cơ rò rỉ thông tin nhạy cảm khi chia sẻ mã nguồn hoặc push lên Git repository.

2. **Fixed Port Configuration**  
   Ứng dụng sử dụng cố định cổng `8000`, thiếu tính linh hoạt khi triển khai trên các nền tảng cloud, nơi cổng thường được cấp phát thông qua biến môi trường.

3. **Debug Mode Enabled**  
   Chế độ debug luôn được bật, có thể làm lộ stack trace và thông tin nội bộ của hệ thống khi xảy ra lỗi, từ đó tạo thêm rủi ro bảo mật.

4. **Missing Health Checks**  
   Ứng dụng không cung cấp các endpoint như `/health` hoặc `/ready`, khiến container orchestrator hoặc cloud platform không thể kiểm tra trạng thái sống và mức độ sẵn sàng của dịch vụ.

5. **Abrupt Shutdown Handling**  
   Ứng dụng không hỗ trợ graceful shutdown, dẫn đến nguy cơ các request đang xử lý bị gián đoạn khi service bị dừng hoặc khởi động lại.

### Exercise 1.3: Comparison Between Develop and Production Versions

| Feature | Develop Version | Production Version | Importance |
|---------|------------------|--------------------|------------|
| Config | Hardcoded values | Environment variables | Giúp tăng tính linh hoạt, bảo mật và phù hợp với nhiều môi trường triển khai khác nhau |
| Health Checks | Not implemented | `/health` and `/ready` endpoints | Hỗ trợ giám sát tự động và giúp hệ thống orchestration xác định khi nào service sẵn sàng hoạt động |
| Logging | `print()` statements | Structured JSON logging | Dễ dàng thu thập, phân tích và truy vết log trong môi trường production |
| Shutdown Behavior | Abrupt termination | Graceful shutdown using `SIGTERM` | Giảm gián đoạn dịch vụ và đảm bảo hoàn tất các request đang xử lý |

---

## Part 2: Docker Containerization

### Exercise 2.1 & 2.3: Docker Optimizations

Trong phần này, em đã phân tích và áp dụng các kỹ thuật tối ưu hóa Docker như sau:

- **Multi-stage Build**  
  Dockerfile được thiết kế theo mô hình nhiều giai đoạn. Stage đầu tiên dùng để build và cài đặt các dependencies cần thiết, trong khi stage cuối chỉ chứa các thành phần cần thiết để chạy ứng dụng. Cách tiếp cận này giúp giảm đáng kể kích thước image.

- **Image Size Optimization**  
  Sau khi áp dụng multi-stage build, kích thước image được giảm đáng kể, từ mức lớn hơn 500MB xuống khoảng 200MB. Điều này không chỉ giúp tiết kiệm tài nguyên lưu trữ mà còn tăng tốc quá trình build, push và deploy.

- **Security Improvement with Non-root User**  
  Ứng dụng được cấu hình chạy dưới user không phải `root` (ví dụ: `agent`), giúp giảm thiểu rủi ro bảo mật trong trường hợp container bị khai thác.

---

## Part 4 & Part 5: Security and Scaling (Local Verification)

### Test Results Verified on Local Docker Environment

1. **Authentication**
   - Khi gọi API mà không cung cấp API key, hệ thống trả về `401 Unauthorized`.  
   - Khi cung cấp đúng API key (`dev-key-change-me-in-production`), hệ thống xử lý request thành công và trả về kết quả từ AI agent.  

   **Kết luận:** Cơ chế xác thực hoạt động đúng như mong đợi.

2. **Rate Limiting**
   - Khi gửi số lượng request vượt quá ngưỡng quy định (trên 20 request/phút trong quá trình kiểm thử), hệ thống trả về lỗi `429 Rate limit exceeded`.

   **Kết luận:** Cơ chế giới hạn tốc độ hoạt động chính xác, giúp giảm nguy cơ lạm dụng tài nguyên hệ thống.

3. **Stateless Design**
   - Sau khi dừng và khởi động lại container, dữ liệu hội thoại vẫn được duy trì thông qua Redis.
   - Điều này chứng minh rằng ứng dụng không phụ thuộc vào bộ nhớ cục bộ của từng instance và có thể scale-out sang nhiều bản sao mà không làm mất trạng thái hội thoại.

   **Kết luận:** Thiết kế stateless đã được triển khai đúng và phù hợp với kiến trúc production.

---

## Part 6: Final Deployment and Troubleshooting Notes

Trong quá trình hoàn thiện Part 6, em đã gặp và xử lý một số vấn đề thực tế để đưa hệ thống đạt trạng thái production-ready:

1. **Fixing PYTHONPATH / Dependency Resolution**
   - Dockerfile đã được điều chỉnh để đảm bảo đường dẫn thư viện Python chính xác trong quá trình multi-stage build.
   - Việc này giúp khắc phục lỗi `ModuleNotFoundError: uvicorn` khi khởi động container.

2. **Fixing Header Removal Logic**
   - Trong quá trình triển khai middleware bảo mật, hệ thống gặp lỗi `AttributeError` khi thao tác xóa header `server`.
   - Vấn đề được xử lý bằng cách sử dụng phương thức phù hợp trên đối tượng `MutableHeaders`, từ đó bảo đảm middleware hoạt động ổn định.

3. **Environment Configuration Alignment**
   - File `.env.local` được chuẩn hóa đồng nhất với các biến môi trường sử dụng trên nền tảng Render.
   - Điều này giúp giảm sai khác giữa môi trường local và cloud, đồng thời hỗ trợ quá trình deploy ổn định hơn.

---

## Conclusion

Sau khi hoàn tất toàn bộ các phần của bài lab, hệ thống đã đáp ứng đầy đủ các yêu cầu về:

- Cấu hình theo environment variables  
- Docker multi-stage build  
- Authentication  
- Rate limiting  
- Stateless architecture với Redis  
- Production-ready deployment practices  

Đồng thời, ứng dụng đã vượt qua **100% (20/20)** tiêu chí kiểm tra của script `check_production_ready.py`, cho thấy hệ thống đã sẵn sàng để vận hành ổn định trên môi trường cloud.

---
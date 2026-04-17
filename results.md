# Day 12 Lab - Mission Answers & Final Report

**Họ và tên:** Nguyễn Công Hùng
**MSSV:** 2A202600140
---

## Part 1: Localhost vs Production (12-Factor App)

### Exercise 1.1: Anti-patterns found in Basic version
1. **Hardcoded Secrets:** API Keys nằm trực tiếp trong code, dễ bị lộ khi push lên Git.
2. **Fixed Port:** Sử dụng port 8000 cố định, không thể thay đổi theo môi trường của Cloud provider.
3. **Debug Mode:** Luôn bật debug khiến hacker có thể xem được cấu trúc code khi gặp lỗi.
4. **No Healthchecks:** Không có cơ chế báo cáo tình trạng sống còn cho Orchestrator (Docker/Cloud).
5. **Sudden Shutdown:** App bị tắt đột ngột, làm treo các request đang xử lý dở.

### Exercise 1.3: Comparison Table

| Feature | Develop | Production | Importance |
|---------|---------|------------|------------|
| Config  | Hardcoded | Environment Variables | Bảo mật và linh hoạt khi deploy đa môi trường. |
| Health  | N/A | `/health` & `/ready` | Tự động hóa việc giám sát và khôi phục dịch vụ. |
| Logging | `print()` | Structured JSON | Dễ dàng quản lý và truy vết log trên quy mô lớn. |
| Shutdown | Abrupt | Graceful (SIGTERM) | Không làm gián đoạn trải nghiệm người dùng khi cập nhật app. |

## Part 2: Docker Containerization

### Exercise 2.1 & 2.3: Docker Optimizations
- **Multi-stage Build:** Đã triển khai Dockerfile 2 giai đoạn. Stage 1 dùng để build thư viện, Stage 2 chỉ copy kết quả cuối cùng. 
- **Kết quả:** Dung lượng Image được tối ưu đáng kể (giảm từ mức >500MB xuống còn khoảng ~200MB) và loại bỏ được các lỗi bảo mật từ các công cụ build.
- **Non-root user:** Ứng dụng chạy dưới user `agent`, giảm thiểu rủi ro bị tấn công chiếm quyền root của hệ thống.

## Part 4 & 5: Security & Scaling (Local Verification)

### Test Results (Verified on Local Docker)
1. **Authentication:** 
   - Gọi API không key: Trả về `401 Unauthorized` (Thành công).
   - Gọi API đúng key (`dev-key-change-me-in-production`): Trả về kết quả AI (Thành công).
2. **Rate Limiting:** Khi gọi liên tục >20 request/phút, hệ thống tự động trả về lỗi `429 Rate limit exceeded`.
3. **Stateless Design:** Đã kiểm tra việc tắt container và khởi động lại, dữ liệu hội thoại vẫn được đồng bộ qua Redis giúp hệ thống có thể Scale-out lên nhiều bản sao mà không mất dữ liệu.

---

## Part 6: Final Deployment & Troubleshooting Notes

Trong quá trình thực hiện Part 6, em đã phát hiện và xử lý các vấn đề thực tế sau để đưa ứng dụng đạt chuẩn Production:
1. **Fix PYTHONPATH:** Điều chỉnh lại Dockerfile để đường dẫn thư viện Python chính xác trong multi-stage build, khắc phục lỗi `ModuleNotFoundError: uvicorn`.
2. **Fix Header Deletion:** Sửa lỗi `AttributeError` khi xóa Header "server" bằng cách sử dụng phương thức xóa trực tiếp trên MutableHeaders, giúp app hoàn thành middleware bảo mật.
3. **Environment Setup:** Cấu hình file `.env.local` đồng nhất với các tham số trên Cloud Render.

---
**Kết luận:** Hệ thống đã vượt qua 100% (20/20) các bài kiểm tra của script `check_production_ready.py` và đã sẵn sàng chạy ổn định trên Cloud.

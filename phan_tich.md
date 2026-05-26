# Báo cáo: Chiến lược kiểm thử cho hệ thống Booking Microservices

# Phần 1 - Phân tích logic

## 1. Thách thức kiểm thử trong kiến trúc Microservices

So với kiến trúc Monolithic, Microservices tạo ra nhiều khó khăn hơn trong kiểm thử vì hệ thống được chia thành nhiều
service độc lập.

### Các thách thức chính

### 1.1. Giao tiếp giữa các services phức tạp

Các service như:

- UserService
- BookingService
- NotificationService

giao tiếp với nhau qua REST API hoặc message queue.

Vấn đề:

- Sai format request/response.
- API thay đổi gây lỗi service khác.
- Timeout hoặc lỗi mạng.
- Dữ liệu không đồng bộ giữa các services.

Trong monolithic:

- Các module gọi trực tiếp trong cùng process.
- Ít lỗi giao tiếp hơn.

---

### 1.2. Database tách biệt

Mỗi microservice có database riêng:

- Khó kiểm tra tính nhất quán dữ liệu.
- Transaction phân tán phức tạp.
- Một service lỗi có thể ảnh hưởng toàn bộ flow nghiệp vụ.

Ví dụ:

- Booking tạo thành công.
- Nhưng Notification gửi thất bại.

---

### 1.3. Khó tái hiện môi trường thực tế

Muốn test đầy đủ cần:

- Chạy nhiều service cùng lúc.
- Có database, cache, queue.
- Cấu hình mạng giống production.

Điều này làm:

- Test phức tạp hơn.
- Tốn tài nguyên hơn monolithic.

---

### 1.4. Tốc độ release nhanh

Microservices thường deploy độc lập:

- Một service update có thể phá vỡ service khác.
- Regression bug dễ xảy ra nếu thiếu automated testing.

---

## 2. Vì sao chỉ dùng Unit Test là chưa đủ?

Unit Test chỉ kiểm tra logic bên trong từng class hoặc method riêng lẻ.

Unit Test KHÔNG kiểm tra:

- Giao tiếp REST giữa services.
- Kết nối database thực tế.
- Serialization/deserialization JSON.
- Authentication/JWT flow.
- Timeout, retry, network failure.
- Tính tương thích API giữa các service.

Ví dụ:

- BookingService unit test pass.
- Nhưng API gọi NotificationService sai field.
- Hệ thống thực tế vẫn lỗi.

Do đó:

- Unit Test giúp phát hiện lỗi logic nhỏ nhanh chóng.
- Nhưng chưa đảm bảo toàn hệ thống hoạt động đúng.

Cần kết hợp:

- Integration Test
- Contract Test
- End-to-End Test

để đảm bảo chất lượng thực tế của microservices.

---

# Phần 2 - Chiến lược kiểm thử toàn diện

# 1. Mô hình Test Pyramid cho Microservices

## Đề xuất Test Pyramid

### 70% Unit Test

### 20% Integration Test

### 10% End-to-End Test

---

## 1.1. Unit Test (70%)

Mục tiêu:

- Kiểm tra logic nghiệp vụ độc lập.
- Chạy nhanh.
- Phát hiện lỗi sớm.

Kiểm tra:

- Service logic.
- Validation.
- Utility methods.
- Mapper.
- Business rules.

Ví dụ:

- Kiểm tra logic đặt lịch.
- Kiểm tra validate thời gian booking.

Lý do chiếm nhiều nhất:

- Chi phí thấp.
- Chạy cực nhanh.
- Dễ maintain.

---

## 1.2. Integration Test (20%)

Mục tiêu:

- Kiểm tra sự tích hợp giữa:
    - Service ↔ Database
    - Service ↔ API
    - Service ↔ Queue

Kiểm tra:

- Repository.
- REST API.
- Security.
- Transaction.
- External communication.

Ví dụ:

- BookingService gọi UserService.
- NotificationService gửi email.

Lý do quan trọng:

- Phần lớn bug microservices xuất hiện ở tầng integration.

---

## 1.3. End-to-End Test (10%)

Mục tiêu:

- Kiểm tra toàn bộ flow nghiệp vụ thực tế.

Ví dụ flow:

1. User đăng nhập.
2. Tạo booking.
3. Booking lưu DB.
4. Notification gửi email.

Đặc điểm:

- Chậm.
- Tốn tài nguyên.
- Khó maintain.

Do đó:

- Chỉ test các business flow quan trọng nhất.

---

# 2. Công cụ và kỹ thuật kiểm thử

## 2.1. Unit Test

### Công cụ

- JUnit 5
- Mockito
- AssertJ

### Kỹ thuật

- Mock dependencies.
- Test độc lập từng class.
- Kiểm tra branch coverage.

Ví dụ:

- Mock UserRepository.
- Test BookingService.

---

## 2.2. Integration Test

### Công cụ

- Spring Boot Test
- Testcontainers
- @DataJpaTest
- @WebMvcTest
- REST Assured

### Kỹ thuật

- Chạy database thật bằng Docker.
- Test API thực tế.
- Kiểm tra transaction và persistence.

Ví dụ:

- Test Booking API với PostgreSQL container.
- Test JWT authentication.

---

## 2.3. End-to-End Test

### Công cụ

- Cypress
- Selenium

### Kỹ thuật

- Test theo user workflow.
- Kiểm tra UI + backend flow.

Ví dụ:

- User đặt lịch hoàn chỉnh.
- Kiểm tra email notification.

---

# 3. Chiến lược Testing Coverage với JaCoCo

## Mục tiêu coverage

| Loại            | Mức tối thiểu |
|-----------------|---------------|
| Line Coverage   | 80%           |
| Branch Coverage | 70%           |

---

## Cách áp dụng

Mỗi microservice:

- Tích hợp JaCoCo trong Maven/Gradle.
- Sinh report tự động khi build.
- Fail build nếu coverage thấp hơn quy định.

Ví dụ:

- BookingService coverage < 80% → pipeline fail.

---

## Vì sao cần Quality Gates?

Quality Gates giúp:

- Ngăn code chưa test đầy đủ được merge.
- Duy trì chất lượng lâu dài.
- Giảm technical debt.
- Tăng độ tin cậy khi release nhanh.

Branch Coverage đặc biệt quan trọng vì:

- Giúp kiểm tra đầy đủ các nhánh logic.
- Giảm bug nghiệp vụ tiềm ẩn.

---

# 4. Kiểm thử giao tiếp giữa các Services

## 4.1. Integration Test với Testcontainers

Sử dụng:

- PostgreSQL container
- Redis container
- Kafka container

Mục tiêu:

- Test gần giống production.
- Kiểm tra API thực tế.

Ví dụ:

- BookingService gọi UserService thật.

---

## 4.2. Consumer-Driven Contract Testing (CDC)

### Công cụ

- Pact

### Mục tiêu

Đảm bảo:

- API provider không phá vỡ consumer.

Ví dụ:

- NotificationService thay đổi JSON response.
- Pact sẽ phát hiện BookingService không còn tương thích.

Lợi ích:

- Giảm lỗi tích hợp.
- Hạn chế regression khi deploy độc lập.

---

## 4.3. End-to-End Testing

Kiểm tra toàn bộ flow:

- Login
- Booking
- Payment
- Notification

Mục tiêu:

- Đảm bảo hệ thống hoạt động đúng từ góc nhìn người dùng.

---

# 5. Tích hợp kiểm thử vào CI/CD

## Quy trình CI/CD đề xuất

### Bước 1 - Pull Request

Tự động chạy:

- Unit Test
- Static Analysis
- JaCoCo Coverage

Nếu fail:

- Không cho merge.

---

### Bước 2 - Build Pipeline

Chạy:

- Integration Test
- Contract Test

Kiểm tra:

- Database
- API communication
- Service compatibility

---

### Bước 3 - Staging

Chạy:

- End-to-End Test
- Smoke Test

Kiểm tra:

- Business flow chính.

---

### Bước 4 - Production

Triển khai:

- Canary deployment hoặc Blue-Green deployment.
- Monitoring + logging.

---

# 6. Kết luận

Đối với hệ thống Booking Microservices:

- Chỉ sử dụng Unit Test là chưa đủ.
- Cần kết hợp nhiều tầng kiểm thử để đảm bảo chất lượng.

Chiến lược hiệu quả cần:

- Test Pyramid hợp lý.
- Automation tối đa.
- Coverage Quality Gates.
- Integration Test và Contract Test mạnh mẽ.
- Tích hợp chặt chẽ với CI/CD.

Nhờ đó hệ thống sẽ:

- Giảm bug production.
- Tăng tốc độ release.
- Dễ mở rộng và maintain lâu dài.
# TẬP HỢP USER STORIES VÀ LUỒNG TRẢI NGHIỆM NGƯỜI DÙNG (IRONFITNESS / HOMEFIT)

Tài liệu này tổng hợp danh sách các **User Stories (Chuyện của người dùng)** cho ứng dụng luyện tập thể thao **IronFitness (HomeFit)**. Tất cả các User Stories, quy trình thực hiện và kịch bản kiểm thử (Acceptance Criteria) đều được trực quan hóa bằng **Biểu đồ Mermaid** thay vì bảng biểu truyền thống.

---

## 🗺️ Tổng quan Hệ thống User Stories (Epics Overview)

```mermaid
mindmap
  root((IronFitness App))
    Epic 1: Xác thực & Tài khoản
      US01: Khởi động & Tự động Đăng nhập
      US02: Đăng ký & Đăng nhập Email/Google
      US03: Quản lý Hồ sơ & BMI
    Epic 2: Kế hoạch & Khám phá
      US04: Tập theo Nhóm cơ
      US05: Tập theo Mục tiêu
      US06: Lọc bài tập theo Thời lượng
    Epic 3: Thực hiện Buổi tập
      US07: Chuẩn bị & Kỹ thuật
      US08: Đếm giờ & Player Controls
      US09: Nghỉ & Xem Tổng kết
    Epic 4: Lịch sử & Đồng bộ
      US10: Xem Lịch tập & Nhật ký ngày
      US11: Xem Thống kê Tiến độ
      US12: Lưu Offline & Đồng bộ Cloud
    Epic 5: Cài đặt & Trải nghiệm
      US13: Đặt Lịch Nhắc nhở Tập luyện
      US14: Nâng cấp Premium & Ẩn Ads
```

---

## 📌 Epic 1: Xác thực & Quản lý Tài khoản (Authentication & Profile)

### 🔹 US01: Khởi động & Tự động Đăng nhập (Auto Login)
**Câu chuyện người dùng:**
> *"Là một người tập luyện thường xuyên, tôi muốn ứng dụng tự động nhận diện phiên đăng nhập khi mở app để tôi có thể vào ngay màn hình tập luyện mà không phải đăng nhập lại mỗi lần."*

```mermaid
flowchart TD
    subgraph US01 ["US01: Khởi động & Tự động Đăng nhập"]
        A["👤 Người dùng: Mở ứng dụng IronFitness"] --> B["System: SplashFragment kích hoạt"]
        B --> C{"TokenStore: Kiểm tra JWT Token?"}
        C -- "Token hợp lệ & Chưa hết hạn" --> D["System: Chuyển thẳng đến HomeWorkoutFragment"]
        C -- "Chưa có Token / Hết hạn" --> E["System: Chuyển đến LoginFragment"]
        
        D --> F["✅ Acceptance Criteria: Người dùng vào ngay trang chính trong < 2 giây"]
        E --> G["✅ Acceptance Criteria: Hiển thị thông báo Hết phiên đăng nhập nếu token hết hạn"]
    end
```

---

### 🔹 US02: Đăng ký & Đăng nhập Tài khoản (Email / Google Sign-In)
**Câu chuyện người dùng:**
> *"Là một người dùng mới, tôi muốn đăng ký/đăng nhập tài khoản bằng Email hoặc Google để lưu trữ tiến trình tập luyện cá nhân an toàn."*

```mermaid
flowchart TD
    subgraph US02 ["US02: Đăng ký & Đăng nhập"]
        Start["👤 Người dùng tại LoginFragment"] --> Choice{"Lựa chọn phương thức đăng nhập?"}
        
        Choice -- "Email & Password" --> Form["Nhập Email & Mật khẩu"]
        Form --> Valid{"Validate dữ liệu?"}
        Valid -- "Lỗi (Email sai định dạng / Mật khẩu ngắn)" --> ErrMsg["Hiển thị lỗi tại TextInputLayout"]
        Valid -- "Hợp lệ" --> ApiCall["Gọi API /auth/login"]
        
        Choice -- "Google Sign-In" --> GBtn["Nhấn Đăng nhập với Google"]
        GBtn --> GAuth["Xác thực qua Google SDK lấy ID Token"]
        GAuth --> GApiCall["Gọi API /auth/google"]
        
        ApiCall --> Response{"Kết quả Server?"}
        GApiCall --> Response
        
        Response -- "Thành công" --> SaveSession["Lưu Access & Refresh Token vào TokenStore"]
        SaveSession --> GoHome["Điều hướng vào HomeWorkoutFragment"]
        
        Response -- "Thất bại (Sai mk / Lỗi mạng)" --> Snackbar["Hiển thị Snackbar thông báo lỗi"]
    end
```

---

### 🔹 US03: Quản lý Hồ sơ Cá nhân & Theo dõi BMI
**Câu chuyện người dùng:**
> *"Là một người dùng, tôi muốn cập nhật Chiều cao và Cân nặng để ứng dụng tự động tính chỉ số BMI và phân loại thể trạng cho tôi."*

```mermaid
flowchart LR
    subgraph US03 ["US03: Quản lý Hồ sơ & Tính BMI"]
        Input1["👤 Nhập Chiều cao (cm)"] --> Calc["BmiCalculator: Tính BMI = W / (H * H)"]
        Input2["👤 Nhập Cân nặng (kg)"] --> Calc
        
        Calc --> Classify{"Phân loại thể trạng BMI"}
        Classify -- "< 18.5" --> UW["Gầy (Underweight) - Màu Vàng"]
        Classify -- "18.5 - 24.9" --> N["Bình thường (Normal) - Màu Xanh lá"]
        Classify -- "25.0 - 29.9" --> OW["Thừa cân (Overweight) - Màu Cam"]
        Classify -- ">= 30.0" --> OB["Béo phì (Obese) - Màu Đỏ"]
        
        UW & N & OW & OB --> RenderUI["Cập nhật Card BMI trên giao diện Profile"]
        RenderUI --> SaveBtn["👤 Nhấn nút 'Lưu thay đổi'"]
        SaveBtn --> RoomDB["Lưu vào Room DB (UserProfileEntity) & PUT /me/profile"]
    end
```

---

## 📌 Epic 2: Khám phá & Kế hoạch Tập luyện (Workout Plans)

### 🔹 US04: Chọn Bài tập theo Nhóm cơ Mục tiêu
**Câu chuyện người dùng:**
> *"Là một người tập, tôi muốn chọn nhanh bài tập theo từng nhóm cơ (Bụng, Tay, Ngực, Chân, Toàn thân) để tập trung phát triển nhóm cơ mong muốn."*

```mermaid
flowchart TD
    subgraph US04 ["US04: Tập theo Nhóm cơ"]
        A["👤 Người dùng ở HomeWorkoutFragment"] --> B["Bấm chọn Chip nhóm cơ (Vd: Bụng / Ngực / Tay)"]
        B --> C["HomeWorkoutViewModel: Lọc bài tập theo GroupKey"]
        C --> D["Hiển thị danh sách nhóm bài tập phù hợp trên RecyclerView"]
        D --> E["👤 Chọn 1 nhóm bài tập (Vd: 'Cơ bụng nâng cao')"]
        E --> F["Chuyển sang ExerciseListFragment hiển thị danh sách các động tác"]
    end
```

---

### 🔹 US05: Tập luyện theo Mục tiêu Phổ biến (Build Muscle / Fat Loss / Keep Fit)
**Câu chuyện người dùng:**
> *"Là một người cần định hướng tập luyện, tôi muốn ứng dụng đề xuất các lộ trình bài tập sẵn có theo mục tiêu Tăng cơ, Đốt mỡ hoặc Giữ dáng với cấp độ từ Dễ đến Khó."*

```mermaid
flowchart TD
    subgraph US05 ["US05: Tập theo Mục tiêu Phổ biến"]
        A["👤 Chọn Tab Mục tiêu (Tăng cơ / Đốt mỡ / Giữ dáng)"] --> B["PopularWorkoutGenerator: Sinh dữ liệu lộ trình tự động"]
        B --> C{"Chọn Cấp độ khó"}
        C -- "Easy (Dễ)" --> L1["Gói 11 - 15 phút (Mạch 2 vòng)"]
        C -- "Medium (Trung bình)" --> L2["Gói 20 - 22 phút (Mạch 3 vòng)"]
        C -- "Hard (Khó)" --> L3["Gói 25 - 28 phút (Cường độ cao)"]
        
        L1 & L2 & L3 --> D["Hiển thị cấu trúc bài tập: Warm-up -> Main -> Cool-down"]
    end
```

---

### 🔹 US06: Lọc Bài tập theo Thời lượng & Nhu cầu
**Câu chuyện người dùng:**
> *"Là một người bận rộn, tôi muốn lọc bài tập theo quỹ thời gian của mình (<7 phút, 7-15 phút, >15 phút) để có thể tập luyện ngay cả khi có ít thời gian rảnh."*

```mermaid
flowchart LR
    subgraph US06 ["US06: Lọc bài tập theo thời lượng"]
        User["👤 Người dùng nhấn nút Filter"] --> FilterOption{"Chọn mốc thời lượng?"}
        
        FilterOption -- "< 7 phút" --> Quick["Bài tập siêu ngắn / Tabata 6 phút"]
        FilterOption -- "7 - 15 phút" --> Medium["Bài tập vừa phải / Khởi động & Giãn cơ"]
        FilterOption -- "> 15 phút" --> Long["Chương trình tập toàn thân chuyên sâu"]
        
        Quick & Medium & Long --> Display["FilteredGroupsFragment: Hiển thị các nhóm bài tập thỏa mãn"]
    end
```

---

## 📌 Epic 3: Thực hiện & Điều khiển Buổi tập (Workout Player & Timer)

### 🔹 US07 & US08 & US09: Luồng Trình phát Bài tập Chi tiết (Player Sequence)
**Câu chuyện người dùng:**
> *"Là một người đang tập luyện, tôi muốn ứng dụng đếm ngược thời gian chuẩn bị, hiển thị hướng dẫn động tác, tự động chuyển bài và đếm thời gian nghỉ giữa các bài để tôi tập luyện liên tục mà không cần chạm vào điện thoại nhiều lần."*

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 Người tập
    participant PlayerUI as 📱 PlayerFragment
    participant VM as ⚙️ PlayerViewModel
    participant DB as 💾 Room DB / Sync

    User->>PlayerUI: Nhấn "Bắt đầu tập" từ ExerciseListFragment
    PlayerUI->>VM: loadExercises(groupKey, difficulty)
    VM->>PlayerUI: Cập nhật State = PREPARING (Đếm ngược 5 giây)
    Note over PlayerUI: Màn hình hiện "Chuẩn bị", đếm 5, 4, 3, 2, 1

    loop Cho từng bài tập trong danh sách
        VM->>PlayerUI: Cập nhật State = EXERCISING (Bài tập N)
        Note over PlayerUI: Hiện tên bài tập, hướng dẫn, bộ đếm giờ (hoặc số reps)
        
        alt Bài tập đếm giờ (30s)
            VM->>PlayerUI: Tự động giảm remainingSeconds mỗi giây
        else Bài tập theo Reps (12 lần)
            User->>PlayerUI: Nhấn nút "Đã xong"
        end

        alt Chưa phải bài tập cuối
            VM->>PlayerUI: Cập nhật State = RESTING (Đếm ngược nghỉ 15 giây)
            Note over PlayerUI: Hiện "Thời gian nghỉ" & xem trước Bài tiếp theo
        end
    end

    VM->>PlayerUI: Cập nhật State = FINISHED
    PlayerUI->>User: Điều hướng tới WorkoutSummaryFragment (Tổng kết: Số bài, Thời gian, Calo)
    VM->>DB: saveWorkout() -> Ghi nhận log vào Room DB & Tự động gọi syncPending()
```

---

## 📌 Epic 4: Lịch sử & Đồng bộ Dữ liệu (History & Cloud Sync)

### 🔹 US10 & US11: Lịch tập luyện & Thống kê Tiến độ
**Câu chuyện người dùng:**
> *"Là một người theo dõi sức khỏe, tôi muốn xem lại lịch sử các ngày đã tập trong tháng qua giao diện Lịch và xem tổng số buổi tập cũng như tổng thời gian đã tích lũy."*

```mermaid
flowchart TD
    subgraph US10_11 ["US10 & US11: Lịch tập luyện & Thống kê"]
        A["👤 Mở Tab Lịch sử (reportFragment)"] --> B["HistoryViewModel: Truy vấn Room DB"]
        
        B --> C["Lấy danh sách các mốc thời gian đã tập trong tháng"]
        C --> D["HistoryAdapter: Vẽ Lịch 42 ô & Đánh dấu chấm màu tại ngày có tập"]
        
        B --> E["Truy vấn SUM(totalSeconds) và COUNT(*) từ WorkoutLogDao"]
        E --> F["Hiển thị Thống kê: Tổng số buổi tập & Tổng số phút tích lũy"]
        
        D -->|👤 Bấm chọn 1 ngày trên Lịch| G["Lọc danh sách các buổi tập hoàn thành trong ngày đó"]
        G --> H["Hiển thị WorkoutLogAdapter bên dưới Lịch"]
    end
```

---

### 🔹 US12: Lưu trữ Offline-First & Đồng bộ Cloud (Cloud Sync)
**Câu chuyện người dùng:**
> *"Là một người dùng di động, tôi muốn ứng dụng vẫn lưu lại nhật ký tập luyện ngay cả khi không có mạng (Offline), và tự động đồng bộ lên Server khi có mạng trở lại mà không làm mất dữ liệu."*

```mermaid
flowchart TD
    subgraph US12 ["US12: Lưu trữ Offline-First & Đồng bộ Cloud"]
        WorkoutDone["🏋️ Hoàn thành buổi tập"] --> GenUUID["Tạo clientUuid ngẫu nhiên (UUID.randomUUID)"]
        GenUUID --> SaveRoom["Ghi nhật ký vào Room DB với synced = false"]
        
        SaveRoom --> CheckNet{"Kiểm tra kết nối Mạng & Token?"}
        
        CheckNet -- "Mất mạng / Chưa đăng nhập" --> StayOffline["Giữ nguyên synced = false. Dữ liệu an toàn ở máy"]
        
        CheckNet -- "Có mạng & Đã đăng nhập" --> SyncBatch["Gọi API /sessions/batch gửi danh sách các log chưa sync"]
        
        SyncBatch --> ServerResp{"Server phản hồi 200 OK?"}
        ServerResp -- "Thành công" --> MarkSynced["Cập nhật Room DB: markSynced(syncedUuids) -> synced = true"]
        ServerResp -- "Lỗi mạng / Server bận" --> RetryLater["Giữ synced = false. Tự động thử lại ở lần mở app sau"]
    end
```

---

## 📌 Epic 5: Cài đặt Nhắc nhở & Trải nghiệm (Settings & Ads)

### 🔹 US13: Lên lịch Nhắc nhở Tập luyện Hằng ngày
**Câu chuyện người dùng:**
> *"Là một người cần tính kỷ luật, tôi muốn cài đặt giờ nhắc nhở tập luyện hằng ngày để ứng dụng gửi thông báo đúng giờ giúp tôi duy trì thói quen."*

```mermaid
flowchart TD
    subgraph US13 ["US13: Đặt Lịch Nhắc nhở Tập luyện"]
        A["👤 Vào SettingsFragment -> Bật Switch Nhắc nhở"] --> B{"Kiểm tra Android Version & Quyền?"}
        
        B -- "Android 13+ & Chưa cấp quyền" --> C["Hiển thị hộp thoại xin quyền POST_NOTIFICATIONS"]
        C -- "Từ chối" --> D["Tự động tắt Switch & Hiển thị thông báo Từ chối"]
        C -- "Cho phép" --> E["Mở MaterialTimePicker cho người dùng chọn Giờ:Phút"]
        
        B -- "Đã có quyền / Android < 13" --> E
        
        E --> F["Lưu Giờ/Phút vào SettingsStore"]
        F --> G["ReminderScheduler: Đăng ký Báo thức với AlarmManager"]
        
        G --> H["⏰ Đúng giờ hẹn: ReminderReceiver nhận tín hiệu & Bắn Notification nhắc tập"]
        
        I["📱 Thiết bị khởi động lại (Reboot)"] --> J["BootReceiver nhận sự kiện BOOT_COMPLETED"]
        J --> K["Đọc SettingsStore & Đăng ký lại AlarmManager tự động"]
    end
```

---

### 🔹 US14: Nâng cấp Premium & Trải nghiệm Không Quảng cáo
**Câu chuyện người dùng:**
> *"Là một người dùng Premium, tôi muốn ứng dụng tự động ẩn toàn bộ quảng cáo banner để có trải nghiệm tập luyện tập trung và mượt mà nhất."*

```mermaid
flowchart LR
    subgraph US14 ["US14: Kiểm tra Cờ Premium & Quản lý Quảng cáo"]
        ProfileObs["ProfileViewModel: Theo dõi UserProfileEntity từ Room DB"] --> CheckPrem{"isPremium == true?"}
        
        CheckPrem -- "Đúng (Tài khoản Premium)" --> DisableAds["AdsManager.setAdsEnabled(false)"]
        DisableAds --> HideBanner["BannerAdHelper: Ẩn adContainer & Không tải Ads"]
        
        CheckPrem -- "Sai (Tài khoản Thường)" --> EnableAds["AdsManager.setAdsEnabled(true)"]
        EnableAds --> LoadBanner["BannerAdHelper: Tải AdMob Banner Ad thử nghiệm"]
    end
end
```

---

## 🧪 Quy trình Kiểm thử & Xử lý Ngoại lệ (Edge Cases & Exception Handling)

```mermaid
flowchart TD
    subgraph ExceptionHandling ["Xử lý Ngoại lệ Hệ thống"]
        E1["Hết hạn Access Token (401)"] --> R1["TokenAuthenticator gọi Refresh Token -> Nếu thất bại: Điều hướng về LoginFragment"]
        E2["Mất mạng khi Đăng ký/Đăng nhập"] --> R2["Hiển thị Snackbar báo lỗi mạng, giữ nguyên thông tin form"]
        E3["Mất mạng khi lưu Buổi tập"] --> R3["Lưu offline vào Room DB, chờ có mạng tự đồng bộ lại"]
        E4["Thiếu chiều cao/cân nặng"] --> R4["Ẩn thẻ BMI trên Profile, không báo lỗi app"]
    end
```

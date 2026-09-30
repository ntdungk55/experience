# Giải phóng dung lượng ổ C trên Windows

## Ngày

2026-08-18

## Tóm tắt

Khi ổ C gần đầy, cần đo dung lượng trước và ưu tiên cache hoặc thành phần không còn sử dụng. Không xóa thủ công các thư mục hệ thống, `Program Files`, hoặc `C:\ProgramData\Package Cache`.

## Kiểm tra nhanh

```powershell
Get-Volume -DriveLetter C
```

Chú ý cả dung lượng trống (GB) và phần trăm trống. Ổ C chỉ còn vài GB trống có thể làm Windows Update hoặc bộ cài MSI thất bại với lỗi `1603` / `OutOfDiskSpace`.

## Các vị trí thường có thể dọn

| Vị trí | Cách xử lý | Lưu ý |
|---|---|---|
| npm cache | `npm cache clean --force` | Sẽ được tải lại khi cài package. |
| pip cache | `python -m pip cache purge` | Sẽ được tải lại khi cài package Python. |
| `%temp%` | Mở bằng `Win + R`, chọn tất cả và xóa; bỏ qua file đang dùng. | Chỉ xóa nội dung, không xóa thư mục Temp. |
| Cache Chrome/Edge | Đóng trình duyệt, dùng `Ctrl + Shift + Delete` để xóa cache. | Không xóa toàn bộ profile nếu chưa sao lưu dữ liệu. |
| Windows Update/Temporary files | Settings > System > Storage > Temporary files hoặc Disk Cleanup. | Dùng công cụ Windows thay vì xóa trực tiếp thư mục hệ thống. |

## Android Studio: nguồn chiếm dung lượng lớn

Android SDK System Image khác với Android Virtual Device (AVD):

```text
System Image = bộ hệ điều hành đã tải cho emulator
AVD          = máy ảo được tạo từ một System Image
```

Nếu Device Manager không có emulator, System Images vẫn có thể còn trong SDK và chiếm nhiều GB. Có thể xóa image không dùng bằng:

```text
Android Studio > Tools > SDK Manager > SDK Platforms > Show Package Details
```

Bỏ chọn các System Image không cần rồi chọn `Apply`. Việc này không ảnh hưởng việc viết code, build APK/AAB, hoặc chạy ứng dụng trên điện thoại thật; chỉ cần tải lại nếu sau này muốn tạo emulator với image đó.

Nếu không dùng emulator, có thể bỏ thêm `Android Emulator` trong tab `SDK Tools`.

## Gradle và cache Android Studio

- `C:\Users\<user>\.gradle\caches`: có thể xóa sau khi đóng Android Studio. Gradle sẽ tải lại dependencies khi build lần sau.
- Android Studio: dùng `File > Invalidate Caches... > Invalidate and Restart` để dọn index/cache an toàn. IDE sẽ index lại project sau khi khởi động.
- Không xóa bừa thư mục SDK hoặc thư mục cấu hình Android Studio nếu vẫn sử dụng công cụ này.

## Ví dụ thực tế

Trên máy đã kiểm tra:

- Android SDK System Images chiếm 9.48 GB (Android 34: 7.20 GB; Android 36: 2.28 GB).
- Gradle cache chiếm 6.18 GB.
- Android Studio index/cache/log chiếm khoảng 3.86 GB.
- Dọn npm và pip cache đã giải phóng 1.82 GB.

Kết luận: khi thiếu dung lượng, hãy ưu tiên xóa System Images không dùng, Gradle cache và cache IDE trước; đây thường là các mục giải phóng được nhiều nhất mà không ảnh hưởng mã nguồn.

# Thủ phạm chính làm đầy ổ C

- `AppData`: thường chứa dữ liệu ứng dụng, cache và extension; cần bóc tách trước khi xóa.
- IDE/dev tools: VS Code, Kiro, Cursor, Android Studio thường phình do extension, index, log và cache.
- Android SDK System Images: có thể chiếm nhiều GB; chỉ giữ image cần cho emulator.
- Gradle/npm/pip/uv/temp/cache: có thể dọn; công cụ sẽ tải lại khi cần.
- `pagefile.sys`, `Windows`, `Program Files`: không xóa thủ công.
- Khi dọn: đóng ứng dụng, xóa cache trước; giữ thư mục `User` nếu còn cần cấu hình/lịch sử.

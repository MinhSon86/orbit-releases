# Cài Orbit trên máy Windows khác

Yêu cầu: Windows 10/11 64-bit. Bộ cài đã có Python và giao diện; không cần Python, Node.js, Git hoặc quyền quản trị.

1. Vào https://github.com/MinhSon86/orbit-releases/releases/latest và tải tệp `Orbit-...-windows.zip`.
2. Giải nén toàn bộ ZIP, mở thư mục `Orbit`, chạy `Install.cmd`. Nếu Windows hỏi xác nhận nguồn tải, kiểm tra đúng địa chỉ GitHub trên và SHA-256 đi kèm trước khi tiếp tục.
3. Mặc định app được cài tại `%LOCALAPPDATA%\Orbit`, tạo lối tắt Orbit trên Desktop và mở trình duyệt.
4. Cài và đăng nhập Codex CLI hoặc Antigravity CLI trên máy mới. Mở Orbit → Cài đặt để kiểm tra CLI và IDE. Chọn các thư mục dự án của máy mới.

Cài vào đường dẫn khác: mở PowerShell trong thư mục bộ cài và chạy:

```powershell
.\scripts\install.ps1 -Destination 'D:\Apps\Orbit'
```

Nếu máy đã chạy Orbit từ mã nguồn trên cổng 8765, dừng runner đó trước khi mở bản cài, hoặc dùng:

```powershell
& "$env:LOCALAPPDATA\Orbit\scripts\launcher.ps1" -Port 8766
```

## Nhận cập nhật

App kiểm tra bản phát hành công khai khi khởi động và mỗi 6 giờ. Thông báo bản mới xuất hiện trong app; vào Cài đặt → Phiên bản & cập nhật → Tải & cập nhật. Không cần tài khoản GitHub hoặc quyền truy cập mã nguồn.

Cập nhật chỉ bắt đầu khi không có tác vụ AI đang chờ/chạy. App kiểm tra SHA-256, giải nén vào thư mục phiên bản riêng, sao lưu SQLite, chuyển phiên bản và khởi động lại. Khi bản mới không vượt qua kiểm tra khởi động, app khôi phục bản cũ và SQLite. Bộ cài và bản trước được giữ lại để chẩn đoán; chưa tự dọn các phiên bản cũ.

Dữ liệu nằm tại thư mục cài đặt `\.data`, tách khỏi `\versions`. Cập nhật không ghi đè thư mục dự án. Sao lưu thủ công: dừng Orbit khi không còn tác vụ rồi sao chép toàn bộ `.data`. Máy mới có dữ liệu riêng; bộ cài không tự chuyển dữ liệu từ máy cũ.

Bản 0.4.0 và các bản trước chưa có trình cập nhật này: cần cài một lần bằng bộ cài mới. Khi chuyển từ mã nguồn, giữ nguyên dữ liệu cũ và chỉ sao chép `.data` sang thư mục cài khi cả hai runner đã dừng. Kiểm tra lại đường dẫn dự án nếu đổi máy.

Để dừng: chạy `Stop-Orbit.cmd` trong thư mục cài khi không còn tác vụ. Để gỡ, dừng Orbit rồi xóa thư mục cài và lối tắt. Sao lưu dữ liệu trước khi xóa.

## Phát hành mã mới

Mã nguồn riêng tư: https://github.com/MinhSon86/ai-orchestrator

Bộ cài công khai: https://github.com/MinhSon86/orbit-releases

Push lên `master` sẽ chạy kiểm thử, dựng giao diện, đóng gói rồi phát hành sang repository công khai khi tất cả bước thành công. Push nhánh khác hoặc pull request chỉ kiểm tra và tạo artifact. Mỗi build có định danh thời gian UTC riêng; không cần sửa VERSION cho từng thay đổi nhỏ. Sửa VERSION khi đổi phiên bản sản phẩm. Nếu build lỗi, app tiếp tục dùng bản phát hành trước.

Workflow cần secret `ORBIT_RELEASE_TOKEN` ở repository mã nguồn, có quyền Contents write cho repository phát hành. Ưu tiên token chỉ cấp quyền repository phát hành; thay thế/gia hạn secret khi token hết hạn. Không đặt token trong mã nguồn, biến VITE hoặc bộ cài. Bản phát hành chứa bytecode Python và giao diện đã dựng; bytecode có thể được phân tích ngược, nên repository riêng tư không bảo đảm bí mật thuật toán trong bộ cài công khai.


## Bộ phát hành tự động trên máy phát triển

GitHub Actions hiện trả `startup_failure` trước khi tạo job; chưa xác minh được nguyên nhân từ API. Đã bổ sung bộ phát hành cục bộ để không phụ thuộc việc này.

Tác vụ Windows **Orbit Release Publisher** kiểm tra `origin/master` mỗi 5 phút khi tài khoản Windows đã đăng nhập và máy có mạng. Khi có commit mới, nó tạo worktree tách biệt, cài dependency vào môi trường riêng, chạy kiểm thử, build, kiểm chứng cài/cập nhật/rollback rồi phát hành công khai. Nó không pull/reset cây mã nguồn đang làm việc, không cần lưu thêm token: chỉ lấy thông tin đăng nhập từ Git Credential Manager trong bộ nhớ lúc phát hành. Máy khách không có tác vụ phát hành này.

Kích hoạt trên máy phát triển khác: chạy `scripts/enable-publisher.ps1` sau khi clone mã nguồn, setup môi trường và đăng nhập GitHub có quyền phát hành. Xem log tại `.data/publisher/publisher.log`; chạy ngay bằng `scripts/publish-watch.ps1`. Tạm dừng bằng `Disable-ScheduledTask -TaskName 'Orbit Release Publisher'`. Có thể gỡ bằng `Unregister-ScheduledTask -TaskName 'Orbit Release Publisher'`.

Nếu máy phát triển tắt hoặc đăng xuất, bản phát hành tự động cục bộ sẽ chờ đến khi máy hoạt động lại. Khi GitHub Actions hoạt động ổn định, có thể tắt bộ cục bộ. Chỉ một bộ phát hành nên chạy cho một commit để tránh tạo hai bản tương đương.

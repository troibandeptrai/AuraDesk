# AuraDesk — Master Implementation Plan & Execution Log

> **Tài liệu điều hành dự án duy nhất (living document)**  
> Version: **1.0**  
> Ngày khởi tạo: **2026-11-17**  
> Trạng thái: **PLANNING APPROVED — M0 NOT STARTED**  
> Product name: **AuraDesk** (working name; có thể đổi sau)  
> Target release: **Windows 11 desktop application + Android companion**

---

## 0. Mục đích và quy tắc sử dụng tài liệu

Tài liệu này là **nguồn sự thật vận hành** cho kế hoạch, thứ tự triển khai, tiêu chí nghiệm thu, các quyết định và tiến độ thực tế của AuraDesk. Khi tiếp tục dự án ở một phiên làm việc mới, phải đọc tài liệu này trước khi đề xuất hoặc thay đổi code.

### 0.1. Cách cập nhật

- Đây là một file Markdown duy nhất được duy trì xuyên suốt: `AURADESK_IMPLEMENTATION_MASTER.md`.
- Sau mỗi phiên triển khai có thay đổi đáng kể, cập nhật mục **Current Project Status**, **Milestone Status**, **Execution Log** và **Next Actions**.
- Sau khi hoàn thành một batch, ghi rõ: branch, commit, file thay đổi, lệnh build/test, kết quả thực tế, lỗi còn tồn tại và quyết định tiếp theo.
- Sau khi hoàn tất một milestone, cập nhật tiêu chí nghiệm thu và trạng thái milestone. Chỉ chuyển sang `DONE` khi có bằng chứng kiểm chứng.
- Nếu có quyết định kiến trúc mới, ghi vào **Decision Log** trong file này trước hoặc cùng batch triển khai.
- Nếu thay đổi làm ảnh hưởng phạm vi, kiến trúc, giao thức hoặc tiêu chí nghiệm thu, cập nhật các phần tương ứng và tăng version tài liệu.
- Không đánh dấu một tính năng đã hoàn thành chỉ vì code đã được viết. Phải có test hoặc xác minh thủ công phù hợp.

### 0.2. Quy tắc về sự thật và bằng chứng

Mỗi mục tiến độ phải phân biệt:

- **Verified:** đã chạy hoặc kiểm tra và có bằng chứng.
- **Implemented, not verified:** đã có code nhưng chưa kiểm chứng đầy đủ.
- **Proposed:** phương án dự kiến, chưa thực hiện.
- **Blocked:** chưa thể triển khai do phụ thuộc hoặc giới hạn kỹ thuật.
- **Deferred:** cố ý chuyển sang giai đoạn sau.
- **Rejected:** đã thử/đánh giá và quyết định không dùng.

Không được biến giả định thành sự thật. Ghi rõ phiên bản Windows, thiết bị Android, ứng dụng nguồn, SDK và cấu hình được dùng trong mỗi kết quả tương thích.

### 0.3. Quy tắc về Git và tài liệu

- Repository GitHub riêng của AuraDesk dùng để lưu **source code, tests, build scripts, cấu hình dự án, workflow CI và lịch sử commit**.
- Theo quyết định hiện tại, không cần đưa toàn bộ PRD/TAS Markdown hoặc bộ tài liệu thiết kế lên Git. File master này là ngoại lệ có chủ đích vì người dùng yêu cầu dùng nó làm nhật ký và nguồn trạng thái dự án.
- PRD v0.2 và Technical Architecture Specification v0.2 hiện là tài liệu tham chiếu bên ngoài. Nếu chúng được cập nhật, ghi phiên bản mới và những thay đổi tương ứng vào file này.
- Không đưa secrets, access tokens, private keys, notification content, dữ liệu cá nhân hoặc audio samples vào Git.
- Không coi ChatGPT Memory là bản sao thay thế Git hoặc file master. File master + Git mới là nguồn trạng thái có thể đối chiếu.

---

# 1. Tầm nhìn sản phẩm

AuraDesk là một ambient desktop experience cho Windows 11. Giao diện desktop thay đổi theo bài nhạc đang phát, có hiệu ứng ánh sáng phản ứng với âm thanh và một tiện ích thông báo nhỏ nằm ở góc màn hình. Android companion chuyển tiếp notification từ điện thoại sang Windows và hỗ trợ Quick Reply khi notification nguồn cung cấp action tương thích.

## 1.1. Trải nghiệm cốt lõi

1. Artwork bài hát là tâm điểm thị giác.
2. Background, gradient, shadow, glow và một số thành phần giao diện lấy palette từ artwork.
3. Bốn cạnh/ góc viền màn hình có dải sáng phản ứng với âm thanh đang phát.
4. Widget nhạc hiển thị metadata và điều khiển playback khi nguồn cho phép.
5. Notification xuất hiện dạng widget/card nhỏ ở một góc, cùng ambient surface; không chiếm vùng trung tâm và không tự phóng to.
6. Người dùng có thể mở rộng notification để xem thêm nội dung và trả lời khi nguồn hỗ trợ.
7. Windows notification được nhận trực tiếp từ Windows; Android notification được chuyển tiếp qua companion app đã ghép đôi.
8. Các chức năng cục bộ cơ bản vẫn hoạt động khi Android hoặc Internet mất kết nối.

## 1.2. Nguyên tắc trải nghiệm

- Music/artwork là trọng tâm; notification là tiện ích phụ.
- Notification mặc định compact, không che artwork hoặc nội dung desktop quan trọng.
- Vùng nền phải click-through; chỉ vùng control được khai báo mới nhận input.
- Có thể thay đổi vị trí/kích thước widget trong giới hạn thiết kế.
- Có chế độ ẩn nội dung nhạy cảm, chỉ hiển thị tên ứng dụng/người gửi hoặc tắt hoàn toàn.
- Có reduced-motion, mute visual effects, pause rendering và quick hide.
- Không đánh đổi khả năng đọc nội dung để lấy hiệu ứng thẩm mỹ: luôn bảo đảm contrast.

## 1.3. Phạm vi MVP

### In scope

- Windows 11 là nền tảng desktop đầu tiên.
- Android là companion app.
- Desktop visual surface / wallpaper-level experience sau khi M0 chứng minh phương án khả thi.
- Album art, color palette, dynamic background, shadow/glow và transition.
- Audio capture cục bộ qua WASAPI loopback với sự đồng ý rõ ràng; DSP và four-edge visualizer.
- Windows Media Sessions (SMTC) cho metadata và điều khiển các session tương thích.
- Windows notification listener có sự cho phép của người dùng.
- Android NotificationListenerService và bridge có xác thực.
- Notification corner widget và Quick Reply theo capability thực tế của nguồn.
- Pairing, TLS, device revoke, reconnect, privacy controls.
- SQLite/settings, logging an toàn, installer và kiểm thử.

### Out of scope trong MVP

- Xây dựng dịch vụ streaming nhạc hoặc catalog backend riêng.
- Đọc toàn bộ lịch sử hội thoại/tin nhắn của mạng xã hội.
- Tự động trả lời bằng AI hoặc gửi tin nhắn không có thao tác xác nhận.
- Cam kết Quick Reply cho mọi ứng dụng nhắn tin.
- iOS notification bridge.
- Linux/macOS desktop release.
- Plugin marketplace.
- Cloud relay/server bắt buộc.
- Trích xuất cookie, session hoặc private credentials từ browser profile.
- Bypass DRM, private API, access control hoặc điều khoản dịch vụ của nền tảng nhạc/mạng xã hội.

---

# 2. Tài liệu và dự án tham khảo

## 2.1. Tài liệu nền tảng AuraDesk

- PRD: `AuraDesk_PRD_v0.2.docx` — yêu cầu sản phẩm và UX.
- Technical Architecture Specification: `AuraDesk_Technical_Architecture_Spec_v0.2.docx` — kiến trúc, module, giao thức, bảo mật và kiểm thử.
- File này: kế hoạch triển khai, trạng thái thực tế, quyết định và execution log.

Nếu PRD/TAS và file master có nội dung mâu thuẫn, không tự chọn một bên. Dừng thay đổi bị ảnh hưởng, ghi discrepancy trong Decision Log và thống nhất phiên bản baseline mới.

## 2.2. Repository tham khảo: Liner Desktop

Repository: https://github.com/tryliner/desktop  
Default branch đã xem: `main`  
Commit/tree đã tham khảo khi lập kế hoạch: `0ec572f682c4e9e3a774bbeed10ca6d27713f850` (cần kiểm tra lại HEAD mỗi khi cần phân tích sâu).

Những ý tưởng kiến trúc chung có thể học hỏi:

- Tách main process / preload / renderer và kiểm soát IPC.
- Tổ chức tính năng theo module/layer.
- Quản lý floating overlay, vị trí, vòng đời, hotkeys và visibility.
- Dùng worker riêng cho animation, giới hạn FPS và pause/resume khi không hiển thị.
- Thiết kế UI theo content-driven palette, glass surfaces và motion tiết chế.
- Có tài liệu privacy/storage và cập nhật nó khi data flow thay đổi.

**Giới hạn license:** Liner hiện dùng Source-Available & Non-Commercial Contribution License. Không sao chép, fork để tạo sản phẩm cạnh tranh, phân phối derivative, trích xuất core algorithms, audio processing, shader, lyrics synchronization, distinctive UI hoặc dùng Liner backend cho AuraDesk. Chỉ rút ra nguyên tắc kỹ thuật chung và tự triển khai độc lập. Trước mọi hình thức sử dụng source/assets/dependencies của Liner phải đọc lại license hiện hành; không suy diễn rằng public repository đồng nghĩa với open-source không hạn chế.

---

# 3. Baseline kiến trúc và công nghệ dự kiến

> Đây là baseline đề xuất, không phải quyết định bất biến. M0 phải xác minh SDK/OS/API và có quyền đề xuất thay đổi bằng Decision Log trước khi triển khai production.

## 3.1. Windows desktop

- Ngôn ngữ: C#.
- Runtime: .NET 10 LTS (xác minh SDK patch hiện hành khi setup).
- UI framework: WinUI 3.
- Packaging: MSIX để hỗ trợ package identity/capabilities khi cần.
- Windows integration: Windows App SDK, Windows SDK APIs.
- Rendering: Windows Composition / Direct2D hoặc công nghệ được chọn sau prototype.
- Media metadata/control: GlobalSystemMediaTransportControlsSessionManager / SMTC.
- Audio capture: WASAPI loopback.
- DSP: FFT, RMS, frequency bands, spectral flux/onset detection.
- Storage: SQLite cho state/config cần lưu; OS-protected storage cho secrets.
- Logging: structured logging, rotation và redaction.

## 3.2. Android companion

- Kotlin native.
- Android SDK/Android Studio; pin compile/target SDK sau khi kiểm tra chính sách Android hiện hành.
- NotificationListenerService để nhận notification được user cho phép.
- Notification.Action/RemoteInput để xác định reply capability.
- PendingIntent chỉ được giữ và thực thi trên Android; không gửi/serialize PendingIntent sang Windows.
- Android Keystore cho key material.
- Không dùng Accessibility Service để đọc hoặc thao tác ứng dụng bên thứ ba như phương án thay thế notification API.

## 3.3. Device bridge

- LAN-first, không cần cloud relay trong MVP.
- WebSocket hoặc transport phù hợp chạy trên TLS.
- Pairing bằng QR hoặc mã một lần.
- Xác thực thiết bị; không tin cậy chỉ vì cùng LAN.
- Certificate/public-key validation hoặc cơ chế pinning được thiết kế phù hợp.
- Nonce, request ID, timestamp/TTL, replay protection, payload size limits và rate limiting.
- Có device revoke, disconnect và re-pair.

## 3.4. Nguyên tắc module

Các module dự kiến:

- Desktop Shell / Desktop Surface Host.
- Core Coordinator / Event Bus.
- Music Engine / Media Session Adapters.
- Artwork & Palette Engine.
- Audio Capture Engine.
- DSP Engine.
- Visual Rendering Engine.
- Windows Notification Adapter.
- Android Notification Bridge.
- Notification Coordinator / Privacy Filter.
- Reply Action Executor.
- Settings / Storage / Diagnostics.

Module chỉ được tách thành project/package riêng khi có contract và trách nhiệm độc lập thực sự. Không tạo abstraction chỉ để tăng số lượng thư mục.

## 3.5. Event model dự kiến

- `TrackChanged`
- `PlaybackStateChanged`
- `ArtworkAvailable`
- `PaletteChanged`
- `AudioFeaturesUpdated`
- `NotificationReceived`
- `NotificationUpdated`
- `NotificationRemoved`
- `ReplyRequested`
- `ReplyResult`
- `DeviceConnectionChanged`
- `PermissionStateChanged`

Không phát từng audio sample qua Event Bus. Audio pipeline phải dùng buffer/ring buffer giới hạn trong RAM và chỉ publish feature aggregate với tần suất kiểm soát.

---

# 4. Dependency map và thứ tự triển khai

```text
M0 Feasibility & Environment
  |
  v
M1 Project Foundation
  |
  v
M2 Desktop Surface & Visual Foundation
  |\
  | \________________________
  v                          v
M3 Music & Dynamic Palette   M5 Windows Notifications
  |
  v
M4 Audio Reactive Engine
  |
  +----------------------------+
                               v
                         M6 Android Companion
                               |
                               v
                         M7 Hardening & Release
```

Lưu ý:

- M0 là gate bắt buộc; chưa qua M0 thì không khóa production architecture.
- M2 cần desktop surface host đã được chứng minh ở M0.
- M3 và M5 có thể phát triển song song sau khi M1/M2 cung cấp contracts ổn định.
- M4 phụ thuộc audio capture prototype M0 và visual renderer M2.
- M6 phụ thuộc notification schema/privacy rules từ M5 và secure transport design.
- M7 chỉ bắt đầu khi feature scope đã ổn định; một số security/performance tests phải chạy liên tục từ M1.

---

# 5. Roadmap chi tiết

## M0 — Feasibility & Environment

**Mục tiêu:** xác minh các giả định kỹ thuật có rủi ro cao trước khi xây production system.

**Trạng thái hiện tại:** `NOT STARTED`.

### M0.1 — Environment inventory

Kiểm tra và ghi lại:

- [ ] Windows edition, version, build, servicing status (`winver`).
- [ ] CPU, RAM, GPU, driver, VRAM nếu có.
- [ ] Màn hình: số lượng, resolution, refresh rate, DPI scale.
- [ ] .NET SDK/runtime hiện cài.
- [ ] Visual Studio/Build Tools và workload cần thiết.
- [ ] Windows SDK và Windows App SDK.
- [ ] Android Studio, Android SDK, JDK.
- [ ] Android phone model, Android version, OEM/battery policy.
- [ ] Git, GitHub CLI (nếu dùng), repository permissions.
- [ ] Các ứng dụng nhạc và nhắn tin thực tế sẽ dùng để test.

**Deliverable:** environment inventory và danh sách thiếu/cần nâng cấp.  
**Acceptance:** có version/build cụ thể; không dựa trên thông tin máy ghi nhớ từ phiên cũ.

### M0.2 — Desktop Surface Host POC

Prototype tối giản để so sánh:

- Phương án A: desktop-level hosting/shell integration nếu có API/phương pháp đủ ổn định.
- Phương án B: borderless, non-activating window hoặc fallback overlay.

Kiểm tra:

- [ ] Desktop icons vẫn click được.
- [ ] Taskbar không bị che.
- [ ] Cửa sổ ứng dụng bình thường luôn ở phía trên AuraDesk.
- [ ] Background click-through.
- [ ] Chỉ control được chọn nhận mouse input.
- [ ] Explorer restart: surface phục hồi hoặc đóng an toàn.
- [ ] Sleep/resume.
- [ ] Lock/unlock.
- [ ] Fullscreen/exclusive fullscreen behavior.
- [ ] Multi-monitor attach/detach.
- [ ] DPI/resolution changes.
- [ ] App crash/restart không để lại cửa sổ mồ côi.

**Acceptance gate:** không chặn thao tác desktop; không nổi trên ứng dụng thường; recover/teardown an toàn. Không dùng undocumented shell behavior làm production assumption nếu không có fallback và compatibility strategy.

### M0.3 — Windows Media Sessions POC

- [ ] Liệt kê media sessions.
- [ ] Xác định session đang phát/được chọn.
- [ ] Đọc title, artist, album, duration, playback status.
- [ ] Lấy artwork/thumbnail.
- [ ] Play/Pause/Next/Previous khi session cho phép.
- [ ] Xử lý session không cung cấp metadata.
- [ ] Xử lý nhiều media sessions.
- [ ] Test từng ứng dụng nhạc mục tiêu, ghi rõ phiên bản.

**Acceptance:** có matrix ứng dụng × capability; không tuyên bố mọi app đều hỗ trợ SMTC.

### M0.4 — WASAPI POC

- [ ] Enumerate render endpoints.
- [ ] Capture loopback từ endpoint được chọn.
- [ ] Kiểm tra format/sample rate/channel conversion.
- [ ] Phát hiện endpoint thay đổi/disconnect.
- [ ] Stop/release capture tức thì.
- [ ] Test nhiều ứng dụng cùng phát âm thanh.
- [ ] Đo CPU/RAM/latency.
- [ ] Xác minh không ghi file và không gửi audio đi nơi khác.
- [ ] Có trạng thái rõ ràng khi capture được bật/tắt.

**Acceptance:** capture/stop ổn định và có policy opt-in; không lưu raw audio.

### M0.5 — Windows Notification Listener POC

- [ ] Tạo MSIX test app.
- [ ] Khai báo capability cần thiết.
- [ ] Xin quyền người dùng.
- [ ] Enumerate/read notification phù hợp.
- [ ] Kiểm tra title/body/app identity/timestamp/action metadata thực tế được expose.
- [ ] Kiểm tra privacy settings và notification không có body.
- [ ] Thu hồi quyền trong Settings.
- [ ] Kiểm tra listener phản ứng khi quyền bị thu hồi.
- [ ] Xác minh Reply action có khả năng sử dụng theo API/nguồn thực tế hay không.

**Acceptance:** biết chính xác field/capability nào có thể đọc; generic Windows reply không được giả định.

### M0.6 — Android Notification/RemoteInput POC

- [ ] Tạo Android app tối giản.
- [ ] Implement NotificationListenerService.
- [ ] Xin user bật Notification Access.
- [ ] Chỉ nhận app trong allowlist test.
- [ ] Đọc Notification.Action và RemoteInput metadata.
- [ ] Kiểm tra action còn hiệu lực khi notification update/remove.
- [ ] Tạo test notification app do dự án kiểm soát có RemoteInput.
- [ ] Thử reply qua action hợp lệ.
- [ ] Thử notification không có RemoteInput.
- [ ] Revoke permission và kiểm tra behavior.
- [ ] Kiểm tra process restart/battery restrictions.

**Security invariant:** không serialize hoặc gửi PendingIntent sang Windows. Windows gửi `replyActionId` ngắn hạn; Android tra cứu action đang giữ trong bộ nhớ và thực thi local.

**Acceptance:** reply thành công trên test app có RemoteInput; unsupported/expired action bị từ chối an toàn.

### M0.7 — Secure LAN Pairing POC

- [ ] Windows tạo one-time pairing session.
- [ ] Android quét QR/mã ghép đôi.
- [ ] Hai thiết bị hiển thị/confirm identity phù hợp.
- [ ] Thiết lập TLS và xác minh peer.
- [ ] Trao đổi test message.
- [ ] Kiểm tra thiết bị lạ bị từ chối.
- [ ] Kiểm tra replay/expired pairing code.
- [ ] Mất mạng/reconnect.
- [ ] Revoke pairing.
- [ ] Không expose listener ra public network.

**Acceptance:** kết nối không dựa vào LAN trust; thiết bị không được ghép đôi không gửi được dữ liệu/hành động.

### M0.8 — M0 Exit Review

- [ ] Tổng hợp mọi POC.
- [ ] Ghi limitations và compatibility matrix.
- [ ] Ghi performance measurements.
- [ ] Chốt API được chọn và API bị loại.
- [ ] Đánh giá desktop-host blocker.
- [ ] Cập nhật baseline công nghệ.
- [ ] Cập nhật Decision Log.
- [ ] Người dùng xác nhận scope thay đổi nếu có.

**M0 chỉ `DONE` khi mọi gate quan trọng có bằng chứng và có quyết định rõ ràng.**

---

## M1 — Project Foundation

**Mục tiêu:** tạo solution, module boundaries, event model, settings, storage, logging và CI.

### M1.1 — Repository/bootstrap

- [ ] Tạo GitHub repository riêng, ưu tiên Private.
- [ ] Clone về `C:\AuraDesk` (nếu không có thay đổi khác).
- [ ] Tạo `.gitignore`, `.editorconfig`, solution và project files.
- [ ] Thiết lập branch protection cho `main` nếu khả dụng.
- [ ] Chọn commit convention.
- [ ] Tạo GitHub Actions workflow cho build/test.
- [ ] Tạo Android companion project tối thiểu khi M1 cần.

### M1.2 — Solution structure

Cấu trúc dự kiến, sẽ tinh chỉnh theo M0/M1:

```text
AuraDesk/
├── src/
│   ├── AuraDesk.Desktop/
│   ├── AuraDesk.Core/
│   ├── AuraDesk.Windows/
│   ├── AuraDesk.Music/
│   ├── AuraDesk.Audio/
│   ├── AuraDesk.Visual/
│   ├── AuraDesk.Notifications/
│   └── AuraDesk.DeviceBridge/
├── android/
│   └── AuraDesk.Companion/
├── tests/
│   ├── Unit/
│   ├── Integration/
│   └── Compatibility/
├── scripts/
└── .github/workflows/
```

Không bắt buộc tạo toàn bộ project ngay ngày đầu. Chỉ tạo project khi module có implementation hoặc contract đủ rõ.

### M1.3 — Core contracts

- [ ] Định nghĩa `IDesktopSurfaceHost`.
- [ ] Định nghĩa `IMediaSessionSource`.
- [ ] Định nghĩa `IMusicProvider`/capability model.
- [ ] Định nghĩa `IPaletteEngine`.
- [ ] Định nghĩa `IAudioCapture`.
- [ ] Định nghĩa `IDspEngine`.
- [ ] Định nghĩa `IVisualRenderer`.
- [ ] Định nghĩa `INotificationSource`.
- [ ] Định nghĩa `IReplyActionExecutor`.
- [ ] Định nghĩa `IDeviceBridge`.
- [ ] Định nghĩa lifecycle/cancellation/error contracts.

### M1.4 — Event Bus

- [ ] Implement event dispatch.
- [ ] Xử lý subscriber lifecycle/unsubscribe.
- [ ] Tránh memory leak từ subscriber.
- [ ] Có backpressure/coalescing cho event tần suất cao.
- [ ] Không gửi raw samples qua bus.
- [ ] Viết unit tests cho ordering, cancellation, exception isolation.

### M1.5 — Settings/storage/secrets

- [ ] Versioned settings schema.
- [ ] SQLite schema và migrations.
- [ ] Data retention/cleanup policy.
- [ ] DPAPI/OS-protected storage cho pairing secrets/credentials phù hợp.
- [ ] Không lưu notification body hoặc audio samples.
- [ ] Export/import settings không chứa secret.

### M1.6 — Logging/diagnostics

- [ ] Structured logging.
- [ ] Log levels.
- [ ] Rotation/size cap.
- [ ] Redaction cho token, key, message body, IP nhạy cảm nếu cần.
- [ ] Crash/exception boundary.
- [ ] Diagnostic bundle không chứa secret hoặc notification content.

### M1.7 — CI baseline

- [ ] Restore dependencies.
- [ ] Build Windows solution.
- [ ] Run unit tests.
- [ ] Build Android companion.
- [ ] Static analysis/format check.
- [ ] Publish test result.
- [ ] Fail pipeline khi test/build thất bại.

**M1 Acceptance:** clean checkout build được; test baseline pass; module contracts được kiểm tra; shutdown không để worker/process treo.

---

## M2 — Desktop Surface & Visual Foundation

**Mục tiêu:** có ambient desktop surface thật với layout và widget mock data.

### M2.1 — Desktop Surface Host

- [ ] Implement phương án host đã được M0 phê duyệt.
- [ ] Có fallback nếu host không khả dụng.
- [ ] Quản lý window ownership/lifecycle.
- [ ] Không chặn desktop input ngoài interactive bounds.
- [ ] Đặt z-order đúng.
- [ ] Explorer restart recovery.
- [ ] Multi-monitor/DPI handling theo phạm vi được chọn.
- [ ] Fullscreen behavior.

### M2.2 — Visual canvas

- [ ] Background canvas.
- [ ] Artwork central placeholder.
- [ ] Clock widget.
- [ ] Music widget placeholder.
- [ ] Notification corner widget placeholder.
- [ ] Four-edge visualization canvas placeholder.
- [ ] Theme tokens.
- [ ] Contrast behavior.
- [ ] Reduced motion.

### M2.3 — Layout/hit testing

- [ ] Layout bounds và safe area.
- [ ] Vị trí widget có thể cấu hình.
- [ ] Scale theo DPI.
- [ ] Hit-test map.
- [ ] Background click-through.
- [ ] Music controls interactive.
- [ ] Notification card interactive.
- [ ] Reply composer interactive.
- [ ] Drag/resize behavior nếu được chấp nhận.

### M2.4 — Corner notification UX mock

- [ ] Mặc định góc dưới bên phải.
- [ ] Có thể cấu hình góc/vị trí.
- [ ] Compact state chỉ chiếm vùng nhỏ.
- [ ] Không tự expand khi notification tới.
- [ ] Có giới hạn số card hiển thị (mặc định tối đa 3 card compact; xác minh qua UX test).
- [ ] Click mới mở expanded state.
- [ ] Reply composer chỉ hiện khi người dùng chọn Reply và capability được xác nhận.
- [ ] Animation nhẹ, không làm mất focus desktop.

### M2.5 — Surface reliability

- [ ] Hide/show hotkey.
- [ ] Pause effects khi hidden.
- [ ] Recover sau display changes.
- [ ] Clean shutdown.
- [ ] No orphan windows/processes.

**M2 Acceptance:** surface ổn định, không chặn desktop; music và corner notification mock cùng hiển thị; bố cục đúng thứ bậc thị giác.

---

## M3 — Music Integration & Dynamic Palette

**Mục tiêu:** nhận metadata/artwork từ media sessions tương thích, tự đổi palette/background và điều khiển playback theo capability.

### M3.1 — SMTC adapter

- [ ] Enumerate sessions.
- [ ] Track selected/active session.
- [ ] Manual session selection.
- [ ] Auto-selection policy.
- [ ] Read metadata.
- [ ] Read playback status/timeline.
- [ ] Retrieve artwork thumbnail.
- [ ] Play/Pause/Next/Previous capability check.
- [ ] Session add/remove/update handling.
- [ ] Application compatibility matrix.

### M3.2 — Music provider capability model

Mỗi provider/session phải mô tả riêng các khả năng:

- Search.
- Metadata.
- Artwork.
- Playback control.
- Queue/playlist access.
- Lyrics (nếu có nguồn hợp pháp/được phép).

Không suy luận rằng API có search thì có stream playback, hoặc có metadata thì có quyền điều khiển.

### M3.3 — Artwork pipeline

- [ ] Artwork resolver.
- [ ] URL/stream validation.
- [ ] File size/dimension limits.
- [ ] Safe decode.
- [ ] Resize/downsample.
- [ ] Invalid image fallback.
- [ ] Cache with bounded size/TTL.
- [ ] Cancel stale request khi track đã đổi.

### M3.4 — Palette extraction

Đánh giá bằng benchmark:

- Median Cut.
- K-means hoặc phương án quantization tương đương.

Đầu ra đề xuất:

- Primary.
- Secondary.
- Accent.
- Background.
- Contrast text.

- [ ] Color extraction unit tests.
- [ ] Test ảnh sáng/tối, đơn sắc, nhiều màu, artwork nhỏ.
- [ ] Contrast analysis.
- [ ] Palette cache.
- [ ] Fallback palette.

### M3.5 — Dynamic rendering

- [ ] Gradient background.
- [ ] Artwork center treatment.
- [ ] Shadow/glow.
- [ ] Smooth palette interpolation.
- [ ] Không đổi màu đột ngột.
- [ ] Cancel/ignore stale track events.
- [ ] Reduced motion.
- [ ] Benchmark CPU/GPU/memory.

### M3 Acceptance

- Track đổi → artwork và palette cập nhật đúng.
- Không có artwork → fallback đẹp và ổn định.
- Session mất → không crash, hiển thị idle state.
- Metadata update P95 mục tiêu ≤ 2 giây trên profile kiểm thử đã chọn.
- Palette transition không gây UI freeze.

---

## M4 — Audio Reactive Engine

**Mục tiêu:** hiệu ứng viền bốn cạnh thực sự phản ứng theo tín hiệu audio, hoạt động hiệu quả và có privacy controls.

### M4.1 — Audio capture

- [ ] Enumerate output devices.
- [ ] Select endpoint.
- [ ] WASAPI loopback session.
- [ ] Format negotiation.
- [ ] Channel mixing.
- [ ] Bounded ring buffer.
- [ ] Device change/disconnect handling.
- [ ] Stop/release on disable/lock/exit theo policy.
- [ ] Explicit user opt-in and active indicator.
- [ ] Không ghi raw samples xuống disk.
- [ ] Không upload audio.

### M4.2 — DSP

- [ ] Window function.
- [ ] FFT.
- [ ] RMS.
- [ ] Low/Mid/High bands.
- [ ] Spectral flux.
- [ ] Onset detection.
- [ ] Noise floor/noise gate.
- [ ] Normalization.
- [ ] Attack/release smoothing.
- [ ] Beat behavior test với bộ test audio do dự án sở hữu/tự tạo.

Không tuyên bố beat detection chính xác chỉ dựa trên FFT. Cần đánh giá onset detector và false positives/false negatives.

### M4.3 — Four-edge visualizer

Mapping ban đầu (cần kiểm tra UX):

- Top: high frequency + onset.
- Right: mid energy.
- Bottom: low/bass energy.
- Left: RMS/overall energy + onset.

- [ ] Dải sáng theo cạnh.
- [ ] Corner propagation.
- [ ] Glow intensity.
- [ ] Border thickness.
- [ ] Sensitivity.
- [ ] Brightness.
- [ ] Smoothing.
- [ ] FPS cap 60; fallback 30.
- [ ] Pause khi không hiển thị.
- [ ] Reduced motion / disable visualizer.

### M4.4 — Performance

- [ ] DSP không chạy trên UI thread.
- [ ] Không publish từng sample lên Event Bus.
- [ ] Coalesce feature updates.
- [ ] No unbounded allocations per frame.
- [ ] GPU resource cleanup.
- [ ] CPU/GPU/RAM benchmark.
- [ ] Power Saver/degraded mode.
- [ ] 30–60 minute soak test.

### M4 Acceptance

- Im lặng → viền không nhấp nháy ngẫu nhiên.
- Bass/mid/high → phản ứng khác biệt, có thể kiểm soát.
- Device change/disconnect → graceful recovery.
- Capture stop → audio stream thực sự dừng.
- Không có audio data persistence/upload.
- Performance mục tiêu đạt trên hardware profile đã ghi nhận.

---

## M5 — Windows Notifications

**Mục tiêu:** hiển thị notification Windows trong corner widget và hỗ trợ action chỉ khi được xác minh.

### M5.1 — Windows notification adapter

- [ ] MSIX capability.
- [ ] User permission flow.
- [ ] Listener lifecycle.
- [ ] Notification normalization.
- [ ] App identity mapping.
- [ ] Update/remove handling.
- [ ] Permission revoke handling.
- [ ] Unsupported/missing field fallback.

### M5.2 — Privacy filter

- [ ] Application allowlist/denylist.
- [ ] Full Preview.
- [ ] Title/ sender only.
- [ ] Hidden Preview.
- [ ] Disable app.
- [ ] Quiet hours.
- [ ] Lock-screen behavior.
- [ ] No sensitive body in logs.
- [ ] In-memory retention/TTL.

### M5.3 — Notification coordinator/UI

- [ ] Compact corner card.
- [ ] Expanded detail.
- [ ] Group/update same notification.
- [ ] Dismiss/remove behavior.
- [ ] Queue/card cap.
- [ ] Timeout/expiry.
- [ ] Accessibility/keyboard.
- [ ] Contrast across palette changes.

### M5.4 — Windows Reply capability

- [ ] Determine whether source exposes supported reply action.
- [ ] Build per-application capability matrix.
- [ ] Show Reply only when verified.
- [ ] Otherwise show Open Application / Dismiss.
- [ ] Never inject into app UI or use undocumented message APIs.
- [ ] Do not claim sent/delivered/read without source confirmation.

### M5 Acceptance

- Notification appears in compact corner region.
- Does not cover central artwork unnecessarily.
- Revoke permission immediately disables source.
- Privacy mode hides content.
- App-specific reply only works where documented and verified.

---

## M6 — Android Companion & Quick Reply

**Mục tiêu:** nhận notification từ Android và chuyển tiếp an toàn; Quick Reply trên điện thoại chỉ với action còn hợp lệ.

### M6.1 — Companion application

- [ ] Kotlin Android project.
- [ ] Onboarding and pairing UI.
- [ ] Notification Access explanation.
- [ ] NotificationListenerService.
- [ ] App allowlist.
- [ ] Connection status.
- [ ] Device identity.
- [ ] Revoke/unpair flow.
- [ ] Background behavior and OEM compatibility.

### M6.2 — Pairing and transport

- [ ] QR/one-time pairing code.
- [ ] Secure key establishment.
- [ ] TLS.
- [ ] Peer validation.
- [ ] Device trust storage.
- [ ] Message schema/version.
- [ ] Request IDs/nonces/TTL.
- [ ] Replay protection.
- [ ] Rate limits.
- [ ] Payload limits.
- [ ] Reconnect/backoff.
- [ ] Device revocation.

### M6.3 — Notification bridge

Message types dự kiến:

- `PairingHello`
- `PairingAccepted`
- `NotificationEnvelope`
- `NotificationUpdated`
- `NotificationRemoved`
- `ReplyCommand`
- `ReplyResult`
- `Heartbeat`
- `RevokePairing`

Notification payload chỉ gửi field cần thiết theo privacy setting. Không truyền credential hoặc PendingIntent.

### M6.4 — Quick Reply executor

- [ ] Android giữ action reference/PendingIntent local.
- [ ] Windows chỉ nhận opaque `replyActionId` ngắn hạn.
- [ ] Action reference gắn với notification identity và TTL.
- [ ] Validate device, notification, action và request.
- [ ] Check RemoteInput exists.
- [ ] Check action còn hiệu lực.
- [ ] Thực thi fill-in intent trên Android.
- [ ] Return `SUBMITTED`, `FAILED`, `EXPIRED`, `UNSUPPORTED`, `SOURCE_UNAVAILABLE`.
- [ ] Remove stale action references.
- [ ] Không giả định SUBMITTED = Delivered/Read.
- [ ] Không tự gửi khi người dùng chưa xác nhận Send.

### M6.5 — Reliability

- [ ] Screen locked.
- [ ] App in background.
- [ ] Wi-Fi disconnect/reconnect.
- [ ] IP address changes.
- [ ] Companion process killed/restarted.
- [ ] Phone reboot.
- [ ] Battery optimization/OEM behavior.
- [ ] Duplicate notification suppression.
- [ ] Expired action.
- [ ] Revoke pairing while connected.

### M6 Acceptance

- Pairing chỉ chấp nhận thiết bị được xác minh.
- Notification relay có privacy filter.
- Reconnect không tạo duplicate không kiểm soát.
- Quick Reply test thành công trên app có RemoteInput đã xác minh.
- Action stale/expired bị từ chối.
- PendingIntent không rời Android.
- Unpair xóa trust material.

---

## M7 — Security, Performance, Compatibility & Release

**Mục tiêu:** đưa MVP vào beta có kiểm soát.

### M7.1 — Security testing

- [ ] IPC allowlist/schema validation.
- [ ] Privilege boundary review.
- [ ] Local storage review.
- [ ] Secret storage review.
- [ ] TLS/peer verification.
- [ ] Pairing code expiry/replay tests.
- [ ] Device revoke tests.
- [ ] Notification privacy/logging audit.
- [ ] Reply action ID validation/TTL.
- [ ] Image parser/resource limits.
- [ ] Update signature/integrity.
- [ ] Dependency and vulnerability scan.
- [ ] Threat model refresh.

### M7.2 — Performance testing

Profiles:

1. Idle.
2. Music only.
3. Music + visualizer.
4. Music + visualizer + Windows notification.
5. Full load with Android bridge.

Metrics:

- CPU average/peak.
- GPU utilization.
- RAM working set.
- UI frame pacing.
- Audio capture/DSP latency.
- Track metadata update latency.
- Notification relay latency.
- Reconnect duration.
- Thermal/power behavior where observable.

Targets ban đầu cần kiểm chứng:

- 60 FPS khi hardware cho phép; fallback 30 FPS.
- Track update P95 ≤ 2 seconds trên profile mục tiêu.
- Android notification relay P95 ≤ 2 seconds trên LAN ổn định.
- Crash-free session goal ≥ 99.5% trong beta; phải có telemetry/crash measurement có consent hoặc phương pháp test nội bộ rõ ràng.
- Không lưu hoặc truyền raw audio ngoài máy.
- Không lưu notification content không cần thiết.

### M7.3 — Compatibility matrix

Windows:

- [ ] Windows version/build được hỗ trợ.
- [ ] Integrated GPU.
- [ ] Discrete GPU.
- [ ] 100/125/150% DPI.
- [ ] Single monitor.
- [ ] Multi-monitor.
- [ ] 60 Hz và refresh cao hơn.
- [ ] Explorer restart.
- [ ] Sleep/resume.

Android:

- [ ] Hai hoặc nhiều Android version trong support range.
- [ ] Ít nhất hai OEM/device.
- [ ] Battery optimization behavior.
- [ ] Notification source variations.
- [ ] RemoteInput support variations.

### M7.4 — Release engineering

- [ ] MSIX package.
- [ ] Signing certificate and secure handling.
- [ ] Install/upgrade/rollback tests.
- [ ] Clean uninstall.
- [ ] Android APK signing.
- [ ] Versioning.
- [ ] Release notes.
- [ ] Crash/diagnostic collection policy.
- [ ] Known limitations.
- [ ] Beta acceptance checklist.

**M7 Acceptance:** security, compatibility, performance và packaging gates đạt; mọi known limitation được ghi rõ; không phát hành nếu còn lỗi nghiêm trọng ảnh hưởng dữ liệu riêng tư hoặc quyền thiết bị.

---

# 6. Quy tắc bảo mật và quyền riêng tư xuyên suốt

## 6.1. Notification privacy

- Default: user chủ động chọn app được chuyển tiếp.
- Cho phép preview mode và hidden-content mode.
- Notification content chỉ ở RAM trong thời gian cần thiết, trừ khi có yêu cầu lưu rõ ràng và được người dùng đồng ý.
- Không log message body, OTP, token hoặc nội dung notification.
- Khi lock screen/quiet mode, ẩn nội dung theo policy.
- Khi revoke source permission, ngừng nhận và dọn dữ liệu liên quan.

## 6.2. Audio privacy

- WASAPI capture phải có opt-in rõ ràng.
- Có trạng thái/indicator khi đang capture.
- Nút dừng phải dừng thực sự capture session.
- Chỉ phân tích trong bộ nhớ; không ghi raw audio.
- Không upload audio.
- Vì loopback có thể nhận system mix, UI phải nói rõ phạm vi capture.

## 6.3. Device bridge security

- Không tin cậy chỉ vì thiết bị cùng LAN.
- Không mở cổng ra Internet trong MVP.
- Pairing code có entropy, TTL và giới hạn thử.
- TLS và peer validation.
- Request authentication, nonce, timestamp, replay protection.
- Device revoke có hiệu lực ngay.
- Rate/payload limits.
- Không truyền PendingIntent hoặc secret của ứng dụng nguồn.

## 6.4. Desktop security

- Không yêu cầu Administrator nếu không cần.
- IPC có allowlist và schema validation.
- Renderer/UI không giữ secret.
- Main/system boundary xác thực caller.
- Không dùng `eval`/dynamic remote script cho visual effects.
- Image decoder có giới hạn kích thước, định dạng và memory.
- Update package phải kiểm tra chữ ký/toàn vẹn.

## 6.5. Third-party integration

- Ưu tiên API/Windows integration chính thức.
- Không scrape hoặc reverse private API để vượt giới hạn.
- Không đọc browser cookies/profile.
- Không bypass DRM hoặc access control.
- Document provider capability và ToS assumptions.
- Nếu API không cho phép hoặc không hỗ trợ, hiển thị unsupported/fallback thay vì lách giới hạn.

---

# 7. Risk Register

| ID | Rủi ro | Mức độ | Phương án xử lý | Trạng thái |
|---|---|---:|---|---|
| R-001 | Desktop-level hosting không ổn định/không được hỗ trợ | Very High | M0 POC, fallback window, compatibility matrix | OPEN |
| R-002 | Một số music apps không cung cấp SMTC đầy đủ | High | Capability matrix, provider adapter, fallback | OPEN |
| R-003 | Windows không có generic reply action cho mọi app | High | App-specific verified connector; otherwise Open App | OPEN |
| R-004 | Android action/PendingIntent hết hạn | High | Opaque action ID, TTL, local lookup, safe failure | OPEN |
| R-005 | Android background service bị hạn chế bởi OEM/battery | High | Device testing, reconnect, user guidance | OPEN |
| R-006 | Notification content bị lộ trong log/storage | Very High | Redaction, in-memory processing, tests | OPEN |
| R-007 | WASAPI thu âm thanh ngoài ý muốn | High | Explicit opt-in, indicator, stop control | OPEN |
| R-008 | GPU/CPU usage quá cao | Medium | FPS caps, worker, degrade mode | OPEN |
| R-009 | Third-party API/ToS thay đổi | High | Official APIs, adapters, graceful degradation | OPEN |
| R-010 | Update package bị giả mạo | High | Signed package and integrity verification | OPEN |
| R-011 | Notification bridge bị replay/unauthorized | Very High | TLS, pairing, nonce, auth, replay protection | OPEN |
| R-012 | Image input gây resource exhaustion | Medium | Size/dimension limits, decode isolation | OPEN |

Mỗi rủi ro chỉ được chuyển sang `MITIGATED` hoặc `CLOSED` khi có kiểm thử/evidence và ghi nhận quyết định.

---

# 8. Git workflow và quy tắc triển khai batch

## 8.1. Repository

- Repository riêng: `AuraDesk` (Private được khuyến nghị).
- Project root dự kiến: `C:\AuraDesk`.
- `main`: trạng thái ổn định đã build/test.
- Feature branch theo milestone/batch.

## 8.2. Branch naming

Ví dụ:

- `feature/m0-environment-audit`
- `feature/m0-desktop-host-poc`
- `feature/m0-smtc-poc`
- `feature/m1-foundation`
- `feature/m2-desktop-surface`
- `feature/m3-music-palette`
- `feature/m4-audio-visualizer`
- `feature/m5-windows-notifications`
- `feature/m6-android-companion`
- `feature/m7-release-hardening`
- `fix/<short-description>`

## 8.3. Batch implementation protocol

Mỗi batch phải có:

1. Mục tiêu cụ thể.
2. Phạm vi file/module được phép thay đổi.
3. Những điều không thay đổi.
4. Test cases/acceptance criteria.
5. Backup hoặc Git checkpoint phù hợp.
6. Implementation.
7. Unit/integration tests.
8. Build/type/lint/static checks phù hợp.
9. `git diff --check` và review diff.
10. Commit/PR.
11. Cập nhật file master này.

Không gom các thay đổi không liên quan vào một batch. Không sửa rộng khi chưa hiểu code hiện tại. Trước khi sửa phải đọc file, test, contract và call sites liên quan.

## 8.4. Commit convention

Dùng Conventional Commits hoặc convention thống nhất trong repository, ví dụ:

- `feat(surface): add desktop host abstraction`
- `feat(music): add SMTC session discovery`
- `fix(notifications): reject expired reply action`
- `test(audio): cover loopback device disconnect`
- `docs(project): update implementation checkpoint`

## 8.5. Definition of Done cho một batch

- Code đúng scope.
- Build thành công.
- Tests liên quan pass.
- Không có lỗi mới chưa được giải thích.
- Diff được review.
- Không chứa secret/dữ liệu cá nhân.
- Có commit hoặc ghi rõ lý do chưa commit.
- Execution Log được cập nhật.

---

# 9. Session protocol — cách bắt đầu và kết thúc mỗi phiên làm việc

## 9.1. Khi bắt đầu phiên mới

1. Đọc file master này từ đầu đến phần status/log mới nhất.
2. Đọc GitHub repository: default branch, latest commit, current branch/PR nếu có.
3. Kiểm tra working tree/status khi có quyền truy cập môi trường local.
4. Đối chiếu file master với code/tests thực tế.
5. Không hỏi lại thông tin đã được ghi rõ và còn đúng.
6. Nếu tài liệu và repository lệch nhau, ghi discrepancy và xác minh trước khi sửa.
7. Chỉ làm batch tiếp theo được ghi trong `Next Actions`, trừ khi có lý do kỹ thuật được giải thích.

## 9.2. Trong phiên

- Nêu mục tiêu và file/phạm vi trước khi sửa.
- Giải thích lý do kiến trúc của thay đổi quan trọng.
- Làm theo batch; tránh chỉnh sửa tùy tiện.
- Cập nhật người dùng khi có kết quả kiểm chứng đáng kể.
- Không tuyên bố thành công trước khi tool/build/test xác nhận.

## 9.3. Khi kết thúc phiên

Cập nhật:

- Milestone/batch đang làm.
- Những gì thực sự hoàn thành.
- File/project đã thay đổi.
- Branch và commit SHA.
- Lệnh đã chạy và test result.
- Những gì chưa làm hoặc bị block.
- Rủi ro/decision mới.
- Next Actions chính xác.

Nếu không có quyền ghi trực tiếp lên GitHub, cung cấp file master đã cập nhật để người dùng commit/upload; không nói rằng đã push khi chưa thực hiện.

---

# 10. Milestone Status Board

Trạng thái ở ngày tạo tài liệu. Mọi milestone hiện chưa được triển khai trừ khi có bằng chứng mới được ghi trong Execution Log.

| Milestone | Status | Exit evidence | Notes |
|---|---|---|---|
| M0 — Feasibility & Environment | NOT STARTED | POC matrix + environment inventory + gate review | Current first milestone |
| M1 — Project Foundation | BLOCKED BY M0 | Clean build + tests + CI | Chờ M0 hoàn tất |
| M2 — Desktop Surface & Visual | BLOCKED BY M0/M1 | Desktop host/hit-test/recovery tests | Surface hosting is critical risk |
| M3 — Music & Palette | BLOCKED BY M0/M1/M2 | SMTC matrix + palette tests | SMTC capability varies by source |
| M4 — Audio Reactive | BLOCKED BY M0/M1/M2 | WASAPI/DSP/performance tests | Audio capture must be opt-in |
| M5 — Windows Notifications | BLOCKED BY M0/M1/M2 | Permission/privacy/action tests | Generic reply not guaranteed |
| M6 — Android Companion | BLOCKED BY M0/M1/M5 | Pairing/relay/reply tests | PendingIntent stays on Android |
| M7 — Hardening & Release | BLOCKED BY M1–M6 | Signed packages + security/perf gates | Beta only after gates pass |

---

# 11. Current Project Status

**Last updated:** 2026-11-17  
**Overall status:** `PLANNING APPROVED — M0 NOT STARTED`  
**Current milestone:** `M0 — Feasibility & Environment`  
**Current batch:** `M0.1 — Environment inventory` (chưa chạy)  
**Repository:** chưa có repository AuraDesk được xác nhận/kết nối.  
**Local project root:** dự kiến `C:\AuraDesk`; chưa xác nhận đã tạo.  
**Baseline docs:** PRD v0.2 và TAS v0.2 đã được soạn; đang dùng làm reference.  
**Reference repo:** Liner Desktop đã đọc README, LICENSE, DESIGN, PRIVACY, STORAGE, overlay, ambient worker và cấu trúc project ở mức ban đầu. Không tái sử dụng source/assets/code bị license hạn chế.

## 11.1. Verified completed

- [x] Đã thống nhất hướng sản phẩm AuraDesk.
- [x] Đã xác định Windows desktop + Android companion.
- [x] Đã xác định notification widget nhỏ nằm ở góc cùng ambient music surface.
- [x] Đã chọn mục tiêu tương tác Reply khi nguồn hỗ trợ.
- [x] Đã soạn PRD v0.2.
- [x] Đã soạn Technical Architecture Specification v0.2.
- [x] Đã xem repository tham khảo Liner và nhận diện giới hạn license.
- [x] Đã lập roadmap M0–M7 và quy trình Git/batch/checkpoint.

## 11.2. Not yet done

- [ ] Tạo repository GitHub riêng AuraDesk.
- [ ] Tạo local project root.
- [ ] Kiểm tra lại Windows build/servicing status.
- [ ] Kiểm tra SDK/IDE/Android environment.
- [ ] Liệt kê ứng dụng nhạc/nhắn tin sẽ dùng để test.
- [ ] Chạy các POC M0.
- [ ] Chốt production stack sau M0.
- [ ] Tạo production solution.

## 11.3. Current blockers

- Chưa có repository AuraDesk được xác nhận.
- Chưa có environment inventory mới nhất.
- Desktop surface hosting chưa được chứng minh trên máy mục tiêu.
- SMTC compatibility chưa được test trên các app nhạc mục tiêu.
- WASAPI behavior/performance chưa được test.
- Windows notification fields/actions chưa được kiểm chứng trên MSIX POC.
- Android RemoteInput compatibility chưa được kiểm chứng với app mục tiêu.
- Secure LAN pairing POC chưa được triển khai.

---

# 12. Next Actions — thứ tự công việc gần nhất

Không nhảy sang M1 trước khi M0 gate được thông qua.

1. **Tạo repository GitHub Private `AuraDesk`** và xác nhận quyền truy cập.
2. Tạo `C:\AuraDesk` bằng clone repository; thêm file master này vào repository.
3. Chạy `winver` và ghi Windows edition/version/build.
4. Thu thập .NET SDK, Visual Studio/Build Tools, Windows SDK, Android Studio/JDK/SDK hiện có.
5. Ghi cấu hình CPU/RAM/GPU/monitor và thiết bị Android.
6. Lập danh sách ứng dụng nhạc và nhắn tin mục tiêu.
7. Tạo branch `feature/m0-environment-audit`.
8. Hoàn tất M0.1 và lưu kết quả trong Execution Log.
9. Tiếp tục từng POC M0.2–M0.7; không gộp tất cả vào một prototype lớn.
10. Tổ chức M0 Exit Review, chốt stack và cập nhật Decision Log.

---

# 13. Execution Log — append-only theo từng phiên/batch

> Mỗi lần làm việc thêm một entry mới ở đầu mục này hoặc ở trên cùng. Không xóa lịch sử cũ; nếu entry sai, thêm correction có timestamp.

## Entry template

```text
### YYYY-MM-DD — [Milestone] — [Batch ID]
Status: VERIFIED / IMPLEMENTED-NOT-VERIFIED / BLOCKED
Branch:
Commit SHA:
Scope:
Changed files:
Implementation summary:
Commands executed:
Build result:
Test result:
Manual verification:
Evidence/environment:
Issues/limitations:
Decision(s):
Next actions:
Master document version:
```

## Initial entry — 2026-11-17

### 2026-11-17 — Planning — PLAN-001

Status: `VERIFIED (planning artifact created; implementation not started)`  
Branch: N/A  
Commit SHA: N/A  
Scope: Khóa định hướng sản phẩm, roadmap M0–M7, rủi ro, Git workflow và session protocol.  
Changed files: `AURADESK_IMPLEMENTATION_MASTER.md` (tài liệu này).  
Implementation summary: Đã tạo kế hoạch triển khai chi tiết. Chưa có source code AuraDesk.  
Build result: N/A.  
Test result: N/A.  
Manual verification: Đối chiếu với phạm vi đã thống nhất và PRD/TAS v0.2.  
Issues/limitations: Chưa có repository AuraDesk; M0 chưa bắt đầu; các API chưa được POC trên máy mục tiêu.  
Decision(s): Windows-first; Android companion; corner notification widget; Reply capability-gated; GitHub là nguồn sự thật cho code/history; file master này là nhật ký tiến độ.  
Next actions: Tạo repository, clone vào `C:\AuraDesk`, thực hiện M0.1.  
Master document version: 1.0.

---

# 14. Decision Log

| ID | Ngày | Quyết định | Lý do | Trạng thái |
|---|---|---|---|---|
| D-001 | 2026-11-17 | Windows 11 desktop là release platform đầu tiên | Tập trung API desktop và giảm phạm vi | APPROVED |
| D-002 | 2026-11-17 | Android là companion cho notification | Android cung cấp NotificationListenerService; tách mobile khỏi desktop | APPROVED |
| D-003 | 2026-11-17 | Notification là corner widget nhỏ trên ambient surface | Không lấn át artwork/music | APPROVED |
| D-004 | 2026-11-17 | Quick Reply chỉ khi source capability được xác minh | Không thể đảm bảo reply phổ quát | APPROVED |
| D-005 | 2026-11-17 | PendingIntent/action handle stays on Android | Không serialize hệ thống capability sang PC | APPROVED |
| D-006 | 2026-11-17 | WASAPI raw audio chỉ xử lý cục bộ trong RAM | Quyền riêng tư và giảm rủi ro dữ liệu | APPROVED |
| D-007 | 2026-11-17 | GitHub riêng là nguồn sự thật cho code/history | Theo dõi code và tiến độ dài hạn | APPROVED |
| D-008 | 2026-11-17 | File master MD này là project execution log | Giảm phụ thuộc vào conversation memory | APPROVED |
| D-009 | 2026-11-17 | Liner chỉ là tài liệu tham khảo nguyên tắc; không sao chép phần bị license hạn chế | Tuân thủ license và tạo implementation độc lập | APPROVED |
| D-010 | 2026-11-17 | .NET 10 LTS + WinUI 3 + Kotlin là baseline đề xuất, chưa khóa trước M0 | Native Windows integration; Android native APIs | PROPOSED — M0 VERIFY |
| D-011 | 2026-11-17 | Desktop surface hosting là M0 critical gate | Rủi ro ảnh hưởng toàn bộ kiến trúc hiển thị | APPROVED |

Khi phát sinh quyết định mới, thêm dòng mới; không ghi đè lịch sử quyết định đã được thay thế. Ghi `SUPERSEDED BY D-xxx` khi cần.

---

# 15. Change Log của tài liệu

| Version | Ngày | Thay đổi |
|---|---|---|
| 1.0 | 2026-11-17 | Tạo master implementation plan; ghi scope, stack baseline, M0–M7, risk register, Git workflow, session protocol, status board và execution log. |


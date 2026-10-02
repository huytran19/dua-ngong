# Lunar Capital Tool (repo dua-ngong)

Game bốc thăm/giveaway chạy trên trình duyệt, dùng cho guild. Toàn bộ nằm trong **một file `index.html`** (HTML + CSS + JS + sprite nhúng base64), không cần build, không phụ thuộc thư viện. Mở trực tiếp bằng trình duyệt là chạy.

## Cấu trúc trong index.html
- `<style>`: CSS. Màu theo token trên `:root`. **Mặc định giao diện tối** (không theo cài đặt máy); nút ☀️/🌙 trên thanh trên đặt `data-theme="light"` trên `<html>`, lưu ở `dua-ngong-theme`. Một script nhỏ trong `<head>` đặt giao diện trước khi vẽ để không chớp sáng. Viền sân chơi và khung người thắng dùng token `--frame`.
- `<script>`: một IIFE, chia theo khối comment:
  - **core**: hằng số, RNG, sprite engine, tô màu, thoại (`LINES`, `line()`, `say()`), âm thanh (`honk`, `blip`), `countdown`, `loop`.
  - **Setup**: `MODE_INFO`, tên gọi theo đàn thú (`noun()`, `Noun()`), các tuỳ chọn, cặp dính/cấm, nút xáo tên, `go()`, `solveTeams()`.
  - **Lịch sử các ván**: lưu localStorage `dua-ngong-history`, tối đa 100 ván.
  - **Result screen**: `showResult(cfg)` dùng chung cho mọi mode.
  - **Mode: Đua Ngỗng (give)**, **Chia team**, **Sinh tồn**, **Trứng nở**, **Vòng quay may mắn (pass)** (`startRing`), **Giải đấu (cup)** ở cuối file.

## 6 mode
Thứ tự nút trên màn chuẩn bị: Đua Ngỗng, Vòng quay may mắn, Chia team, Sinh tồn, Trứng nở, Giải đấu. Mã mode trong code vẫn giữ tên cũ (`give`, `pass`).
- **give** (hiển thị: "Đua Ngỗng"): đua ngang, camera bám con dẫn đầu, 3 độ dài 15/30/50s.
- **team**: 2/3/4 đội đều sĩ số; cặp "dính" (union-find) và "không chung chuồng"; giải bằng quay lui có trọng số trong `solveTeams`. Tốc độ Nhanh/Vừa/Lề mề có giờ tối đa 15/30/60s (gần hết giờ thì hết lưỡng lự, chạm mốc thì lùa thẳng vào chuồng).
- **survive**: mỗi lần loại bốc đều 1 con còn sống, nên ai cũng 1/n.
- **egg**: trứng nứt dần, mẹ ấp quả nào thì quả đó ấm hơn.
- **pass** (hiển thị: "Vòng quay may mắn", trước đây là "Tranh Pass"): nhấn giữ lệnh bài lấy đà. Người thắng **bốc lúc thả tay**; lực chỉ quyết định số vòng (nhẹ 7–9, mạnh nhất 15–17). Dưới 15 người: ghế quanh sân; 15–40 người: vòng chia lát. Tên giải tuỳ chỉnh (mặc định "Battle Pass"), có tuỳ chọn né người vừa thắng.

- **cup** (Giải đấu): không có thú chạy, chỉ là bảng đấu. Màn bảng đấu thêm class `wide` trên body để dùng hết bề ngang, có nút toàn màn hình (Fullscreen API, iPhone không có thì ẩn nút), phóng to/thu nhỏ (`dua-ngong-cupzoom`), hoàn tác, sao chép.
  - Thể thức: `double` (nhánh thắng/thua, mặc định), `single` (loại trực tiếp), `rr` (vòng tròn, tối đa 16 đội, thắng 3 hoà 1).
  - Tạo đội: `solo` (mỗi dòng 1 đội), `random` (ghép ngẫu nhiên đội 2–5 người, dư người thì vài đội hơn 1), `preset` (dòng `Tên đội: người 1, người 2`). Chia team xong có nút tạo bảng đấu từ các đội đó.
  - `buildCup()` dựng danh sách trận; mỗi ô có nguồn `{team}`, `{bye}`, `{win:id}`, `{lose:id}`. Hạt giống xếp theo `seedOrder` (1 gặp hạt cuối), số đội không tròn 2^k thì có miễn đấu. Nhánh thua đảo thứ tự người rớt xen kẽ cho khỏi gặp lại sớm. Chung kết tổng có trận "chung kết lại" chỉ đá khi đội từ nhánh thua thắng.
  - Kết quả lưu `C.res[id] = [đội thắng hoặc "D", đội ô trên, đội ô dưới]`. `cupCompute()` tính lại toàn bộ mỗi lần; trận nào đổi người (dù đội thắng cũ vẫn còn trong trận) thì kết quả cũ tự bị xoá và báo "đã xoá kết quả N trận sau".
  - Mỗi ô có đội trong trận đã đủ 2 đội đều có nút **Thắng** (`.wbtn`). Bấm là ghi kết quả ngay; thông báo ghi rõ đội thắng vào trận nào, đội thua xuống nhánh thua trận nào hay bị loại (`nextMatch()`, đi xuyên qua trận miễn đấu).
  - Nhập kết quả: kéo thẻ đội (pointer events, thẻ có `touch-action:none`) thả vào ô ✅ Thắng / ❌ Thua trên thanh dưới, hoặc thả vào ô của trận kế tiếp; kéo sát mép thì tự cuộn. Chạm/bấm thẻ thì hiện thanh để chọn.
  - Giải đang dở lưu ở `dua-ngong-cup` (không lưu danh sách trận, mở lại thì dựng lại). Có vô địch thì ghi vào lịch sử.
## Danh sách tên
- Mọi mode: 2–40 tên (Giải đấu: 64 dòng). Quá giới hạn thì báo lỗi và không cho chơi (không cắt bớt).
- Tên trùng (không phân biệt hoa thường, khoảng trắng) bị chặn trước khi chơi, vì cặp dính/cấm, "Loại, chơi tiếp" và "Né người vừa thắng" đều so theo tên. Cảnh báo hiện ngay dưới ô nhập (`listProblem()`).

## Random
- Mỗi ván gọi `freshSeed()`: lấy 128 bit từ `crypto.getRandomValues`, nạp vào PRNG `sfc32`. Mọi lượt bốc dùng `rnd()`.
- Không hiển thị seed (người dùng không muốn).
- Mô phỏng 20.000 ván Đua Ngỗng 10 con: mỗi làn thắng 9,7–10,4%. Mỗi con có "tốc độ gốc" lệch ±4% (`base`), con có base cao nhất thắng ~17,6%. Có thể bỏ độ lệch này nếu muốn kết quả do diễn biến quyết định hoàn toàn.

## Sprite
- Nguồn: Duckhive trên itch.io (ngỗng, thỏ, sóc: CC0; cánh cụt: tác giả cho dùng tự do). Đã bỏ bò và ếch vì giống asset của game khác.
- Sheet 264x432, mỗi ô 40x32 với viền trống 2px (bước 44x36) để chống lem pixel khi phóng to. Mọi con đã lật cho quay mặt sang phải.
- Hàng: 0–3 ngỗng (idle, walk, run, flap), 4–5 thỏ, 6–7 sóc, 8–11 cánh cụt (idle, walk, flap, roll). Bảng ánh xạ ở `ANIMS`; `flap` = bứt tốc, `cheer` = ăn mừng.
- Tô màu: `tintedSheet(hex)` thay đúng các màu gốc trong `TINT_RULES` (thân/lông), giữ viền, mỏ, chân.

## Quy ước
- Giao diện và thoại bằng tiếng Việt. Tiêu đề trên thanh trên và tab luôn là "Lunar Capital Tool" (cỡ chữ co theo màn hình để không bị cắt ở 360px). Lời dẫn và nút vẫn đổi theo đàn (ngỗng/thỏ/sóc/cánh cụt/thú).
- Mọi thứ lưu localStorage với tiền tố `dua-ngong-`.
- Mỗi ván tăng biến `session`; vòng lặp và setTimeout cũ tự dừng khi `session` đổi.

## Kiểm tra sau khi sửa
- Tách phần `<script>` ra rồi `node --check` để bắt lỗi cú pháp.
- Chạy thử từng mode bằng Playwright (viewport 390x844) và chụp màn hình. Máy này không cài Playwright: import từ bản cache `~/.npm/_npx/*/node_modules/playwright/index.mjs` và truyền `executablePath` tới Chromium có sẵn trong `~/Library/Caches/ms-playwright/chromium_headless_shell-*/` (bản đúng phiên bản có thể chưa tải).

## Deploy
- Repo public `huytran19/dua-ngong`, GitHub Pages từ branch `main`, thư mục root: https://huytran19.github.io/dua-ngong/
- Push lên `main` là Pages tự build lại (khoảng 1 phút).
- Link riêng từng mode: `/giveaway/` (Đua Ngỗng), `/vong-quay/` (Vòng quay may mắn), `/chia-team/`, `/sinh-ton/`, `/trung-no/`, `/giai-dau/`. `/tranh-pass/` là link cũ, giữ lại để link đã gửi vẫn chạy. Mỗi thư mục chỉ có một `index.html` nhỏ chứa thẻ Open Graph (thẻ xem trước khi dán vào Discord) rồi chuyển về `../?mode=<mode>`. Game đọc `?mode=` lúc mở và cập nhật lại URL khi đổi mode.
- Ảnh xem trước 1200x630 ở `og/` (`home.jpg` cho trang chính, `<mode>.jpg` cho từng link). Tạo bằng Playwright: chụp khu vực chơi giữa ván rồi ghép với tiêu đề. Đổi giao diện nhiều thì chụp lại.
- Discord lưu thẻ xem trước khá lâu, sửa ảnh xong có thể phải chờ hoặc thêm `?v=2` vào link.

## Việc tiếp theo đã bàn
- Nút gửi kết quả lên Discord qua Webhook (URL webhook chỉ lưu localStorage trên máy host, không ghi vào code). Chỉ chạy được trên bản GitHub Pages.
- Có thể làm sau: Discord Activity, vé nhiều suất.

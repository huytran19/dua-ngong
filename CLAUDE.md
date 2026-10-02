# Đua Thú (Đua Ngỗng)

Game bốc thăm/giveaway chạy trên trình duyệt, dùng cho guild. Toàn bộ nằm trong **một file `index.html`** (HTML + CSS + JS + sprite nhúng base64), không cần build, không phụ thuộc thư viện. Mở trực tiếp bằng trình duyệt là chạy.

## Cấu trúc trong index.html
- `<style>`: CSS. Màu theo token trên `:root`, có dark mode qua `prefers-color-scheme` và `data-theme`.
- `<script>`: một IIFE, chia theo khối comment:
  - **core**: hằng số, RNG, sprite engine, tô màu, thoại (`LINES`, `line()`, `say()`), âm thanh (`honk`, `blip`), `countdown`, `loop`.
  - **Setup**: `MODE_INFO`, tên gọi theo đàn thú (`noun()`, `Noun()`), các tuỳ chọn, cặp dính/cấm, nút xáo tên, `go()`, `solveTeams()`.
  - **Lịch sử các ván**: lưu localStorage `dua-ngong-history`, tối đa 100 ván.
  - **Result screen**: `showResult(cfg)` dùng chung cho mọi mode.
  - **Mode: Giveaway**, **Chia team**, **Sinh tồn**, **Trứng nở**, **Tranh Pass** (`startRing`).

## 5 mode
- **give**: đua ngang, camera bám con dẫn đầu, 3 độ dài 15/30/50s.
- **team**: 2/3/4 đội đều sĩ số; cặp "dính" (union-find) và "không chung chuồng"; giải bằng quay lui có trọng số trong `solveTeams`. Tốc độ Nhanh/Vừa/Lề mề có giờ tối đa 15/30/60s (gần hết giờ thì hết lưỡng lự, chạm mốc thì lùa thẳng vào chuồng).
- **survive**: mỗi lần loại bốc đều 1 con còn sống, nên ai cũng 1/n.
- **egg**: trứng nứt dần, mẹ ấp quả nào thì quả đó ấm hơn.
- **pass**: nhấn giữ lệnh bài lấy đà. Người thắng **bốc lúc thả tay**; lực chỉ quyết định số vòng (nhẹ 7–9, mạnh nhất 15–17). Dưới 15 người: ghế quanh sân; 15–40 người: vòng chia lát. Tên giải tuỳ chỉnh (mặc định "Battle Pass"), có tuỳ chọn né người vừa thắng.

## Danh sách tên
- Mọi mode: 2–40 tên. Quá 40 thì báo lỗi và không cho chơi (không cắt bớt).
- Tên trùng (không phân biệt hoa thường, khoảng trắng) bị chặn trước khi chơi, vì cặp dính/cấm, "Loại, chơi tiếp" và "Né người vừa thắng" đều so theo tên. Cảnh báo hiện ngay dưới ô nhập (`listProblem()`).

## Random
- Mỗi ván gọi `freshSeed()`: lấy 128 bit từ `crypto.getRandomValues`, nạp vào PRNG `sfc32`. Mọi lượt bốc dùng `rnd()`.
- Không hiển thị seed (người dùng không muốn).
- Mô phỏng 20.000 ván Giveaway 10 con: mỗi làn thắng 9,7–10,4%. Mỗi con có "tốc độ gốc" lệch ±4% (`base`), con có base cao nhất thắng ~17,6%. Có thể bỏ độ lệch này nếu muốn kết quả do diễn biến quyết định hoàn toàn.

## Sprite
- Nguồn: Duckhive trên itch.io (ngỗng, thỏ, sóc: CC0; cánh cụt: tác giả cho dùng tự do). Đã bỏ bò và ếch vì giống asset của game khác.
- Sheet 264x432, mỗi ô 40x32 với viền trống 2px (bước 44x36) để chống lem pixel khi phóng to. Mọi con đã lật cho quay mặt sang phải.
- Hàng: 0–3 ngỗng (idle, walk, run, flap), 4–5 thỏ, 6–7 sóc, 8–11 cánh cụt (idle, walk, flap, roll). Bảng ánh xạ ở `ANIMS`; `flap` = bứt tốc, `cheer` = ăn mừng.
- Tô màu: `tintedSheet(hex)` thay đúng các màu gốc trong `TINT_RULES` (thân/lông), giữ viền, mỏ, chân.

## Quy ước
- Giao diện và thoại bằng tiếng Việt. Tên tiêu đề đổi theo đàn: Đua Ngỗng/Thỏ/Sóc/Cánh Cụt, trộn thì "Đua Thú".
- Mọi thứ lưu localStorage với tiền tố `dua-ngong-`.
- Mỗi ván tăng biến `session`; vòng lặp và setTimeout cũ tự dừng khi `session` đổi.

## Kiểm tra sau khi sửa
- Tách phần `<script>` ra rồi `node --check` để bắt lỗi cú pháp.
- Chạy thử từng mode bằng Playwright (viewport 390x844) và chụp màn hình. Máy này không cài Playwright: import từ bản cache `~/.npm/_npx/*/node_modules/playwright/index.mjs` và truyền `executablePath` tới Chromium có sẵn trong `~/Library/Caches/ms-playwright/chromium_headless_shell-*/` (bản đúng phiên bản có thể chưa tải).

## Deploy
- Repo public `huytran19/dua-ngong`, GitHub Pages từ branch `main`, thư mục root: https://huytran19.github.io/dua-ngong/
- Push lên `main` là Pages tự build lại (khoảng 1 phút).

## Việc tiếp theo đã bàn
- Nút gửi kết quả lên Discord qua Webhook (URL webhook chỉ lưu localStorage trên máy host, không ghi vào code). Chỉ chạy được trên bản GitHub Pages.
- Có thể làm sau: Discord Activity, vé nhiều suất.

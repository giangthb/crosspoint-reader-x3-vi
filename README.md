# CrossPoint Reader — Xteink X3 (Vietnamese)

Firmware CrossPoint cho máy đọc sách **Xteink X3** với hỗ trợ tiếng Việt đầy đủ (UI + font).

> ⚠️ **Bản không chính thức.** Flash tự chịu rủi ro. Hãy backup partition stock trước khi flash.

## Điểm đặc biệt

- ✅ **UI tiếng Việt** — 292 chuỗi đã dịch (menu, settings, dialog, thông báo)
- ✅ **Font UbuntuVietHoa** — hiển thị đủ dấu tiếng Việt ngay trên giao diện (không cần copy font vào SD)
- ✅ **X3 native support** — dựa trên upstream `crosspoint-reader` bao gồm các fix mới nhất cho X3:
  - Grayscale antialiasing cải thiện ([#1607](https://github.com/crosspoint-reader/crosspoint-reader/pull/1607))
  - EPUB images render đúng ([#1572](https://github.com/crosspoint-reader/crosspoint-reader/pull/1572))
  - Sleep screen dimensions X3 ([#1688](https://github.com/crosspoint-reader/crosspoint-reader/pull/1688))
  - Hyphenation EPUB theo ISO 639-2 language code
- ✅ **Đa ngôn ngữ** — ngoài tiếng Việt vẫn giữ 19 ngôn ngữ khác của upstream (EN, ES, FR, DE, IT, PT, RU, UK, PL, ...)
- ✅ **Base on stable release 1.2.0** — không phải dev build experimental
- ✅ **Patch riêng:** fix sleep screen không fill đủ màn hình X3 khi dùng ảnh kích thước X4 (xem [Patches](#patches-riêng-của-fork-này))

## Cách flash

### Cách 1 — Web flasher (dễ nhất, khuyên dùng)

1. **Backup trước:** Vào [https://x3.crosspointreader.com/](https://x3.crosspointreader.com/), dùng phần "Backup" để lưu partition stock về máy (để rollback nếu cần).
2. Kết nối X3 với máy tính qua USB-C, bật máy.
3. Trình duyệt: **Chrome / Edge** (Firefox không hỗ trợ WebSerial).
4. Tải `firmware.bin` từ [Releases](../../releases/latest).
5. Vào [https://x3.crosspointreader.com/](https://x3.crosspointreader.com/), chọn **"Flash custom firmware"**, chọn file `firmware.bin` vừa tải, flash.
6. Rút USB và cắm lại để restart.

### Cách 2 — esptool (manual)

```bash
pip install esptool
esptool.py --chip esp32c3 --port /dev/ttyACM0 --baud 921600 \
    write_flash 0x10000 firmware.bin
```

Thay `/dev/ttyACM0` bằng device của máy bạn.

## Sau khi flash

1. Boot lên, vào **Settings → Language**, chọn **Tiếng Việt**.
2. (Khuyến nghị) Muốn đọc sách tiếng Việt đẹp hơn? Nạp thêm font Unicode có dấu vào SD:
   - Vào [xteink.lakafior.com](https://xteink.lakafior.com/), upload font .ttf/.otf (VD: Be Vietnam Pro, iCiel, Lexend Deca).
   - Click **Convert to .BIN**, rename theo format `FontName_size_WxH.bin` (VD: `Lexend_38_33x39.bin`).
   - Copy vào SD card thư mục `/fonts/`, reboot, chọn trong Settings → Font.

## Rollback về firmware stock

Vào [https://x3.crosspointreader.com/](https://x3.crosspointreader.com/), chọn **"Flash stock firmware"**.
Hoặc swap boot partition ở [https://x3.crosspointreader.com/debug](https://x3.crosspointreader.com/debug).

## Nguồn gốc & Credits

Fork này tổng hợp:

- **Base:** [`crosspoint-reader/crosspoint-reader`](https://github.com/crosspoint-reader/crosspoint-reader) bởi [@daveallie](https://github.com/daveallie) — firmware CrossPoint gốc (MIT License, commit [`e8645ed`](https://github.com/crosspoint-reader/crosspoint-reader/commit/e8645ed))
- **Vietnamese i18n:** cherry-pick từ [`danoooob/crosspoint-reader-vi`](https://github.com/danoooob/crosspoint-reader-vi) bởi [@danoooob](https://github.com/danoooob) — bản dịch 292 chuỗi
- **Font UbuntuVietHoa:** HoangDesign / DesignerViet ([designerviet.com](https://designerviet.com)) — font Ubuntu Việt hóa

Khác biệt so với `danoooob/crosspoint-reader-vi`:

- Target **Xteink X3** (danoooob build cho X4)
- Dựa trên commit upstream mới hơn (bao gồm X3 support + fix release 1.2.0)
- Giữ đầy đủ 19 ngôn ngữ upstream (danoooob xoá Spanish để tiết kiệm flash)
- Có thêm patch sleep-screen crop fix chưa có trong upstream (xem bên dưới)

## Patches riêng của fork này

Những fix tôi tự phát hiện và vá thêm so với upstream `crosspoint-reader`. Đã submit upstream PR để các fork khác cùng hưởng.

### Fix: Sleep screen không fill đủ màn hình trên X3

**Hiện tượng:** Đặt `sleep.bmp` kích thước 480×800 (chuẩn X4) trên X3 (528×792) ở chế độ `Cover Mode: CROP` → ảnh bị render ở góc trái trên với 1:1 pixel, bên phải trống 48px, bên dưới trống 72px. Mặc định lẽ ra phải scale lên 1.1× và crop vừa màn hình.

**Nguyên nhân (3 bugs cộng dồn trong render path):**

1. `SleepActivity::renderBitmapSleepScreen()` chỉ vào branch scale/crop khi bitmap **lớn hơn** màn hình ở ít nhất một chiều. Ảnh 480×800 nhỏ hơn 528×792 ở chiều ngang → rơi vào branch "center only" không hề scale.
2. `GfxRenderer::drawBitmap()` chỉ áp dụng scaling khi `fitScale < 1.0` (chỉ downscale). Yêu cầu upscale bị bỏ qua âm thầm.
3. Render loop iterate source pixels và map 1:1 sang dest pixel. Khi upscale, loop này để lại dòng/cột trống giữa các source row/col (nearest-neighbor artifact).

**Fix:**

- Trigger scale/crop khi `bitmap size != screen size` (không chỉ khi lớn hơn).
- Thêm flag `allowUpscale` vào `drawBitmap()` (default `false`). Chỉ `SleepActivity` opt-in. BmpViewer, cover thumbnails, Lyra themes giữ nguyên hành vi 1:1 centering như master.
- Bypass 1-bit fast path `drawBitmap1Bit()` khi `allowUpscale=true` → ảnh 1-bit cũng được upscale đúng.
- Thay render loop 1:1 bằng **dest-span fill** (nearest-neighbor block): mỗi source pixel ghi ra span `[floor(idx*scale), floor((idx+1)*scale) - 1]`. Khi `scale ≤ 1.0` span = 1, nên hành vi downscale / no-scale không đổi, bit-for-bit giống master.

**Test case:** ảnh X4 480×800 → X3 528×792 CROP mode → giờ render full 528×792, không còn viền trống (đúng cả 2-bit greyscale lẫn 1-bit monochrome).

> Behavior change nhỏ: user có `/sleep/*.bmp` nhỏ hơn panel (trước đây render 1:1 centered) giờ sẽ thấy ảnh bị scale lên fill panel ở CONTAIN/CROP. Đúng semantics của 2 mode đó, nhưng khác hành vi cũ.

Commits: [`10aeb6f`](../../commit/10aeb6f) + [`1740fdc`](../../commit/1740fdc) + [`7f03723`](../../commit/7f03723) • Files: `lib/GfxRenderer/GfxRenderer.{h,cpp}`, `src/activities/boot_sleep/SleepActivity.cpp`.

Upstream PR: [crosspoint-reader/crosspoint-reader#1716](https://github.com/crosspoint-reader/crosspoint-reader/pull/1716) — đã address 2 rounds review của CodeRabbit, chờ maintainer.

## Build từ source

Yêu cầu: Python 3.8+, PlatformIO Core, USB-C cable.

```bash
git clone --recursive https://github.com/<your-user>/crosspoint-reader-x3-vi.git
cd crosspoint-reader-x3-vi
git submodule update --init --recursive
pio run -e gh_release
# Output: .pio/build/gh_release/firmware.bin
```

## License

MIT License — kế thừa từ upstream `crosspoint-reader/crosspoint-reader`. Xem [LICENSE](./LICENSE).

Font UbuntuVietHoa thuộc về HoangDesign / DesignerViet, phân phối theo tinh thần cộng đồng. Font Ubuntu gốc: Ubuntu Font License.

## Disclaimer

Project này **không liên kết với Xteink**. Firmware được build bởi cộng đồng, không có bảo hành. Flash sai có thể brick thiết bị. Luôn backup partition trước khi thử firmware mới.

---

Góp ý / issue: mở [issue](../../issues) hoặc PR.

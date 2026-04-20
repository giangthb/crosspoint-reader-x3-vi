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

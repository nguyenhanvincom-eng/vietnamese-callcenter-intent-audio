# Vietnamese Callcenter Intent Audio

Bộ dữ liệu audio tiếng Việt cho bài toán phân loại **ý định cuộc gọi tổng đài**, gồm 4 nhãn:
`Hỏi thông tin`, `Khiếu nại`, `Mua hàng`, `Khác`, `Đặt lịch hẹn`, `Hỗ trợ kỹ thuật`.

## Thông số

| | |
|---|---|
| Số file | **5.578** |
| Số nhãn | **6** — 4 nhãn gốc mỗi nhãn **1.164** file, 2 nhãn mới ~**445** file |
| Số giọng đọc | **18** (16 Viettel AI + 2 Microsoft Edge) |
| Định dạng | WAV, mono, **8000 Hz**, PCM 16-bit |
| Độ dài | 0,38 – 1,73 giây (trung bình 0,70 giây) |
| Dung lượng | ~73 MB |

## Cấu trúc

```
vn_intent_audio/
├── Hỏi thông tin/
│   ├── kw000__hn-quynhanh.wav      ← kw = từ khoá, giọng Viettel
│   ├── ...
│   └── ph000__edge-hoaimy.wav      ← ph = cụm từ, giọng edge-tts
├── Khiếu nại/
├── Mua hàng/
└── Khác/
```

Tên file: `<tầng><số thứ tự>__<mã giọng>.wav`

- **`kw`** — *từ khoá đơn* (`lỗi`, `hư`, `trả hàng`, `mua`, `bao nhiêu`…). 50 từ mỗi nhãn, mỗi từ đọc bởi cả 16 giọng Viettel → 800 file/nhãn.
- **`ph`** — *cụm từ / câu ngắn*, **54 cụm mỗi nhãn**, chia hai đợt:
  - `ph000`–`ph039` — cụm 2–6 từ (`Đơn hàng tới đâu rồi`, `Tôi muốn đặt mua`…), 2 giọng edge-tts → 80 file/nhãn.
  - `ph040`–`ph053` — **bộ cân bằng từ đệm** (thêm 18/08/2026), 16 giọng Viettel → 224 file/nhãn. Xem mục dưới.

Số thứ tự ứng với **vị trí trong danh sách ngữ liệu** ở notebook. Muốn biết file nào là chữ gì, xem file `ngu_lieu_va_audio.xlsx`.

## Bộ cân bằng từ đệm (`ph040`–`ph053`)

Bộ ban đầu có một lỗi thiết kế: **từ đệm phân bố lệch giữa các nhãn**. Đếm trên tập train:
`cái` / `này` / `thêm` / `luôn` / `liền` chỉ xuất hiện ở nhãn *Mua hàng*; `dạ` / `xin` / `thôi` /
`rõ` / `lại` / `bạn` / `em` / `anh` / `chị` chỉ ở *Khác*; `quá` chỉ ở *Khiếu nại*. Đây đều là từ
không mang ý định, nhưng mô hình nghe thấy là đoán được nhãn — tức **học tắt theo từ đệm**
thay vì học lõi ý định. Hệ quả đo được: mô hình dồn dự đoán về *Mua hàng* (đoán 298 lần trên
190 file thật, precision chỉ 0,53).

14 cụm mới mỗi nhãn dùng **cùng một bộ từ đệm cho cả 4 nhãn**, chỉ khác lõi ý định:

| Nhãn | Ví dụ |
|---|---|
| Hỏi thông tin | *Dạ cho em hỏi **cái này*** · *Xin hỏi thông tin sản phẩm* |
| Khiếu nại | ***Dạ cái này** bị lỗi* · *Mua hai **cái** hư một **cái*** |
| Mua hàng | ***Dạ** đặt giùm em **cái này*** · *Lấy một **cái thôi bạn*** |
| Khác | *Tôi gọi nhầm **cái này*** · ***Dạ** không phải hàng của em* |

Sau khi thêm: **không còn từ đệm nào kẹt ở một nhãn duy nhất**.

## Hai nhãn mở rộng (thêm 19/08/2026)

`Đặt lịch hẹn` và `Hỗ trợ kỹ thuật` được thêm sau, với hai khác biệt so với 4 nhãn gốc:

- Chỉ **6 giọng Viettel** đọc thay vì 16: `hn-thanhphuong`, `hn-tienquan`, `hue-maingoc`,
  `hue-baoquoc`, `hcm-phuongly`, `hcm-minhquan` — đúng 1 nam + 1 nữ mỗi miền. Bốc ngẫu
  nhiên nhưng ép điều kiện phải có ít nhất 1 giọng nằm trong tập test và 1 giọng trong
  tập val, nếu không hai nhãn này sẽ không có mẫu nào để chấm điểm.
- Vì vậy chúng **nhỏ hơn 4 nhãn gốc khoảng 2,6 lần**. Khi huấn luyện nên dùng
  `class_weight='balanced'` để mô hình không bỏ rơi nhãn ít mẫu.

Từ vựng hai nhãn này được chọn để **không trùng mục nào** với ngữ liệu 4 nhãn cũ.
Cả 6 nhãn hiện đều có **60 từ khoá**.

## Giọng đọc

| Vùng | Số giọng | Mã |
|---|---|---|
| Miền Bắc | 8 | `hn-quynhanh`, `hn-phuongtrang`, `hn-thaochi`, `hn-thanhha`, `hn-thanhphuong`, `hn-thanhtung`, `hn-namkhanh`, `hn-tienquan` |
| Miền Trung | 2 | `hue-maingoc`, `hue-baoquoc` |
| Miền Nam | 6 | `hcm-diemmy`, `hcm-phuongly`, `hcm-thuydung`, `hcm-thuyduyen`, `hcm-minhquan`, `hn-leyen` |
| Không phân vùng | 2 | `edge-hoaimy`, `edge-namminh` |

Lưu ý: `hn-leyen` có tiền tố `hn-` nhưng theo API của Viettel thì thuộc **miền Nam** — đừng suy vùng miền từ tiền tố mã giọng.

Ngữ liệu có chứa **từ địa phương** theo cặp: Bắc `hỏng`/`vỡ`/`bẩn`/`đắt` ↔ Nam `hư`/`bể`/`dơ`/`mắc`, cùng các từ miền Trung `mô`/`răng`/`rứa`/`chi`. Mọi giọng đều đọc mọi từ, để mô hình học chính cái *từ* chứ không học tắt theo *giọng*.

## Nguồn gốc dữ liệu — đọc kỹ trước khi dùng

Đây **KHÔNG phải giọng người thật**. Toàn bộ audio sinh bằng Text-to-Speech:

- **Viettel AI** (viettelai.vn) — 16 giọng, phần từ khoá
- **Microsoft Edge TTS** — 2 giọng, phần cụm từ

Hệ quả cần biết:

- Không có tiếng ồn nền, không ngập ngừng, không nói lắp, không biến thiên tự nhiên của lời nói thật.
- Chỉ có **18 "người nói"**, đều là giọng tổng hợp.
- Mô hình huấn luyện trên bộ này **có thể nhận kém khi gặp giọng người thật ghi âm qua điện thoại**.

Đây là hạn chế cố hữu của cách sinh dữ liệu này, không phải lỗi có thể sửa bằng code. Muốn dùng thật thì cần thu thêm giọng người thật rồi tinh chỉnh.

## Dùng lại

Tải trực tiếp bằng URL zip:

```python
url = "https://github.com/<user>/<repo>/archive/refs/heads/main.zip"
```

Notebook `capstone_vietnamese_intent.ipynb` có sẵn hàm `download_dataset_from_github()` ở Bước 3A làm việc này — chạy nó là có dữ liệu, **không cần token Viettel, không tốn quota**.

## Giấy phép

Dùng cho mục đích **học tập**. Giọng đọc thuộc bản quyền Viettel AI và Microsoft — kiểm tra điều khoản dịch vụ của họ trước khi dùng cho mục đích khác.

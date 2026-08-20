# Vietnamese Callcenter Intent Audio

Bộ dữ liệu audio tiếng Việt cho bài toán phân loại **ý định cuộc gọi tổng đài**, gồm 4 nhãn:
`Hỏi thông tin`, `Khiếu nại`, `Mua hàng`, `Khác`.

## Thông số

| | |
|---|---|
| Số file | **9.410** |
| Số nhãn | **4** dùng chính thức, mỗi nhãn ~**2.000** file (+2 thư mục để dành, xem dưới) |
| Số giọng đọc | **18** (16 Viettel AI + 2 Microsoft Edge) |
| Định dạng | WAV, mono, **8000 Hz**, PCM 16-bit |
| Độ dài | 0,38 – 1,73 giây (trung bình 0,70 giây) |
| Dung lượng | ~120 MB |

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

## Đợt V2 — chuyển hẳn sang KÊNH BÁN HÀNG BÁN LẺ (19/08/2026)

Bộ gốc xây theo kiểu tổng đài đa ngành: có ngân hàng (`số dư`, `lãi suất`, `mở thẻ`),
viễn thông (`gói cước`, `sim số`, `cáp quang`), du lịch (`đặt vé máy bay`, `đặt phòng`),
nhà hàng (`đặt bàn`), gym (`gói tập một năm`). Với một shop bán lẻ thì lạc đề.

Đợt V2 làm hai việc:

- **Thêm 346 mẫu** phủ đúng nghiệp vụ bán lẻ: sản phẩm (`còn size nào`, `hàng chính hãng`,
  `xuất xứ ở đâu`), đơn hàng (`mã vận đơn`, `đơn tới đâu`), vận chuyển (`phí ship`,
  `có ship tỉnh`, `ship ra bến xe`), thanh toán (`quét mã`, `hoá đơn đỏ`, `ship cod`,
  `chuyển cọc`), khiếu nại bán lẻ (`giao sai hàng`, `gửi nhầm màu`, `không giống hình`,
  `shipper thái độ`, `bóc phốt`).
- **Đánh dấu 62 mẫu lạc ngành để KHÔNG nạp** — file audio vẫn giữ nguyên trong repo,
  danh sách chỉ số nằm ở `LOAI_TRU_IDX` trong notebook. Bỏ tên khỏi đó là dùng lại được.

Sau đợt này mỗi nhãn có **185 mẫu ngữ liệu** (95 từ khoá + 90 cụm từ), cân đều tuyệt đối.
Số giọng: từ khoá **16 giọng Viettel**, cụm từ **4 giọng Viettel + 2 giọng edge-tts**.

Hai thư mục `Đặt lịch hẹn` và `Hỗ trợ kỹ thuật` vẫn còn trong repo nhưng **không thuộc
bài toán hiện tại** — giữ lại phòng khi cần mở rộng nhãn về sau.

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

## Mô hình đã huấn luyện (`mo_hinh/`)

Ba file để **dùng ngay mà không phải huấn luyện lại**:

| File | Là gì |
|---|---|
| `vn_intent_bilstm.keras` | Mô hình BiLSTM đã train (Masking → BiLSTM 64 → Dropout → Dense 32 → softmax) |
| `vn_intent_stats.json` | Thông số tiền xử lý: 4 nhãn, 8000 Hz, 13 hệ số MFCC, 44 khung, mean/std chuẩn hoá |
| `ngu_lieu.json` | 802 câu ngữ liệu, để dựng lại bộ xếp nhãn theo chữ trong ~1 giây |

**Ba file phải đi cùng nhau.** `mean_val`/`std_val` trong file thông số là của đúng lần train
đã sinh ra file `.keras`; lẫn bản cũ với bản mới thì kết quả sai âm thầm, không có lỗi nào báo.

Độ chính xác đo trên chính bộ này (chia theo giọng — 4 giọng bị giấu hoàn toàn khỏi lúc học):

| Cách chấm | Giọng đã học | Giọng lạ | Câu lạ |
|---|---|---|---|
| BiLSTM nghe thẳng âm thanh | ~93% | **~63%** | ~39% |
| Đọc ra chữ rồi xếp nhãn theo chữ | | | **~83%** |

Con số đáng tin cho đời thật là cột **câu lạ**, vì khách gọi thật luôn nói câu chưa từng có
trong ngữ liệu (đoán mò = 25%). BiLSTM học dáng âm của cả đoạn nên gặp câu lạ là đuối;
thêm giọng không cứu được, chỉ thêm CÂU mới cứu được.

## Giấy phép

Dùng cho mục đích **học tập**. Giọng đọc thuộc bản quyền Viettel AI và Microsoft — kiểm tra điều khoản dịch vụ của họ trước khi dùng cho mục đích khác.

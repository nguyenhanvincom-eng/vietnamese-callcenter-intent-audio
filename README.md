# Vietnamese Callcenter Intent Audio

Bộ dữ liệu audio tiếng Việt cho bài toán phân loại **ý định cuộc gọi tổng đài**, gồm 4 nhãn:
`Hỏi thông tin`, `Khiếu nại`, `Mua hàng`, `Khác`.

## Thông số

| | |
|---|---|
| Số file | **3.520** |
| Số nhãn | 4 (mỗi nhãn đúng **880** file — cân bằng tuyệt đối) |
| Số giọng đọc | **18** (16 Viettel AI + 2 Microsoft Edge) |
| Định dạng | WAV, mono, **8000 Hz**, PCM 16-bit |
| Độ dài | 0,38 – 1,73 giây (trung bình 0,70 giây) |
| Dung lượng | ~46 MB |

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
- **`ph`** — *cụm từ 2–6 từ* (`Đơn hàng tới đâu rồi`, `Tôi muốn đặt mua`…). 40 cụm mỗi nhãn, mỗi cụm đọc bởi 2 giọng edge-tts → 80 file/nhãn.

Số thứ tự ứng với **vị trí trong danh sách ngữ liệu** ở notebook. Muốn biết file nào là chữ gì, xem file `ngu_lieu_va_audio.xlsx`.

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

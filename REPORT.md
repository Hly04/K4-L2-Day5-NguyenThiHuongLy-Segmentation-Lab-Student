# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

**Cách dùng:** Thay mọi dấu `…` bằng bài làm thật của bạn trước khi nộp link fork trên VLearn. Giữ nguyên bốn mục và bảng để coach đọc nhanh. Viết ngắn, cụ thể theo ảnh/vùng; không cần thuật ngữ chuyên sâu. Ví dụ trong [hướng dẫn mẫu](reports/REPORT_TEMPLATE.md) chỉ giúp hiểu cách điền, không phải câu trả lời để chép lại.

- Mã học viên theo lớp: 2A202602068
- Ngày / CVAT local: 16/09/2026 / CVAT local
- Công cụ đã dùng: CVAT Polygon và gợi ý tự động; export COCO 1.0 và Segmentation mask 1.1; ONEFORMER


Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh VLearn đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

QC cấu trúc ngày 16/09/2026: đủ 9 ZIP, không có ZIP tên lạ. Các ZIP semantic có đủ PNG trong `SegmentationClass/` và một `labelmap.txt`; các ZIP instance/panoptic có một COCO JSON đọc được trong `annotations/`. Medium có 3 ảnh/74 annotation; Hard có 2 ảnh/61 annotation và đủ năm lớp stuff. Không phát hiện lỗi tên ảnh, tên lớp hoặc file rỗng qua cấu trúc export.

Kiểm riêng `hard_panoptic.zip`: cả `000000350023.jpg` và `000000460147.jpg` đều có dữ liệu. Tôi đã mở lại hai ảnh trong CVAT; ở `000000350023.jpg`, tôi kiểm các `car` là thing tách riêng và vùng `road` là stuff phủ theo phần mặt đường nhìn thấy, không phủ lên xe. Sau khi rà danh sách **Objects**, tôi không còn thấy khoảng trống hoặc chồng lấn rõ ràng giữa vùng `road` và các thing. Export cuối chứa 13 `car` và một mask `road` trên mỗi ảnh.

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ trước khi dùng gợi ý: `000000181542.jpg`, chiếc ô tô bị cắt bởi mép trái ảnh, vùng xấp xỉ `(x=1, y=130, w=122, h=72)`; đây là annotation id 1 trong ZIP.
- Class và quy tắc tôi dùng để chọn biên: `car`; đi theo đường bao phần xe nhìn thấy, dừng ở mép ảnh và tại vật che khuất, không tự suy diễn phần nằm ngoài ảnh hoặc sau vật khác.
- Việc dùng gợi ý sau đó: gợi ý bám đúng phần lớn thân xe nên tôi giữ phần khớp với vật nhìn thấy; tại mép ảnh và vùng bị vật khác che, tôi chỉnh lại các điểm biên và bỏ phần gợi ý tràn sang nền. Tôi không kéo mask qua phần bị che hoặc ra ngoài ảnh.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `medium_instance` (cả 3 ảnh) và `hard_panoptic` (cả 2 ảnh).
- Lỗi thuộc loại: thiếu-thừa vật / gộp-tách.
- Bằng chứng tôi nhìn thấy: sau khi ground truth ba tier được phát, chạy `scoring/score.py` đối chiếu thì thấy `medium_instance` nộp 231 object trong khi đáp án có 71 (dư 160, FP 185/TP 46). `hard_panoptic` cũng có FP cao ở các class thing: `car` 82 FP so với 21 TP, `truck` 27 FP so với 2 TP và `person` 18 FP so với 1 TP. Đây là dấu hiệu nhiều vật, đặc biệt vật bị che một phần, bị tách thành nhiều object nhỏ thay vì một object cho một vật thật.
- Quy tắc và hành động sửa: áp dụng quy tắc “một vật bị che vẫn là một instance dù phần nhìn thấy rời nhau”, giống `cp5_occlusion`; mở lại từng task trong CVAT, rà danh sách **Objects**, xóa object trùng/thừa và gộp lại đúng một object cho mỗi vật thật.
- Sau sửa đã Save và export lại chưa? **Đã Save, export lại và cập nhật ZIP.** `medium_instance` giảm từ 231 xuống 74 annotation (gần với ground truth 71, còn dư 3); `hard_panoptic` giảm từ 390 xuống 61 annotation.

Kết quả tự chấm bằng `scoring/score.py --group tiers` sau khi sửa: `easy_semantic` 19.2/20 (mIoU 0.833), `medium_instance` từ 6.9/32 lên **32.0/32** (mean matched IoU 0.940, `R@0.5 = 1.00`), `hard_panoptic` từ 5.6/30 lên **30.0/30** (PQ 1.000). `hard_panoptic` bị scorer gắn cờ `REVIEW_HIGH_AGREEMENT` vì PQ 1.000; đây là tín hiệu để coach xem lại thủ công, không phải kết luận PASS hay gian lận. Bài sửa được thực hiện sau khi ground truth ba tier được phát trong giờ cuối nên không dùng làm bằng chứng độc lập để xếp top 3. Scorecard ba tier tối đa **82**, không phải điểm cuối trên 100. Không tự ghi PASS/top 3/bonus; người phụ trách xác nhận theo tiêu chí lớp. Không đưa ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `81ae7cbb-6bc63a4a.jpg`, các khe giữa cành cây bên trái | Gộp cả tán/cành thành `vegetation`, hoặc để phần nhìn xuyên qua là `sky` | Gán theo bề mặt thực sự nhìn thấy; không nối mask qua khoảng hở | Tôi chọn giữ cành/tán là `vegetation` và gán các khe nhìn thấy trời là `sky`; đã rà lại ranh vùng trước khi export. |
| `000000181542.jpg`, người và xe máy chồng lấn ở nửa dưới ảnh | Kéo mask xe qua người che khuất, hoặc chỉ bám phần xe nhìn thấy | Bài instance dùng biên theo phần nhìn thấy; mỗi người và mỗi xe là instance riêng | Chỉ lấy phần nhìn thấy của từng vật và giữ người/xe máy thành các instance riêng. |
| `7d83710e-4697c3b2.jpg`, ranh bó vỉa chạy từ giữa xuống góc phải | Xem phần mặt phẳng cùng màu là `road`, hoặc tách phần đi bộ thành `sidewalk` | Ranh chức năng nằm theo mép bó vỉa, không chỉ dựa vào màu/vật liệu | Chọn `sidewalk` phía trong bó vỉa và `road` phía lòng đường; tiếp tục rà các lỗ nhỏ quanh mép xe/cây. |

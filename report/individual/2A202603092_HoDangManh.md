# Báo cáo thực hành Day 3 — Image Segmentation

Mã học viên theo lớp: 2A202603092  
Họ và tên: Hồ Đăng Mạnh  
Ngày thực hiện: 11/11/2026  
Môi trường thực hiện: CVAT Local (`http://localhost:8080`)  
Công cụ đã sử dụng: CVAT, Polygon, Brush và AI/SAM hỗ trợ gán nhãn.

## 1. Bài đã nộp

Tôi đã thực hiện gán nhãn cho 14 ảnh thuộc 9 task theo yêu cầu của bài Day 03. Sau khi hoàn thành, tôi kiểm tra, lưu dữ liệu và xuất các file ZIP tương ứng.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
|---|---|---:|---:|
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | **14 / 14** | **100** |

Các file ZIP được xuất từ CVAT và lưu trong thư mục `submissions/`. Tôi giữ nguyên cấu trúc bên trong các file ZIP sau khi xuất.

## 2. Một quyết định trước khi dùng gợi ý
Ảnh, vị trí và object Medium đầu tiên tự vẽ:
Trong task `medium_instance`, tôi tự vẽ mask đầu tiên tại ảnh `000000181542.jpg`, Frame 0, Object ID 1.
Vật thể là một chiếc xe bán tải màu xám, nằm gần giữa ảnh, hơi lệch sang phải và ở phía sau người đi bộ.

Class và quy tắc tôi dùng để chọn biên:
Tôi chọn class `car` và tự vẽ mask bằng tay trước khi sử dụng gợi ý AI/SAM.
Khi vẽ, tôi xác định đường viền theo phần chiếc xe thực sự nhìn thấy trong ảnh. Tôi cố gắng bám sát mép xe, không tô sang người đi bộ và không tự suy đoán những phần xe bị che khuất.
Tôi lựa chọn cách này vì mục tiêu của Instance Segmentation là xác định chính xác vùng thuộc từng vật thể, thay vì chỉ sử dụng khung hình chữ nhật bao quanh.

Việc sử dụng gợi ý AI/SAM sau đó:
Sau khi hoàn thành mask đầu tiên bằng tay, tôi sử dụng AI/SAM để hỗ trợ quá trình gán nhãn các vật thể tiếp theo.
Tôi nhận thấy kết quả gợi ý chưa hoàn toàn chính xác. Đặc biệt, tại Frame 2 của `medium_instance`, AI/SAM bỏ sót một số chiếc ô tô.
Tôi không giữ nguyên toàn bộ kết quả gợi ý mà kiểm tra lại ảnh gốc, xác định các vật thể bị thiếu và tự vẽ bổ sung mask riêng cho từng chiếc xe.
Tôi rút ra rằng AI/SAM giúp hỗ trợ quá trình gán nhãn nhưng vẫn cần người thực hiện kiểm tra và chỉnh sửa kết quả.

## 3. Một lỗi tôi tìm thấy và sửa

Task/ảnh/vùng:
`medium_instance` – Frame 2 (ảnh thứ ba), tại các vị trí có ô tô bị AI/SAM bỏ sót.

Lỗi thuộc loại:
Thiếu vật thể (Missing Objects).

Bằng chứng tôi nhìn thấy:
Khi quan sát ảnh gốc và đối chiếu với danh sách object cùng các vùng mask đã được tạo trong CVAT, tôi phát hiện một số chiếc ô tô xuất hiện trong ảnh nhưng chưa được AI/SAM tạo mask tương ứng.
Điều này khiến kết quả gán nhãn chưa bao phủ đầy đủ các vật thể cần nhận diện.

Quy tắc và hành động sửa:
Tôi kiểm tra lại ảnh, xác định từng chiếc ô tô bị bỏ sót và sử dụng công cụ gán nhãn thủ công trong CVAT để bổ sung.
Với mỗi chiếc xe, tôi tạo một object riêng, chọn class `car` và vẽ mask bám sát phần xe thực sự nhìn thấy.
Tôi không gộp nhiều chiếc ô tô thành một mask, kể cả khi chúng đứng gần nhau.
Sau khi bổ sung, tôi kiểm tra lại danh sách object và các vùng đã được gán nhãn.

Sau sửa đã Save và export lại chưa?
Tôi đã lưu kết quả và xuất dữ liệu hoàn chỉnh của task `medium_instance` thành file `medium_instance.zip` theo định dạng COCO 1.0.
**Kết quả tự đánh giá trên GitHub Actions hoặc script:**
Chưa có điểm tự đánh giá được xác nhận. Tôi không tự xác định kết quả PASS hoặc điểm số chính thức.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
|---|---|---|---|
| 1. `cp2_slice` – hai chiếc ô tô đứng sát nhau** | Cách 1: Gộp hai xe thành một mask vì chúng nằm sát nhau. Cách 2: Tách thành hai mask riêng vì đây là hai chiếc xe độc lập. | Trong Instance Segmentation, mỗi vật thể đếm được phải có một instance riêng, dù thuộc cùng một class. | Tôi quyết định tách thành hai mask riêng, cùng nhãn `car`, và xác định ranh giới giữa hai xe. |
| 2. `cp4_curb` – ranh giới mặt đường và vỉa hè** | Cách 1: Xem vùng ranh giới là một phần của `road`. Cách 2: Xem vùng đó là một phần của `sidewalk`. | Màu sắc giữa mặt đường và vỉa hè có thể tương đồng. Cần dựa vào cấu trúc, vị trí và chức năng của từng khu vực để xác định ranh giới. | Tôi quan sát ranh giới thực tế, sau đó tách vùng mặt đường và vỉa hè để gán đúng nhãn `road` và `sidewalk`. |
| 3. `cp3_thin` – các chi tiết mảnh của vật thể** | Cách 1: Bỏ qua những phần quá mảnh vì khó vẽ chính xác. Cách 2: Giữ lại các chi tiết mảnh vẫn nhìn thấy rõ trong ảnh. | Mask cần bao phủ các phần thực sự nhìn thấy của lớp cần gán nhãn, đồng thời hạn chế tô tràn sang vùng nền. | Tôi quyết định giữ lại các chi tiết mảnh nhìn thấy và sử dụng Brush kích thước nhỏ để tô chính xác hơn. |

Kết luận:

Qua bài thực hành Day 03, tôi đã thực hiện ba hình thức gán nhãn gồm Semantic Segmentation, Instance Segmentation và Panoptic Segmentation, đồng thời xử lý các trường hợp đặc biệt trong sáu checkpoint.
Tôi nhận thấy việc lựa chọn đúng class, xác định ranh giới mask, tách riêng từng instance và kiểm tra kết quả AI/SAM là những yếu tố quan trọng để bảo đảm chất lượng dữ liệu gán nhãn.
Tôi đã hoàn thành việc gán nhãn và xuất 9 file ZIP theo định dạng yêu cầu, đồng thời ghi nhận các quyết định và lỗi thực tế trong quá trình thực hiện.

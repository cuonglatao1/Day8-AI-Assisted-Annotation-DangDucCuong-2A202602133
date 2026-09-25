# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Đặng Đức Cường

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Camera đặt cố định trên cầu vượt, một chiếc xe nằm trong khung hình vài giây, và hai ảnh cách nhau 0.4 giây gần như giống hệt nhau. Vì vậy tập chưa gán nhãn (pool) và tập kiểm thử (test) được chia theo trục thời gian, có vùng đệm 4 giây ở giữa (pool gần test nhất vẫn cách 4.4 giây). Nếu chia ngẫu nhiên, cùng một chiếc xe có thể vừa nằm trong tập học vừa nằm trong tập chấm. Khi đó mô hình được chấm trên những xe nó đã thấy, nên điểm sẽ cao hơn thực tế (rò rỉ dữ liệu).

## 2. Mô hình khởi đầu lạnh (cold start)

Dòng vòng 0 trong `rounds_table.md`: yolov8n cold start, AP50 0.771, P@0.25 0.925, R@0.25 0.489, F1 0.640, R small 0.182, R medium 0.547, R large 0.561. Recall xe nhỏ chỉ 0.182 so với xe vừa 0.547 và xe lớn 0.561, nghĩa là xe ở xa bị bỏ sót nhiều hơn xe ở gần. Trong `compare_round0.jpg`, frame_0050 có TP 11, FP 2, FN 7, frame_0250 có TP 6, FP 2, FN 9 và frame_0350 có TP 9, FP 2, FN 14. Các khung bỏ sót nằm chủ yếu ở xe nhỏ và tối phía xa. 
Nhãn dùng để chấm do một mô hình tạo ra, chưa có người rà từng khung, nên có thể nhãn tham chiếu sai chứ không phải mô hình của tôi sai. Trước khi kết luận mô hình sai, cần rà lại những khung tham chiếu ở xe rất xa hoặc quá tối.

## 3. Chiến lược chọn mẫu

Mỗi ảnh có điểm `score = W_U·U + W_A·A + W_D·D` với W_U = 0.5, W_A = 0.3, W_D = 0.2. U là độ bất định của mô hình (một nửa điểm), A là phần khung mô hình vẽ nhưng còn lưỡng lự (ba phần mười), D là độ khác thời gian so với ảnh khác (hai phần mười). `MIN_GAP_S = 2` giây nghĩa là hai ảnh trong cùng một lô phải cách nhau ít nhất 2 giây, vì camera đứng yên nên ảnh sát nhau gần như trùng lặp.
Theo `SELECTION.md`, nếu chỉ rà năm ảnh tôi chọn frame_0182 (hạng 1, 0.959), frame_0369 (hạng 2, 0.932), frame_0380, frame_0326 và frame_0331. Trong 12 ảnh mô hình chọn, tôi xem frame_0182, frame_0099 và frame_0107, cả ba có nhiều khung lưỡng lự (n_ambiguous 18, 14 và 15). frame_0372 (hạng 6, 0.910) có điểm cao nhưng bị bỏ vì chỉ cách frame_0369 khoảng 1.2 giây. Về công gán nhãn, frame_0369 cần thêm 18 khung, nhiều nhất trong 12 ảnh, nên rất tốn công.
Điểm bất định cao không chứng minh sửa ảnh đó sẽ làm mô hình tốt hơn, chỉ cho biết mô hình đang phân vân. Kết quả vòng 1 ở mục 4 cũng cho thấy điều này.

## 4. Các vòng học chủ động (active learning)

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 291 | 0.363 | -0.409 | 1.000 | 0.050 | 0.095 | 0.000 | 0.037 | 0.220 |

Vòng 1 (theo `round1_diff.md`): 12 ảnh, mô hình đề xuất 169 khung. Tôi giữ nguyên 150 khung (accept rate 89%), chỉnh sửa 11, xóa 8 và thêm mới 130, còn tổng 291 khung. AP50 giảm 0.408 so với cold start (0.771 xuống 0.363). Đây là vòng đầu nên chưa có vòng trước để so. Cả ba nhóm xe đều xấu đi: xe nhỏ từ 0.182 xuống 0.000, xe vừa từ 0.547 xuống 0.037, xe lớn từ 0.561 xuống 0.220. Precision lên 1.000 nhưng recall chỉ còn 0.050, nghĩa là mô hình sau fine-tune gần như không dám khoanh xe nào ở ngưỡng 0.25 (giải thích có thể, chưa kiểm chứng: fine-tune yolov8n chỉ với 12 ảnh làm độ tin cậy giảm thấp).

Ba việc khác nhau: (1) quan sát độc lập trong `BLIND_SCAN.md`: tôi tự đếm 21 xe ở frame_0099 trước khi xem khung AI, và `round1_diff.md` ghi frame_0099 có 21 khung sau khi sửa. (2) Lỗi pre-label đã sửa trong `REVIEW_LOG.csv`: khung bỏ sót (added), khung lệch hoặc chưa bao hết xe (edited), khung không phải xe (deleted). (3) Kết quả mô hình sau khi học lại: các số ở bảng trên.


## 5. Kết luận và giới hạn

Kết quả vòng 1 kém cold start: AP50 giảm từ 0.771 xuống 0.363. Vì điểm giảm mạnh, tôi chọn **dừng** và kiểm tra lại trước khi cho mô hình học thêm, thay vì rà thêm ảnh. Việc cần kiểm: khung tôi đã sửa có nhất quán không (cách vẽ xe xa, xe bị cắt mép), số ảnh học 12 có quá ít không, và mô hình có bị giảm độ tin cậy sau fine-tune không.
Hai ca còn yếu: (1) xe ở xa chỉ còn hai chấm đèn, R small chỉ còn 0.000. (2) xe bị cắt ở mép ảnh hoặc bị xe khác che, khó vẽ khung nhất quán. Chi phí rà tiếp khá cao: frame_0369 cần thêm 18 khung. Không nên chọn hai ảnh sát nhau như frame_0369 và frame_0372 (cách khoảng 1.2 giây) vì chúng gần như một cảnh.
Giới hạn: tập kiểm thử chỉ có 20 ảnh, các khung tham chiếu cao dưới 16 pixel (14 khung) bị bỏ qua nên xe quá nhỏ không được tính, và nhãn tham chiếu do mô hình tạo, chưa được người kiểm. Vì thế điểm chỉ cho biết mức khớp với bộ tham chiếu này, chưa phải chất lượng tuyệt đối. Nếu AP50 giảm, tôi sẽ xem lại khung đã sửa trước khi train thêm.
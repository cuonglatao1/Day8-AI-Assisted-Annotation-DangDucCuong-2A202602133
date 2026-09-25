# Vì sao chọn lô này?

Nếu chỉ có ngân sách rà năm ảnh, tôi chọn theo thứ tự điểm: frame_0182.jpg (hạng 1, điểm 0.959, giây 72.8), frame_0369.jpg (hạng 2, điểm 0.932, giây 147.6), frame_0380.jpg (hạng 3, điểm 0.917, giây 152.0), frame_0326.jpg (hạng 4, điểm 0.916, giây 130.4) và frame_0331.jpg (hạng 5, điểm 0.915, giây 132.4). Năm ảnh này đứng đầu danh sách. Tôi bỏ frame_0372.jpg (hạng 6, điểm 0.910, giây 148.8) vì nó chỉ cách frame_0369.jpg khoảng 1.2 giây, camera đứng yên nên hai ảnh gần như cùng một cảnh, sửa cả hai tốn công mà học thêm ít.

Trong 12 ảnh model chọn, tôi xem frame_0182.jpg, frame_0099.jpg và frame_0107.jpg. Cả ba đều có nhiều khung chưa chắc: n_ambiguous lần lượt là 18, 14 và 15 trên tổng 28, 29 và 33 khung. Theo round1_diff.md, model chỉ đề xuất 13 khung cho mỗi ảnh, sau khi sửa còn 23, 21 và 24 khung. Nghĩa là model bỏ sót nhiều xe mà điểm bất định chưa cho thấy hết.

frame_0372.jpg hạng 6, điểm 0.910, cao hơn nhiều ảnh được chọn như frame_0099.jpg (hạng 8, 0.906), nhưng model không chọn vì sát giờ với frame_0369.jpg. Ngược lại, frame_0392.jpg hạng 15 (điểm 0.887, thấp hơn nhiều ảnh khác) vẫn được chọn vì có U cao nhất (0.975), tức model rất phân vân.

Điểm cao chỉ nghĩa là model đang phân vân, chưa chứng minh sửa ảnh đó xong thì model nhận xe tốt hơn. Kết quả vòng 1 còn cho thấy điểm AP50 giảm từ 0.771 xuống 0.363, nên chọn ảnh theo độ bất định không bảo đảm làm model khá hơn.
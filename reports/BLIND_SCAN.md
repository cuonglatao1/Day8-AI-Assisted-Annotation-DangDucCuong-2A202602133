# Quét độc lập trước khi xem pre-label

Frame:  frame_0099.jpg trong `to_label/round1/images/train/`

Số xe nhìn thấy bằng mắt: 21

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe: Bên phải ảnh, phía trên xe xám, có hai xe rất tối gần như hòa vào nền đêm, chỉ còn thấy đèn hậu đỏ mờ; độ tương phản thấp nên AI dễ bỏ sót.

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.

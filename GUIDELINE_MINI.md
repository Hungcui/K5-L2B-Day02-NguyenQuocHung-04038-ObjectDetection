# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con, sedan, hatchback, SUV, CUV, taxi, xe bán tải (pickup cabin nhỏ). | Xe chở người cỡ nhỏ/gia đình, xe bán tải cabin đôi. Phân biệt với van bằng chiều cao và thiết kế đầu xe. |
| `truck` | Xe tải chở hàng, xe ben, container, xe đầu kéo, xe chuyên dụng chở nặng. | Cabin chở hàng lớn tách rời hoặc xe tải chở hàng hóa nặng. Phân biệt với pickup bằng kích thước khung gầm và tải trọng. |
| `bus` | Xe buýt công cộng, xe khách lớn trên 16 chỗ. | Thân xe dài, nhiều hàng ghế chở khách thương mại công cộng. Phân biệt với van bằng chiều dài và sức chứa lớn. |
| `van` | Xe van, minivan, MPV chở hàng hoặc chở người tầm trung (7-16 chỗ). | Thân xe liền một khối (monocoque), cao hơn xe con, dạng hộp vuông vức (như Ford Transit, Toyota HiAce, Solati). |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Ôm khít phần thân xe nhìn thấy được; không lấy thừa mặt đường, bóng đổ, bầu trời hoặc xe bên cạnh.
- Vật thể bị che khuất: Gán box quanh phần nhìn thấy được nếu nhận diện chắc chắn được lớp; nếu bị che khuất quá lớn (>80%) và không rõ hình dáng thì không đoán.
- Vật thể bị cắt bởi rìa ảnh: Box chạm sát đúng rìa ảnh, không tự vẽ suy đoán phần nằm ngoài màn hình.
- Vật thể rất nhỏ/mơ hồ: Chỉ gán khi đủ độ phân giải nhận diện được đặc trưng của lớp; không gán các đốm mờ ở hậu cảnh quá xa.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Xe bán tải di chuyển trong làn và các xe ở khu vực xa ngoài luồng giao thông | Gán xe bán tải là `car (0)`; không gán xe ngoài luồng chính | Schema quy định xe bán tải cabin nhỏ thuộc `car`; các xe nằm ngoài luồng giám sát không gán để tránh FP |
| `drive_002` | Xe dạng minivan/van chuyển phát nhanh có kích thước nhỉnh hơn SUV | Gán `van (3)` | Dáng xe hình hộp vuông vức, nóc cao, thân liền khối phục vụ chở hàng/người cỡ vừa |
| `drive_005` | Xe khách cỡ nhỏ/minibus ở ngã tư giao nhau | Gán `bus (2)` | Chiều dài thân xe lớn và phục vụ vận tải hành khách công cộng, không gán vào van cá nhân |

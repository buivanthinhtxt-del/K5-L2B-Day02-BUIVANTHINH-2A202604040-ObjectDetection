# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con nhỏ 4 bánh chở người cá nhân/gia đình (4–7 chỗ), bao gồm sedan, hatchback, SUV, crossover, taxi và xe bán tải (pickup) cabin nhỏ. | Có nắp capo trước rõ ràng, thân thấp hoặc dáng SUV/sedan; phân biệt với `van` ở chỗ `van` có thân hình hộp cao liền khối và đầu xe ngắn/phẳng hơn. |
| `truck` | Xe tải hàng cỡ lớn hoặc chuyên dụng chở hàng hóa/vật liệu, xe đầu kéo, xe tải thùng kín (box truck) hoặc xe tải ben. | Cấu trúc tách biệt rõ giữa cabin lái phía trước và thùng hàng/rơ-moóc phía sau; không gộp xe bán tải cá nhân (thuộc `car`). |
| `bus` | Xe chở người số lượng lớn (xe buýt công cộng, minibus chở khách, xe buýt khớp nối 2 khoang, xe khách). | Thân xe dài, đầu và đuôi vuông vức, có dãy cửa sổ kính lớn liên tiếp dọc hai bên hông và bảng điện tử tuyến đường phía trước/sau. |
| `van` | Xe van/minivan hình hộp liền khối (1 khoang liền từ cabin đến khoang sau) dùng chở người hoặc chở hàng nhẹ, thường có cửa trượt bên hông. | Đầu xe ngắn/dốc liền với thân hình hộp chữ nhật, cao hơn sedan nhưng ngắn và nhỏ hơn nhiều so với `bus`; không có thùng hàng tách rời như `truck`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh của bounding box tiếp xúc sát với các điểm ngoài cùng của phần thân xe nhìn thấy được (bao gồm bánh xe, cản trước/sau, nóc xe), không lấy dư mặt đường, bóng đổ hay khoảng trống xung quanh; mỗi xe hợp lệ là 1 box độc lập.
- Vật thể bị che khuất: Nếu xe bị che một phần bởi xe khác, cột đèn, biển báo hoặc dải phân cách nhưng phần nhìn thấy vẫn đủ đặc trưng để nhận diện chắc chắn thuộc 1 trong 4 lớp (`car`, `truck`, `bus`, `van`), vẫn vẽ 1 box bao quanh phần nhìn thấy được. Nếu bị cột mảnh chắn ngang ở giữa nhưng thấy rõ hai đầu của cùng một xe thì vẽ 1 box bao trọn phần xe nhìn thấy.
- Vật thể bị cắt bởi rìa ảnh: Vẽ box dừng đúng tại mép khung ảnh nơi vật thể bị cắt (không kéo vượt biên ảnh `[0, 1]` và không đoán phần nằm ngoài ảnh), miễn là phần trong ảnh đủ nhận diện lớp.
- Vật thể rất nhỏ/mơ hồ: Các xe ở quá xa (như chấm nhỏ gần chân trời/cầu vượt) hoặc chỉ lộ một góc nhỏ không thể xác định chắc chắn lớp thì **không tự đoán và không gán nhãn**, ghi chú lại để hỏi giảng viên. Không gán mô tô, xe máy, xe đạp, người đi bộ, biển báo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe SUV màu trắng ở làn bên phải bị cột đèn kim loại chắn dọc chính giữa thân xe; đồng thời có dãy xe con đang đỗ tĩnh sát lề trái. | Vẽ 1 box `car` (ID 0) bao trọn phần thân xe SUV trắng nhìn thấy ở hai bên cột đèn; vẫn gán `car` cho toàn bộ dãy xe đang đỗ bên lề trái. | Cột đèn mảnh chỉ che một dải hẹp của cùng 1 chiếc xe nên không tách làm 2 box; theo schema, lớp mô tả loại vật thể chứ không phụ thuộc trạng thái đang chạy hay đang đỗ. |
| `drive_008.jpg` | Trong giao lộ xuất hiện xe tải ben chở đất màu đỏ, xe buýt khớp nối dài 2 khoang, 2 xe minivan hình hộp màu trắng/bạc ở giữa ngã tư, các xe con bị xe buýt che một phần và 2 người đi xe máy/mô tô bên trái. | Gán `truck` (ID 1) cho xe tải đỏ; 1 box `bus` (ID 2) cho toàn bộ xe buýt khớp nối; `van` (ID 3) cho 2 xe minivan hình hộp; vẽ box `car` riêng cho phần nhìn thấy của xe con bị che phía sau bus; **không gán** xe máy/mô tô. | Xe tải có thùng hàng rời chở vật liệu là `truck`; xe minivan thân hộp liền khối đầu ngắn là `van`; mô tô/xe máy nằm ngoài 4 lớp của schema nên bỏ qua. |
| `drive_022.jpg` | Xe màu trắng bên phải có kích thước tương đương xe van nhưng có cabin đầu bằng tách biệt với thùng hàng kín phía sau; ở giữa ảnh có xe bạc bị trụ biển báo che một phần. | Gán `truck` (ID 1) cho xe tải thùng kín màu trắng bên phải; vẽ box bao phần nhìn thấy cho xe bị trụ biển báo che. | Dù kích thước nhỏ gọn, xe trắng bên phải có kết cấu cabin rời + thùng hàng vuông chuyên chở hàng hóa (box truck) nên thuộc `truck`, không phải thân liền khối như `van`. |
| `drive_033.jpg` | Xe buýt màu đỏ ở góc dưới trái, xe buýt bên mép phải và xe sedan bạc ở góc dưới phải bị cắt mất một phần thân bởi biên ảnh; phía rất xa ở đầu đường trên cao có các chấm xe rất nhỏ. | Vẽ box `bus` (ID 2) và `car` (ID 0) chạm sát biên ảnh cho các xe bị cắt khung vì vẫn nhận diện rõ loại xe; không gán các chấm xe quá nhỏ/mờ ở tít xa gần cầu vượt. | Quy tắc gán nhãn yêu cầu box chạm đúng rìa ảnh khi xe bị cắt khung mà vẫn đủ nhận diện lớp, và không tự đoán đối với vật thể quá nhỏ/mơ hồ không xác định chắc chắn lớp. |

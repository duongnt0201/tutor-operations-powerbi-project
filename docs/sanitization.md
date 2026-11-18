# Sanitization

Repo này là case study của một báo cáo nội bộ. Nội dung được xử lý như sau trước khi công bố.

## Không công bố

| Hạng mục | Ghi chú |
|---|---|
| File dự án Power BI (`.pbip`, TMDL, PBIR) | Repo chỉ gồm tài liệu và ảnh chụp |
| Dữ liệu (`.pbix`, `cache.abf`) | Không đưa vào repo |
| Kết nối nguồn dữ liệu, câu truy vấn nguồn | Không công bố |
| Cấu hình cục bộ của máy làm việc | Không đưa vào repo |

## Đã ẩn danh trong bản dùng để chụp ảnh và trích tài liệu

| Hạng mục | Cách xử lý |
|---|---|
| Tên thương hiệu, tên sản phẩm | Đổi sang tên trung tính (`Tutor Basic`, `Tutor Pro`...) |
| Tên bảng/cột nội bộ | Đổi sang tên trung tính |
| Logo | Thay bằng logo placeholder |
| Mã trạng thái hệ thống nội bộ | Thay bằng mã trung tính |
| Ngưỡng vận hành | Tách ra bảng `dim_policy` với giá trị minh họa |
| Comment trong DAX | Xóa |
| Tên measure/tiêu đề chứa ngưỡng | Đổi tên trung tính |

## Ảnh chụp màn hình

Ảnh chụp được thực hiện trên dữ liệu giả, sinh ngẫu nhiên:

- Tên gia sư có dạng `Gia sư 001`, số điện thoại có dạng `0900xxxxxx` / `0911xxxxxx`. Không có thông tin nào của người thật.
- Phân phối trạng thái được chọn để report hiển thị hợp lý. Đây **không** phải số liệu thực tế.

## Giữ nguyên

- Cấu trúc model (bảng, quan hệ) và bố cục report.
- Logic DAX của các measure trích trong tài liệu giữ nguyên cấu trúc; mã trạng thái là mã trung tính.

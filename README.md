# Tân Linh Auto – Tailwind V9

V9 tiếp tục từ V8, sử dụng database `tan_linh_auto` hiện tại.

## Mới trong V9
- Dashboard có biểu đồ doanh thu/đã thu 14 ngày và trạng thái phiếu sửa chữa.
- Trang báo cáo có biểu đồ Chart.js.
- Cài đặt thông tin gara: tên, địa chỉ, điện thoại, email, MST, website, logo, ghi chú.
- Logo upload PNG/JPG/WEBP tối đa 2MB.
- Quyền `settings` cho ADMIN/QUAN_LY/KE_TOAN.
- Migration an toàn `database/migrations/002_cai_dat_gara.sql`, không DROP database.

## Cài đặt
1. Giữ nguyên database `tan_linh_auto`.
2. Chạy migration `002_cai_dat_gara.sql` một lần trong phpMyAdmin.
3. Giải nén project vào `C:\xampp\htdocs\TanLinhAuto`.
4. Mở `http://localhost/TanLinhAuto/public/`.
5. Cài đặt gara tại `/cai-dat`.

Chart.js và Tailwind được tải qua CDN; máy cần Internet khi tải giao diện nếu chưa chuyển sang bản local.

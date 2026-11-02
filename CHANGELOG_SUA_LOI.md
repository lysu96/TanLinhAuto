# Báo cáo sửa lỗi – Tân Linh Auto (v9)

Lưu ý: môi trường của tôi không có PHP/MySQL nên chưa chạy thử được. Tôi đã đọc toàn bộ mã, sửa và kiểm tra cân bằng cú pháp. Hãy test trên XAMPP trước khi dùng thật.

## Lỗi nghiêm trọng
1. **Mọi link/menu mất đường dẫn gốc** – `View::render` không truyền `$config` vào view nên `base_url` rỗng (menu, nút, tìm kiếm nhanh sai URL). → `app/Core/View.php`
2. **Tiếp nhận & Thợ sửa bị 403 ở hầu hết trang** – router chặn mọi route phía sau nếu vai trò không có quyền `invoices`/`debt`. → `public/index.php` (bỏ chốt chặn chung, thêm kiểm tra quyền theo từng route).
3. **Trang Kho hàng hiện header/sidebar 2 lần** – view `inventory/*` tự `require` header/footer trong khi `View::render` đã nạp. → 5 view kho.
4. **Công nợ sai trong view `v_cong_no_khach_hang`** – cộng lặp `no_cu` trên mỗi hóa đơn và trừ cả thanh toán của hóa đơn đã hủy → Báo cáo lệch với trang Công nợ. → `tan_linh_auto_fixed.sql` + `database/migrations/003_sua_view_cong_no.sql`.
5. **Dump SQL thiếu 2 bảng** `nhat_ky_he_thong`, `cai_dat_gara` (Cài đặt gara không lưu được, nhật ký trống). → đã thêm vào `tan_linh_auto_fixed.sql`.

## Lỗi nghiệp vụ
6. Hủy phiếu sửa chữa **không hoàn kho** phụ tùng đã xuất; hoàn kho giờ không bị trùng lặp. Phiếu đã hủy không đổi trạng thái được nữa.
7. Vẫn **thêm hạng mục** được vào phiếu đã hủy / đã lập hóa đơn → đã chặn.
8. **Thu tiền** được trên hóa đơn đã hủy → đã chặn.
9. Phiếu kho **"Điều chỉnh" không thể giảm tồn** → nay nhập số âm để giảm (có kiểm tra không âm tồn). Form phiếu kho lấy đủ mọi hàng (trước bị cắt 300).
10. Dropdown **chủ xe** khi thêm xe chỉ hiện 200 khách đầu → hiện đủ.
11. Danh sách khách hàng: công nợ tính cả hóa đơn hủy; "Số xe" tính cả xe đã chuyển chủ → đã sửa.
12. Cho phép **trùng biển số** xe đang hoạt động → đã chặn.
13. Mã tạm `''` (cột UNIQUE) có thể gây lỗi trùng khi 2 người lưu cùng lúc → dùng `LEFT(UUID(),30)`.
14. Tự khóa chính tài khoản đang đăng nhập khi sửa nhân viên → đã chặn.

## Giao diện / khác
15. `app.css` không được nạp (các class `.card`, `.input`, `.btn-primary`… không có tác dụng) → đã nạp; thêm `.badge`.
16. Nút ☰ trên mobile không làm gì → đã hoạt động. Các nút Dashboard, kết quả tìm kiếm nhanh giờ bấm được.
17. Đường dẫn `/TanLinhAuto/public` bị viết cứng ở controller/view → dùng `base_url`.
18. In hóa đơn: bỏ thẻ `<img v-if>` lỗi, hiện logo thật; in phiếu sửa chữa nay dùng thông tin gara.
19. Ghi thêm nhật ký cho: lập hóa đơn, thu tiền, tạo phiếu sửa, đổi trạng thái.

## Chưa sửa (khuyến nghị)
- Chưa có **CSRF token** cho các form POST.
- `QUAN_LY` có thể gán vai trò ADMIN cho nhân viên.
- Chưa có chức năng hủy hóa đơn / giao diện điều chỉnh công nợ.
- Mật khẩu DB `root` rỗng trong `config/config.php` – cần đổi khi chạy thật.

## Cách áp dụng
- DB mới: import `tan_linh_auto_fixed.sql`.
- DB đang có dữ liệu: chạy `database/migrations/001`, `002` (nếu chưa), rồi `003`.
- Chép đè thư mục `TanLinhAuto_v9` (đã sửa) vào `htdocs/TanLinhAuto`.


---
# Cập nhật tính năng (lần 3)

1. **Màn hình gộp "Thêm khách hàng & xe"** (`/khach-hang/them`, `/xe/them` dùng chung): chọn *Khách hàng mới* hoặc *Khách hàng có sẵn* (có ô lọc), nhập một hoặc nhiều xe trong cùng một lần lưu (tối đa 10). Khách + xe được lưu trong **một transaction** – lỗi ở xe nào thì không tạo gì cả. Có 2 nút: *Lưu* và *Lưu & tiếp nhận xe sửa chữa →*. Biển số được chuẩn hóa chữ hoa, chặn trùng (cả trùng trong danh sách đang nhập).
2. **Màn hình Xe & chủ xe**: mỗi xe có nút **Tạo phiếu tiếp nhận** (cũng có ở trang chi tiết xe). Bấm vào sẽ mở form tiếp nhận với khách hàng (chủ hiện tại) và xe đã được chọn sẵn. Xe chưa có chủ hiển thị "Chưa có chủ".
3. **Phiếu sửa chữa – sửa trực tiếp hạng mục**: Hạng mục, Loại, SL, Đơn giá, Giảm sửa ngay trên bảng rồi bấm **Lưu thay đổi** (thành tiền và tổng tính lại ngay khi gõ). Đổi SL hoặc loại của phụ tùng tự động xuất thêm / hoàn kho tương ứng. Phiếu đã hủy hoặc đã có hóa đơn thì khóa không cho sửa. Nếu lưu lỗi, số liệu bạn vừa nhập được giữ lại trên màn hình.
4. Đã chặn xóa hạng mục của phiếu đã hủy (tránh hoàn kho 2 lần); lịch sử kho trong phiếu hiện cả các lần hoàn kho.
5. Trang chi tiết xe nay kiểm tra quyền `vehicles`.

Lưu ý: chưa chạy thử được vì môi trường không có PHP/MySQL; đã kiểm tra cú pháp. Hãy test trên XAMPP.


---
# Cập nhật (lần 4): Khách trả thừa tiền → số dư của khách

- **Thu tiền**: không còn giới hạn bằng số còn nợ. Nếu khách đưa nhiều hơn, hóa đơn được thanh toán đủ và **phần thừa ghi vào số dư của khách** (gara đang nợ khách). Form hiện cảnh báo ngay khi gõ số tiền thừa.
- **Trừ số dư vào hóa đơn khác**: trên hóa đơn còn nợ của khách có số dư, có ô "Trừ số dư" (mặc định trừ tối đa có thể).
- **Hoàn tiền cho khách**: ở trang chi tiết khách hàng có thẻ "Số dư của khách" kèm lịch sử và form hoàn tiền (tiền mặt/chuyển khoản...). Chỉ vai trò có quyền hóa đơn (ADMIN, QUAN_LY, KE_TOAN) mới hoàn được.
- **Trang Công nợ**: thêm bảng "Khách có số dư (gara đang nợ khách)".
- Bảng mới `so_du_khach_hang` (sổ cái số dư) + phương thức thanh toán `SO_DU`. Ứng dụng tự tạo khi cần; cũng có `database/migrations/004_so_du_khach_hang.sql` và đã thêm vào `tan_linh_auto_fixed.sql`.
- Mọi thao tác (thu thừa, trừ số dư, hoàn tiền) chạy trong transaction có khóa khách hàng, chống trừ trùng khi 2 người thao tác cùng lúc; có ghi nhật ký.
- Lưu ý báo cáo: mục "đã thu" cộng cả khoản trừ từ số dư; tiền thừa chỉ được tính vào "đã thu" khi được trừ vào hóa đơn.


---
# Cập nhật (lần 5): Gán / cập nhật chủ cho xe chưa có chủ

- Xe chưa xác định chủ: ở **danh sách Xe & chủ xe** hiện nút **Gán chủ xe**; ở **trang chi tiết xe** có nút "Gán chủ xe" và khung "Xe chưa xác định chủ — cập nhật chủ xe".
- Chọn **khách hàng có sẵn** (có ô lọc theo tên/SĐT/mã) hoặc **thêm khách hàng mới** ngay tại chỗ; chọn ngày bắt đầu sở hữu và ghi chú.
- Khách mới + gán chủ chạy trong một transaction; chặn gán khi xe đã có chủ (kể cả khi 2 người thao tác cùng lúc). Có ghi nhật ký.
- Vai trò không có quyền `customers` (VD: thợ sửa) chỉ chọn được khách có sẵn.
- Sau khi gán chủ, nút "Tạo phiếu tiếp nhận" của xe hoạt động bình thường. Không cần chạy thêm SQL.


---
# Cập nhật (lần 6): Đổi chủ xe

- Trang chi tiết xe đã có chủ: nút **Đổi chủ xe** ở đầu trang, mở khung "Đổi chủ xe (sang tên / bán xe)".
- Chọn **khách hàng có sẵn** (không hiện chủ hiện tại, có ô lọc) hoặc **thêm khách hàng mới**; chọn ngày đổi chủ và ghi chú.
- Hệ thống đóng thời gian sở hữu của chủ cũ (đến ngày trước ngày đổi) và mở chủ mới; **lịch sử chủ cũ được giữ nguyên**, ghi chú hiện trong "Lịch sử chủ xe".
- Chặn: chủ mới trùng chủ hiện tại; ngày ở tương lai hoặc trước ngày chủ hiện tại bắt đầu sở hữu; xe còn phiếu sửa chữa chưa giao xe/chưa hủy của chủ cũ.
- Khách mới + đổi chủ chạy trong một transaction; có ghi nhật ký. Form "Gán chủ" và "Đổi chủ" dùng chung một partial (`vehicles/_owner_form.php`).
- Không cần chạy thêm SQL.


---
# Cập nhật (lần 7): Xóa hóa đơn

- Trang chi tiết hóa đơn có khung **🗑 Xóa hóa đơn** (cuối trang). Chỉ **ADMIN** và **QUAN_LY** thấy và dùng được (quyền riêng `invoices_delete`; KE_TOAN không xóa được).
- Phải **gõ đúng số hóa đơn** để xác nhận, tránh bấm nhầm.
- Xóa vĩnh viễn hóa đơn, chi tiết và lịch sử thanh toán của hóa đơn đó (trong một transaction).
- **Không mất dấu tiền**: nếu hóa đơn đã thu tiền (kể cả phần trừ từ số dư khách), tổng số đã thu được chuyển thành **số dư của khách** (gara đang nợ khách) để trừ vào hóa đơn khác hoặc hoàn tiền ở trang khách hàng. Các dòng số dư liên quan đến hóa đơn được giữ lại (ghi chú vẫn có số hóa đơn).
- Phiếu sửa chữa liên quan giữ nguyên và có thể lập hóa đơn lại. Công nợ của khách tự cập nhật.
- Có ghi nhật ký hệ thống (số hóa đơn, khách, tổng tiền, số đã thu). Không cần chạy thêm SQL.


---
# Cập nhật (lần 8): Lọc nhanh phụ tùng + Sửa / xóa / kiểm kê trong Kho hàng

**Lọc nhanh phụ tùng khi thêm hạng mục vào phiếu sửa chữa**
- Ô tìm "🔍 Gõ mã hoặc tên phụ tùng..." lọc ngay khi gõ, nhiều từ khóa (VD: `loc nhot`), **không cần gõ dấu**, hiện số kết quả (x/tổng).
- Danh sách phụ tùng hiển thị dạng hộp 6 dòng; có tick "Chỉ hiện hàng còn tồn"; hàng hết ghi (HẾT).
- Phím tắt: Enter chọn kết quả đầu tiên, ↓ nhảy xuống danh sách. Chọn phụ tùng sẽ tự điền tên, đơn giá, ĐVT và tự chuyển loại sang "Phụ tùng".

**Kho hàng**
- **Sửa** hàng hóa (`/hang-hoa/{id}/sua`): mã, tên, nhóm, đơn vị, giá nhập/bán, tồn tối thiểu, quản lý tồn, mô tả. Chặn trùng mã hàng (cả khi thêm mới).
- **Kiểm kê / điều chỉnh tồn** (trang chi tiết): nhập tồn thực tế, hệ thống tạo phiếu "Điều chỉnh" ghi vết chênh lệch (không sửa số trực tiếp).
- **Ngừng kinh doanh / kích hoạt lại**: ẩn hàng khỏi danh sách và khi chọn phụ tùng; lịch sử vẫn giữ. Danh sách có tick "Cả hàng ngừng KD" để xem lại.
- **Xóa hàng hóa**: chỉ xóa được hàng chưa từng phát sinh giao dịch (phiếu sửa, hóa đơn, phiếu kho); hàng đã dùng thì chỉ được ngừng kinh doanh.
- Quyền: sửa / ngừng KD / xóa chỉ cho **ADMIN** và **QUAN_LY** (quyền mới `inventory_manage`); kiểm kê tồn giống lập phiếu kho (ai có quyền kho đều làm được). Mọi thao tác ghi nhật ký. Không cần chạy thêm SQL.


---
# Cập nhật (lần 9): Xóa phiếu tiếp nhận

- Trang chi tiết phiếu sửa chữa có khung **🗑 Xóa phiếu tiếp nhận** (cuối trang), phải **gõ đúng mã phiếu** để xác nhận. Chỉ **ADMIN** và **QUAN_LY** (quyền mới `repairs_delete`).
- Xóa phiếu và toàn bộ hạng mục trong một transaction. Phụ tùng đã xuất kho cho phiếu **tự hoàn về kho** (có phiếu nhập hoàn trong lịch sử kho); phiếu đã ở trạng thái Hủy thì không hoàn lại lần nữa.
- Phiếu **đã có hóa đơn** (kể cả hóa đơn đã hủy) không xóa được — cần xóa hóa đơn trước. Muốn giữ lịch sử thì đổi trạng thái sang "Hủy".
- Ghi nhật ký (mã phiếu, khách, xe, số hạng mục, số phụ tùng hoàn kho). Không cần chạy thêm SQL.


---
# Cập nhật (lần 10): Hóa đơn sửa chữa trong chi tiết khách hàng

- Trang chi tiết khách hàng có thêm mục **Hóa đơn sửa chữa**: số hóa đơn (bấm để mở), ngày lập, biển số, phiếu sửa, phải thu, đã thanh toán, còn nợ, trạng thái; kèm tổng hợp (số hóa đơn, tổng phải thu, đã thanh toán, còn nợ — không tính hóa đơn đã hủy). Hiển thị 50 hóa đơn gần nhất.
- Chỉ hiện với vai trò có quyền hóa đơn. Mã phiếu trong "Lịch sử sửa chữa" giờ bấm mở được phiếu (nếu có quyền phiếu sửa chữa). Không cần chạy thêm SQL.

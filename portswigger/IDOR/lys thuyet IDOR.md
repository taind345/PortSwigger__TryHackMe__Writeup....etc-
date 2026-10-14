# Lý thuyết IDOR / Access control - mình tổng hợp lại

-sau khi học phần này, mình hiểu là: IDOR là khi app lấy object theo id mình gửi lên (`?id=123`, `user_id=...`) mà không thèm check id đó có phải của mình không. Đổi số là đọc được đồ người khác.
-chi tiết dài mình để ở [[IDOR]], file này mình tổng hợp ngắn để tra nhanh.

## 1. Mấy dạng mình hay gặp
- [ ] object reference: `/my-account?id=carlos` -> đổi id là xem được account người khác.
- [ ] role trong request: cookie `Admin=false` -> sửa `true`; param `roleid=1` -> đổi `2` thành admin.
- [ ] admin không bảo vệ: `/administrator-panel` vào thẳng được, hoặc lộ trong `/robots.txt`.
- [ ] check theo URL ẩu: chặn `/admin` nhưng dùng `X-Original-URL: /invalid` hoặc `?username=carlos` là qua.
- [ ] check theo method ẩu: GET `/admin/delete` bị chặn nhưng POST thì không.
- [ ] multi-step thiếu check 1 bước: bước 2 upgrade role không check quyền.
- [ ] check theo Referer: thêm `Referer: https://site/admin` là qua.
- [ ] IDOR đọc file người khác: `/download-transcript?key=...` đổi key là đọc chat người khác.

## 2. Cách mình test
- [ ] đăng nhập 2 account (wiener + carlos), lấy id/object của A rồi dùng session B request lại xem có 200 không.
- [ ] thử đổi method GET<->POST, thêm header `X-Original-URL`, `X-Rewrite-URL`, sửa cookie role.
- [ ] soi `/robots.txt`, JS bundle hay lộ endpoint admin.

## 3. Phòng chống - mình tự nhắc
- [ ] check quyền ở server mọi object mọi action, deny by default.
- [ ] dùng id random khó đoán (UUID) + vẫn phải check quyền, đừng tin id client gửi.
- [ ] không giấu admin bằng obscurity (robots, URL lạ).

> [!NOTE]
> -thứ tự lab mình đi ở file: [[0-index IDOR]]

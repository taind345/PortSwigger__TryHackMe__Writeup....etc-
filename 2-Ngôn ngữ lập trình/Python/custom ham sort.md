Trong Python, "mảng" đối tượng thực chất là một Danh sách (List) chứa các đối tượng. Để sắp xếp danh sách này theo ý muốn, bạn sẽ sử dụng hàm `sort()` kết hợp với tham số `key` (thường dùng hàm ẩn danh `lambda`).

Dưới đây là hướng dẫn chi tiết từng bước rất dễ hiểu.

---

## 1. Cách khai báo mảng đối tượng

Đầu tiên, bạn tạo một lớp (Class) thông thường. Sau đó, bạn tạo các đối tượng và bỏ chúng vào trong một dấu ngoặc vuông `[]`.

```python
class HocSinh:
    def __init__(self, ten, diem):
        self.ten = ten
        self.diem = diem

    # Hàm này giúp khi in đối tượng ra màn hình sẽ hiện chữ đẹp, không bị hiện mã khó hiểu
    def __repr__(self):
        return f"{self.ten}: {self.diem}"

# Cách khai báo mảng (danh sách) chứa các đối tượng HocSinh
danh_sach_hs = [
    HocSinh("An", 8.5),
    HocSinh("Bình", 9.5),
    HocSinh("Cường", 7.0)
]

print("Mảng ban đầu:", danh_sach_hs)
```

---

## 2. Cách tự chế (Custom) hàm `sort()` để sắp xếp

Để sắp xếp mảng đối tượng, hàm `sort()` cần bạn chỉ cho nó biết: "Muốn sắp xếp dựa trên thuộc tính nào?". Bạn dùng cấu trúc `key=lambda x: x.thuộc_tính`.

## Trường hợp A: Sắp xếp theo một thuộc tính (Ví dụ: Theo điểm số tăng dần)

```python
# Sắp xếp danh sách học sinh theo điểm tăng dần
danh_sach_hs.sort(key=lambda hs: hs.diem)
print("Điểm tăng dần:", danh_sach_hs)
# Kết quả: [Cường: 7.0, An: 8.5, Bình: 9.5]
```

_Mẹo: Nếu muốn sắp xếp giảm dần, bạn chỉ cần thêm tham số `reverse=True` vào trong hàm: `danh_sach_hs.sort(key=lambda hs: hs.diem, reverse=True)`._

## Trường hợp B: Sắp xếp theo nhiều tiêu chí (Ưu tiên tiêu chí 1, nếu bằng nhau thì xét tiêu chí 2)

Giả sử bạn có hai học sinh trùng điểm nhau, bạn muốn ai có điểm cao hơn xếp trước (giảm dần số điểm), nếu bằng điểm nhau thì ai tên đứng trước trong bảng chữ cái xếp trước (tăng dần theo chữ cái).

==Bạn chỉ cần truyền một bộ dữ liệu `(tiêu_chí_1, tiêu_chí_2)` vào sau dấu `:` của lambda. Để đảo ngược riêng một tiêu chí số, hãy thêm dấu trừ `-` trước thuộc tính đó.==

```python
danh_sach_hs = [
    HocSinh("Cường", 9.0),
    HocSinh("An", 9.0),
    HocSinh("Bình", 9.5)
]

# Ưu tiên 1: Điểm giảm dần (-hs.diem)
# Ưu tiên 2: Tên tăng dần theo bảng chữ cái (hs.ten)
danh_sach_hs.sort(key=lambda hs: (-hs.diem, hs.ten))

print("Sắp xếp nâng cao:", danh_sach_hs)
# Kết quả: [Bình: 9.5, An: 9.0, Cường: 9.0] (An và Cường cùng 9 điểm, An đứng trước vì chữ A đứng trước chữ C)
```

Nếu bạn muốn, tôi có thể hướng dẫn bạn cách sắp xếp ngày tháng tăng dần/giảm dần trong mảng đối tượng hoặc cách dùng hàm `sorted()` để tạo ra mảng mới mà không làm thay đổi mảng gốc.
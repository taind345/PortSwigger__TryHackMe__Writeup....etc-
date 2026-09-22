# Các Kỹ Thuật Escape Trong JavaScript (JavaScript Escapes)

> Ghi chú trích từ cuốn sách **"JavaScript For Hackers"** của tác giả **Gareth Heyes**  
> Link tham khảo: [Amazon - JavaScript for Hackers](https://www.amazon.com/JavaScript-hackers-Learn-think-hacker-ebook/dp/B0BQSRJ71H)


## 1. Hệ Hexadecimal (Thập Lục Phân) -- `'\x72\x73\x30\x6E'` = `rs0n`

- **CHỈ có thể** được sử dụng dưới dạng **Chuỗi (Strings)**:
  ```javascript
  '\x61' // = "a"
  "\x61" // = "a"
  `\x61` // = "a"
  ```
- **KHÔNG THỂ** sử dụng làm **Tên Định Danh (Identifiers / Tên hàm, tên biến)**:
  ```javascript
  function a() { console.log("rs0n"); }
  \x61() // BÁO LỖI cú pháp (SyntaxError)
  ```

---

## 2. Hệ Unicode -- `'\u0072\u0073\u0030\u006E'` = `rs0n`

- Có thể dùng cho **CẢ Chuỗi (Strings) VÀ Tên Định Danh (Identifiers)**:
  ```javascript
  '\u0061' // = "a"
  "\u0061" // = "a"
  `\u0061` // = "a"

  function a() { console.log("rs0n"); }
  \u0061() // HỢP LỆ! In ra "rs0n"
  ```

### Hai kiểu Unicode Escapes:
1. `\u`
   - Bắt buộc phải có đúng **4 chữ số thập lục phân**.
   - `\u61` -> **LỖI** (FAIL).
2. `\u{}`
   - Có thể chứa **số lượng chữ số tùy ý**.
   - `eval("\u{61}" == "a" && "\u{0000000000000061}" == "a" && "\u{0061}" == "a")` -> `true`

---

## 3. Hệ Octal (Bát Phân) -- `'\162\163\060\156'` = `rs0n`

- **CHỈ có thể** được sử dụng dưới dạng **Chuỗi (Strings)**.
*(Lưu ý: Bị cấm trong JavaScript strict mode `"use strict"`)*

---

## 4. Khai Thác Hàm `eval()` (Exploiting Eval)

Hàm `eval()` cho phép bạn phá vỡ các quy tắc giới hạn nêu trên để bypass các bộ lọc bảo mật:

### Các ví dụ:
- **Gán biến bằng Hex trong eval:**
  ```javascript
  eval('\x62 = "rs0n"')
  // 1. \x62 được giải mã thành ký tự 'b'
  // 2. eval('b = "rs0n"') tương đương với việc khai báo biến b = "rs0n"
  ```

- **Double Unicode Escape (Thoát chuỗi Unicode hai lần):**
  ```javascript
  eval('\\u0062 = "rs0n"')
  // 1. Escape dấu gạch chéo ngược: \\u0062 -> \u0062
  // 2. eval('\\u0062 = "rs0n"') tương đương biến b = "rs0n"
  ```

- **Lồng Escape bên trong Escape (Embed Escapes in Escapes):**
  ```javascript
  eval('\\u\x30062 = "rs0n"')
  // Hex Escape số 0 đầu tiên trong chuỗi Unicode Escape:
  // \x30 -> 0 => sinh ra \u0062 (chính là 'b')

  eval('\\u\x300\662 = "rs0n"')
  // Kết hợp cả Hex và Octal Escape để tạo mã Unicode:
  // \x30 -> 0 VÀ \66 -> 6 => sinh ra \u0062
  ```

---

## 5. Chuỗi Ký Tự Trong JavaScript (Strings)

### Ba dạng khai báo chuỗi:
1. Dấu nháy đơn (Single Quotes): `'rs0n'`
2. Dấu nháy kép (Double Quotes): `"rs0n"`
3. Template Strings (Dấu backtick): `` `rs0n` ``

### Ký tự thoát chuỗi đơn (Single Character Escape Sequences):
- `\b` = Xóa lùi (Backspace)
- `\f` = Đẩy trang (Form Feed)
- `\n` = Xuống dòng (New Line)
- `\r` = Về đầu dòng (Carriage Return)
- `\t` = Phím Tab
- `\v` = Tab dọc (Vertical Tab)
- `\0` = Ký tự Null
- `\'` = Dấu nháy đơn (Single Quote)
- `\"` = Dấu nháy kép (Double Quote)
- `\\` = Dấu gạch chéo ngược (Backslash)

> **Mẹo:** Bất kỳ ký tự nào khác đứng sau dấu `\` đều có thể escape mà không làm thay đổi giá trị của ký tự đó:  
> `"\R\S\O\N"` = `"RSON"`

### Kéo dài chuỗi sang dòng tiếp theo:
Dấu gạch chéo ngược (`\`) ở cuối dòng cho phép kéo dài chuỗi văn bản sang dòng kế tiếp:
```javascript
"If you want to live a happy life \
tie it to a goal\
not to people or things."
```

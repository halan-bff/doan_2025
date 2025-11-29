
- Xóa đoạn đầu: 

```bash
<body>[\s\S]*?<div id="chapter-big-container" class="container chapter">
```

- Xóa đoạn giữa : 


→ Find: 
```bash
<div class="row"><div class="col-xs-12">[\s\S]*?<h2><a class="chapter-title"[^>]*><span class="chapter-text">
``` 

→ replace : trông gọn hơn
```bash
<div class="row"><div class="col-xs-12"><h2><span class="chapter-text">
```


- Xóa đoạn cuôi: 

```bash
</div><hr class="chapter-end"\s*\/>[\s\S]*?</body>\s*</html>
```

- Role play


  + Đoạn đầu
```bash
<h2 id="comments">[\s\S]*?</body>\s*</html>
```
  + Đoạn cuối 
```bash
<body>\s*<div id="top">[\s\S]*?</ul>\s*</div>\s*</div>
```
============Lỗi tại nhà CÁT 1 - Bị nhìn trong sigil bị lỗi (chồng chéo, đè lên chữ)===============

Dùng CSS ghi đè (cho toàn sách, lười sửa tay)

Nếu bạn không muốn sửa từng thẻ <p>:

Mở Styles/stylesheet.css (hoặc file css chính).

Thêm vào cuối file:

```bash
.CalendarMonthGrid_month__hideForAnimation,
p.CalendarMonthGrid_month__hideForAnimation {
    line-height: 1.4 !important;
    margin: 0 0 0.5em 0 !important;
    padding: 0 !important;
    font-size: 1em !important;
}
```

Lưu lại rồi xem lại các chương lỗi.


=======================Lỗi tại nhà Cát 2 - xóa các đoạn bị thừa ================================  
Find:

```bash
<p[^>]*class="CalendarMonthGrid_month__hideForAnimation"[^>]*>.*?</p>\s*
```

Replace with: (để trống)

=======================================================  

1. Bước 1 – Mở Console và cho phép dán

Vào chương truyện trên WordPress.

Nhấn F12 → chọn tab Console.

Bắt buộc phải tự Gõ (do nó đang chưa cho paste) :    allow pasting 
→ nhấn Enter (để vượt qua cái warning xanh như lúc trước).

2. Bước 2 – Chạy script xóa tất cả đoạn “ẩn”

Trong Console, dán đúng đoạn này rồi Enter:

```bash
(() => {
  // Xoá mọi đoạn có class rác mà bạn đã thấy
  document.querySelectorAll(
    '.CalendarMonthGrid_month__hideForAnimation'
  ).forEach(e => e.remove());

  // Xoá mọi <p> có style "font-size: 0.0001px" hoặc "line-height: 0.01"
  document.querySelectorAll('p[style]').forEach(p => {
    const s = p.getAttribute('style');
    if (!s) return;
    if (s.includes('font-size: 0.0001px') || s.includes('line-height: 0.01')) {
      p.remove();
    }
  });

  console.log('Đã xoá xong các đoạn ẩn, giờ bạn Ctrl+A / Ctrl+C bình thường nhé.');
})();
```

Giải thích nhanh:

Nó xóa hẳn khỏi trang những thẻ:

có class CalendarMonthGrid_month__hideForAnimation;

hoặc có font-size: 0.0001px / line-height: 0.01 trong style
(chính là các đoạn “ma” bạn thấy trong HTML).

Chỉ xóa trong trang đang mở trên trình duyệt. Reload lại trang là mọi thứ quay về như cũ, không ảnh hưởng web thật.








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


Xóa đoạn cuôi: 

```bash
</div><hr class="chapter-end"\s*\/>[\s\S]*?</body>\s*</html>
```



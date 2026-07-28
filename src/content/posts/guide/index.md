---
title: Hướng dẫn viết file Markdown
published: 2026-07-29
description: "File mẫu duy nhất để tham khảo cách viết và đăng bài Markdown trên web."
tags: ["Markdown", "Hướng dẫn"]
category: Hướng dẫn
draft: false
---

# Hướng dẫn viết file Markdown

Đây là file hướng dẫn duy nhất được giữ lại trong `src/content/posts/`.
Khi bạn thêm file Markdown thật, trang Giới thiệu sẽ tự động cập nhật bảng đóng góp và danh sách tệp đã đăng.

## 1. Tạo file bài viết

Tạo file mới trong thư mục:

```text
src/content/posts/
```

Ví dụ:

```text
src/content/posts/my-note.md
src/content/posts/hackthebox/keeper.md
src/content/posts/portswigger/sql-injection.md
```

## 2. Thêm frontmatter

Mỗi file Markdown nên có phần thông tin ở đầu file:

```yaml
---
title: Tiêu đề bài viết
published: 2026-07-29
description: Mô tả ngắn của bài viết.
tags: [Markdown, Security]
category: Ghi chú
draft: false
---
```

Nếu bài viết thuộc Learn Platform, thêm `platform`:

```yaml
platform: hackthebox
status: in-progress
```

Giá trị `platform` hợp lệ:

- `hackthebox`
- `tryhackme`
- `portswigger`
- `writeup`

Giá trị `status` hợp lệ:

- `not-started`
- `in-progress`
- `completed`

## 3. Viết nội dung

Sau frontmatter, bạn viết nội dung bằng Markdown bình thường:

```md
## Enumeration

- Quét port với nmap.
- Ghi lại service version.
- Tìm endpoint đáng nghi.
```

## 4. Kiểm tra web

Chạy lệnh:

```powershell
pnpm dev
```

Sau đó mở `http://localhost:4321/`.

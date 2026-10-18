---
layout: post
title: Error - Arguments array must have arguments
subtitle: Error - Arguments array must have arguments
published: true
comment: true
tags: [angular, unit test]
gh-badge: [star, fork, follow]
permalink: /post/2023-12-24-angular-ut-troubleshoot
---
## Fix lỗi "Arguments array must have arguments." khi run Angular unit test:
 
### Nguyên nhân: chưa cung cấp đủ các dependency cho constructor của component trong bước khởi tạo
### Giải pháp: cung cấp các dependencies còn thiếu để
### How: 
**Step 1:** Comment các dependencies trong constructor của component, mở từng dependency rồi chạy file test
		-> mục đích tìm dependency nào chưa được provide

**Step 2:** Add dependency vào provider của file test

**Step 3:** run file test và lặp lại **Step 1** cho đến khi file ts uncomment hết trong constructor
---
title: "PortSwigger - SQL injection attack, querying the database type and version on Oracle"
author: Thien Nguyen
date: "2026-07-01"
keywords: [PortSwigger, CTF, Security, Offensive, SQL Injection]
lang: "vi"

---
# Description
- This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.
- To solve the lab, display the database version string.
# Exploitation  
- Đây là giao diện của trang chủ target
![](../../../05-Assets/Pasted%20image%2020260701124322.png)
- Như mô tả đã nói thì ở phần product category có SQL Injection vậy chúng ta sẽ coi thử xem khi gửi category req sẽ gửi như nào
![](../../../05-Assets/Pasted%20image%2020260701124506.png)
- Oke, giờ gửi request này sang repeater của burpsuite và nhìn lại trang chủ chúng ta thấy được rằng nó chạy database oracle sql, trước tiên chúng ta sẽ check xem đây có phải surface attack bằng `'`
![](../../../05-Assets/Pasted%20image%2020260701124729.png)
- Oke có lỗi từ bên server vậy có thể do lỗi syntax SQL gây ra nó có thể có cú pháp như sau
`SELECT col1,col2,... FROM table_name WHERE category='req_category'`
- Giờ chúng ta sẽ check xem số lượng cột mà nó in ra bao nhiêu bằng `' UNION SELECT 'abc','bcd' FROM dual--`
![](../../../05-Assets/Pasted%20image%2020260701125040.png)
- Vậy là có 2 cột giờ đề bài yêu cầu chúng ta đưa ra version string thì chúng ta sử dụng lệnh sau `'+UNION+SELECT+BANNER,+NULL+FROM+v$version--` là xong
![](../../../05-Assets/Pasted%20image%2020260701125219.png)
# Conclusion
- Ở bài này chúng ta biết được rằng với mỗi server khác nhau thì lệnh UNION cũng khác nhau ví dụ như bên MySQL chúng ta chỉ cần UNION SELECT 1 thì bên oracle buộc phải có 1 table đi chung và có table 'dual' được built-in nên có thể tận dụng nó
# References
1. https://portswigger.net/web-security/sql-injection/cheat-sheet
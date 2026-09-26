# Luôn tìm xem có file nào được xét SUID không 
- Đối với các file hoặc directory được xét SUID và nó có quyền cho phép các user khác chạy thì chúng ta có thể lạm dụng nó để chạy lên quyền root
- Có thể tìm kiếm cách để leo thang lên ở đây: https://gtfobins.org/
- Lệnh để tìm SUID: find / -perm /4000 2>/dev/null
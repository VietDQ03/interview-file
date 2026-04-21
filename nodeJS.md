1. NodeJS 
NodeJS là một runtime JS dựa trên V8 thiết kế quanh event-loop và non-block I/O tối ưu cho concurrency triển khai các bài toán I/O bound. Điểm mạnh của NodeJS là mô hình event-driven kết hợp cùng với event-loop, giúp xử lý workload I/O bound hiểu quả nhờ non-blocking I/O. Khi gặp các I/O như DB, file, network thì JS thread sẽ không đứng chờ mà sẽ chuyển cho libuv/OS xử lý, khi có kết quả callback sẽ đưa lại về queue đưa cho event-loop xử lý 
NodeJS là single thread thì nó chỉ đúng với code trên mainJS. Trên thực tế thì nodeJS tận dụng đa luồng ngầm của libuv cho I/O và cho phép chủ động tạo worker thread khi cần tinhs toán nặng, I/O bound tận dụng async, còn cpu bound dùng worker threads 
Khi scale thường dùng cluster/multiple process mỗi process 1 event-loop 
2. NestJS 
NestJS là framework backend cho nodeJS được xây dựng bằng typescript và lấy cảm hứng từ angular kết hợp các kiểu lập trình hướng OOP, FP, FRP 
Điểm mạnh của nestJS: 
  - Kiến trúc rõ ràng
  - Dependency Injection
  - Modular Architechture 
  - Dễ scale cho hệ thông lớn  
3. Request Lifecycle trong NestJS 
request --> middleware --> guard --> Interceptor(before) --> Pipe --> Controller --> Service --> Interceptor(after) --> exception filter 

Middleware có tác dụng giống là quyết định có nên cho request này đi tiếp không, và request này sẽ trông thế nào
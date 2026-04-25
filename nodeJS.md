1. NodeJS
   NodeJS là một runtime JS dựa trên V8 thiết kế quanh event-loop và non-block I/O tối ưu cho concurrency triển khai các bài toán I/O bound. Điểm mạnh của NodeJS là mô hình event-driven kết hợp cùng với event-loop, giúp xử lý workload I/O bound hiểu quả nhờ non-blocking I/O. Khi gặp các I/O như DB, file, network thì JS thread sẽ không đứng chờ mà sẽ chuyển cho libuv/OS xử lý, khi có kết quả callback sẽ đưa lại về queue đưa cho event-loop xử lý
   NodeJS là single thread thì nó chỉ đúng với code trên mainJS. Trên thực tế thì nodeJS tận dụng đa luồng ngầm của libuv cho I/O và cho phép chủ động tạo worker thread khi cần tinhs toán nặng, I/O bound tận dụng async, còn cpu bound dùng worker threads
   Khi scale thường dùng cluster/multiple process mỗi process 1 event-loop

### 1.1. Định nghĩa đúng thuật ngữ

- **Đồng bộ (Synchronous)**  
  Code chạy tuần tự, hàm gọi (_caller_) sẽ chờ hàm hiện tại thực thi xong rồi mới tiếp tục.

- **Bất đồng bộ (Asynchronous)**  
  Hàm gọi (_caller_) không chờ kết quả ngay, mà đăng ký cách nhận kết quả sau (callback, Promise, event, stream).

### 1.2. Nhầm lẫn phổ biến: Async ≠ Song song

- **Async** → xử lý **thời gian chờ (waiting)**, chủ yếu với I/O
- **Parallel** → chạy **đồng thời** trên nhiều thread/core

> Trong Node.js:

- I/O có thể async
- Nhưng **JavaScript execution vẫn chủ yếu chạy trên 1 thread**

### 1.3. Blocking trong Node là gì?

- **Blocking event loop** = làm thread chính bị chiếm quá lâu

Hậu quả:

- Callback I/O bị delay
- Timer bị trễ
- Latency tăng

Ví dụ gây blocking:

- `fs.readFileSync`
- Sync crypto
- Regex phức tạp
- Loop xử lý lớn

### 1.4. 2 trục tư duy khi nói về Sync / Async

#### (1) Góc nhìn API (lập trình)

- **Sync API**
  - Trả kết quả ngay
  - Có thể `throw error`

- **Async API**
  - Trả về `Promise` hoặc dùng `callback`
  - Không block thread

---

#### (2) Góc nhìn hệ thống (workload)

- **I/O-bound**
  - Ví dụ: DB, API, file
  - Async giúp tăng **concurrency**

- **CPU-bound**
  - Ví dụ: loop lớn, parse JSON nặng, xử lý thuật toán
  - Async **không giúp** nếu vẫn chạy trên JS thread
  - Cần:
    - Worker Threads
    - Multi-process
    - Service riêng
#### Trade-off và rủi ro (điểm middle hay nói)

- **Error handling**: callback hell vs Promise chaining vs `async/await` (try/catch).
- **Backpressure**: xử lý stream cần tôn trọng `drain`/`pipeline` để tránh phình memory.
- **Microtask starvation**: lạm dụng `process.nextTick`/Promise chain khiến I/O bị đói.

#### Một câu chốt

“Trong Node, đồng bộ/bất đồng bộ là cách ta **tổ chức việc chờ**. Bất đồng bộ giúp tận dụng thời gian chờ I/O để xử lý việc khác, nhưng nếu công việc là CPU-bound thì cần worker threads hoặc scale bằng multi-process để tránh block event loop.”
### EXAMPLE 
synchornous 
function Test() {
  console.log(1)
  console.log(2)
  console.log(3)
}
Test() 
output 1 , 2 , 3
asynchornous
callback, promise, async/await 
Way 1: callback 
                                      function muaDo(callback) {
                                            console.log("Đi mua đồ...");

                                            setTimeout(() => {
                                              console.log("Mua xong rồi");
                                              callback(); // gọi sau khi xong
                                            }, 2000);
                                          }

                                          muaDo(() => {
                                            console.log("Ok nhận đồ");
                                          });
        OUTPUT print (đi mua đồ ..., mua xong rồi, ok nhận đồ)                                        
Way 2: promise
        function fetchUser() {
          return new Promise((resolve) => {
            setTimeout(() => {
              console.log("Lấy user...");
              resolve({ id: 1, name: "Việt" });
            }, 1000);
          });
        }

        function getOrders(userId) {
          return new Promise((resolve) => {
            setTimeout(() => {
              console.log("Lấy orders của user:", userId);
              resolve([{ id: 101 }, { id: 102 }]);
            }, 1000);
          });
        }

        function getPayment(orderId) {
          return new Promise((resolve) => {
            setTimeout(() => {
              console.log("Lấy payment của order:", orderId);
              resolve({ id: 5001, amount: 200 });
            }, 1000);
          });
        }
        fetchUser()
          .then(user => getOrders(user.id))
          .then(orders => getPayment(orders[0].id))
          .then(payment => console.log(payment))
          .catch(err => console.log(err));
WAY 3 : async/ await
          async function main() {
              const user = await fetchUser();
              const orders = await getOrders(user.id);
              const payment = await getPayment(orders[0].id);

              console.log("Payment:", payment);
          }

          main();
   
2. NestJS
NestJS là framework backend cho nodeJS được xây dựng bằng typescript và lấy cảm hứng từ angular kết hợp các kiểu lập trình hướng OOP, FP, FRP
Điểm mạnh của nestJS:
  - Kiến trúc rõ ràng
  - Dependency Injection
  - Modular Architechture
  - Dễ scale cho hệ thông lớn
3. Request Lifecycle trong NestJS
request --> middleware --> guard --> Interceptor(before) --> Pipe --> Controller --> Service --> Interceptor(after) --> exception filter

┌─────────────────────────────────────────────────────────────┐
│ HTTP Request đến Express/Fastify                            │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ [1. Middleware] — Express-level, chưa biết route            │
│   • CORS, helmet, cookie-parser, body-parser, logging thô   │
│   • Không đọc được metadata (@Role, @Public...)             │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ [2. Guard] — Đã resolve route, biết handler                 │
│   • Auth, permission, role, feature flag                    │
│   • Đọc metadata qua Reflector (@Roles, @Public...)         │
│   • Return false → ForbiddenException → Exception Filter    │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ [3. Interceptor "before"] — Bọc ngoài handler (RxJS)        │
│   • Start timer, gắn requestId, enrich user context         │
│   • Cache check (có thể KHÔNG gọi next.handle() để chặn)    │
│   • Open transaction context, log request                   │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ [4. Pipe] — Resolve & validate từng parameter của handler   │
│   • ValidationPipe (DTO), ParseIntPipe, ParseUUIDPipe       │
│   • Transform: string "42" → number 42                      │
│   • Pattern "Pipe as resolver" — load entity từ DB          │
│   • Throw BadRequestException → Exception Filter            │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ [5. Controller Handler]                                     │
│   • Nhận data đã validate + transform                       │
│   • Mỏng: chỉ điều phối, không chứa business logic          │
│   • Gọi xuống Service                                       │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ [6. Service (if exists)] — Business logic layer             │
│   • Xử lý nghiệp vụ, orchestrate nhiều repository           │
│   • Gọi Repository / Prisma / ORM → Database                │
│   • Throw domain exception → Exception Filter               │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  Service → Repository → ORM (Prisma) → Database     │   │
│   │  (tầng kiến trúc do bạn tự tổ chức, NestJS          │   │
│   │   không hook vào — chỉ inject qua DI)               │   │
│   └─────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────┘
                       ↓ (return value bubble up)
┌─────────────────────────────────────────────────────────────┐
│ [7. Interceptor "after"] — phần sau next.handle() (RxJS)    │
│   • Transform response: { data, meta, timestamp }           │
│   • ClassSerializerInterceptor (entity → DTO)               │
│   • Log duration, commit transaction, cache result          │
│   • Mask field nhạy cảm (password, token)                   │
└──────────────────────┬──────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────┐
│ NestJS serialize response → Express/Fastify gửi về client   │
└─────────────────────────────────────────────────────────────┘


            ╔═══════════════════════════════════════╗
Throw ở    ═►║  [Exception Filter] — Tầng cuối       ║═► Error Response
bất kỳ      ║  • @Catch() bắt HttpException / all   ║   (đã format)
bước 2–7    ║  • Chuẩn hóa: { statusCode, message,  ║
            ║    error, path, timestamp, requestId }║
            ║  • Log Sentry, ẩn stack ở production  ║
            ╚═══════════════════════════════════════╝


╔═══════════════════════════════════════════════════════════════╗
║ GHI CHÚ QUAN TRỌNG                                            ║
║                                                               ║
║ • Bước 1–5, 7, 8: NestJS CHỦ ĐỘNG hook vào (framework-managed)║
║ • Bước 6 (Service): "if exists" — chỉ là class được DI gọi    ║
║   từ Controller, KHÔNG phải hook của framework                ║
║                                                               ║
║ • Middleware: không đọc được metadata                         ║
║ • Guard/Interceptor/Pipe/Filter: đọc được metadata qua        ║
║   Reflector → đây là sức mạnh của decorator-driven design     ║
║                                                               ║
║ • Thứ tự Interceptor: đăng ký trước → "before" chạy trước,    ║
║   "after" chạy sau (onion model, giống middleware stack)      ║
╚═══════════════════════════════════════════════════════════════╝

# Vòng đời một Request trong NestJS

## 1. Middleware
- Có tác dụng giống lớp quyết định có nên cho request này đi tiếp không, và request này sẽ trông thế nào.
- Chạy **sớm nhất** trong pipeline, cấp độ Express-style (chưa biết controller/handler nào sẽ xử lý).
- Dùng cho: parse body, cookie, CORS, helmet, compression, rate-limit, log raw request.
- Có thể kết thúc request sớm bằng `res.send()` mà không gọi `next()`.
- Hạn chế: **không truy cập được metadata của route** (không biết `@Role`, `@Public`...).

---

## 2. Guard
- Là lớp quyết định **truy cập**, bao gồm cả authentication và authorization.
- Sức mạnh của Guard nằm ở khả năng **đọc metadata từ decorator** (`@Role`, `@Public`, `@Permission`) thông qua `Reflector`.
- Trả về `boolean` (hoặc `Promise<boolean>` / `Observable<boolean>`): `true` → đi tiếp, `false` → throw `ForbiddenException` (403).
- Có thể truy cập `ExecutionContext` → biết được class, handler, request hiện tại.
- Ví dụ điển hình: `AuthGuard('jwt')`, `RolesGuard`, `PermissionsGuard`.

---

## 3. Interceptor (before)
- Dùng cho: logging request, gắn `requestId`, đo thời gian, cache check, enrich user context, set header chuẩn bị.
- Có thể **chặn handler** bằng cách không gọi `next.handle()` (như pattern cache — trả thẳng data từ cache, handler không chạy).
- Bọc quanh handler theo kiểu AOP (aspect-oriented), dùng RxJS `Observable`.
- Khác Middleware: biết được context đầy đủ (class, handler, metadata).
- Khác Guard: không dùng để chặn quyền, mà để **biến đổi luồng**.

---

## 4. Pipe
- Là **người gác cổng dữ liệu** của controller.
- Nhiệm vụ: **Transform** (đổi kiểu) + **Validate** (kiểm tra hợp lệ).
- 4 cách dùng: Param-level, method-level, controller-level, global.
- Built-in phổ biến: `ValidationPipe` (quan trọng nhất), `ParseIntPipe`, `ParseUUIDPipe`, `DefaultValuePipe`.
- Custom pipe: implement `PipeTransform`, viết logic trong `transform()`.
- Pattern mạnh: **"Pipe as resolver"** — load entity từ DB luôn trong Pipe.
- Luôn dùng **class** (không dùng interface) cho DTO để decorator tồn tại ở runtime.
- Luôn bật `whitelist` + `transform` + `forbidNonWhitelisted` ở production.

---

## 5. Controller Handler
- Là nơi **nghiệp vụ thật sự** được gọi — sau khi request đã qua hết các lớp bảo vệ và dữ liệu đã sạch.
- Tại thời điểm này: user đã xác thực (Guard), dữ liệu đã validate + transform (Pipe), context đã enrich (Interceptor).
- Nguyên tắc: **controller phải mỏng** — chỉ gọi service, không chứa business logic.
- Trả về giá trị (object, array, primitive) hoặc `Promise` / `Observable` — NestJS tự serialize sang JSON.
- Có thể throw exception (`NotFoundException`, `ConflictException`...) → nhảy thẳng tới Exception Filter.
- Không nên thao tác trực tiếp với `req`, `res` (trừ streaming/SSE) vì sẽ phá vỡ pipeline của Interceptor sau.

---

## 6. Interceptor (after)
- Chạy **sau khi handler trả về giá trị**, trước khi response được gửi về client.
- Dùng cho: transform response (chuẩn hóa format `{ data, meta, timestamp }`), serialize entity → DTO (`ClassSerializerInterceptor`), đo tổng thời gian xử lý, log response, mask field nhạy cảm (password, token).
- Hoạt động qua RxJS operator: `next.handle().pipe(map(...), tap(...), catchError(...))`.
- Có thể **bắt lỗi** bằng `catchError` — nhưng thường để Exception Filter làm việc đó cho sạch.
- Pattern phổ biến: `TransformInterceptor` bọc mọi response thành `{ success, data, timestamp, requestId }`.
- Thứ tự: Interceptor đăng ký trước sẽ **chạy "before" trước và "after" sau** (stack — giống middleware onion model).

---

## 7. Exception Filter
- Là lớp **cuối cùng** trong pipeline, bắt mọi exception throw ra từ bất kỳ tầng nào (Guard, Pipe, Interceptor, Controller, Service).
- Nhiệm vụ: **format error response** thống nhất cho toàn app.
- Dùng cho: chuẩn hóa error shape (`{ statusCode, message, error, path, timestamp, requestId }`), log error vào file/Sentry, ẩn stack trace ở production, dịch message sang ngôn ngữ client.
- Built-in: `BaseExceptionFilter` — filter mặc định của NestJS.
- Custom filter: implement `ExceptionFilter`, dùng decorator `@Catch(HttpException)` để chỉ định loại exception cần bắt.
- `@Catch()` không tham số → bắt **tất cả** exception (kể cả non-HTTP như `TypeError`, `PrismaError`).
- 4 cách dùng giống Pipe: method-level, controller-level, global (`app.useGlobalFilters()` hoặc `APP_FILTER` token).
- Pattern mạnh: **filter riêng cho từng loại lỗi** (`PrismaExceptionFilter`, `ValidationExceptionFilter`, `HttpExceptionFilter`) — NestJS chọn filter cụ thể nhất để xử lý.
- Luôn có một **global filter fallback** để không bao giờ trả về 500 với message mặc định `"Internal server error"` trống trải.

---

## Tổng kết luồng

```

Request
→ Middleware (raw-level, chưa biết route)
→ Guard (chặn quyền, đọc metadata)
→ Interceptor (before) (log, cache check, enrich)
→ Pipe (transform + validate input)
→ Controller Handler (gọi service, xử lý nghiệp vụ)
→ Interceptor (after) (transform response, log)
→ Exception Filter (bắt lỗi ở bất kỳ tầng nào)
→ Response

```

**Nguyên tắc phân chia trách nhiệm**:
- Check "có được vào không?" → **Guard**.
- Check "dữ liệu có hợp lệ không?" → **Pipe**.
- Đo/log/cache/transform → **Interceptor**.
- Format lỗi → **Exception Filter**.
- Parse body, CORS, helmet → **Middleware**.
- Business logic → **Service** (không phải Controller).
```

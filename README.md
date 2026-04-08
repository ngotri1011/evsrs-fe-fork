# EVSRS Frontend

## 1) Tổng quan dự án

Đây là frontend cho hệ thống thuê xe điện, hỗ trợ nhiều vai trò người dùng:

- **Renter (USER)**: tìm xe, đặt xe, thanh toán, quản lý hồ sơ/chuyến đi/giao dịch.
- **Staff (STAFF)**: quản lý xe tại depot, quản lý chuyến, tạo user tại depot, tạo đơn tại depot.
- **Admin (ADMIN)**: quản trị người thuê, hãng xe, mẫu xe, xe EV, depot, đơn đặt, nhân sự, tiện ích, cấu hình hệ thống, cấu hình membership, giao dịch.

Ngoài luồng màn hình chính còn có nhóm màn hình giấy tờ/biên bản:
- Hợp đồng (`contract`)
- Kiểm tra bàn giao (`hand-over-inspection`)
- Kiểm tra hoàn trả (`return-inspection`)
- Quyết toán hoàn trả (`return-settlement`, `customer-settlement`)

## 2) Chức năng chính 

### Public + Renter

- Trang chủ, giới thiệu, chính sách thuê, chính sách bảo mật.
- Đăng nhập/đăng ký + OTP + hoàn tất đăng ký (qua `authAPI`).
- Tìm xe và lọc xe, đặt xe, trả tiền/checkout, trang payment.
- Khu vực tài khoản:
  - Hồ sơ cá nhân
  - Chuyến đi của tôi + chi tiết từng đơn
  - Xem các biểu mẫu/hồ sơ liên quan chuyến đi
  - Lịch sử giao dịch
  - Đổi mật khẩu

### Staff

- Dashboard staff.
- Quản lý xe tại depot.
- Quản lý chuyến đi và xem chi tiết theo `orderId`.
- Tạo user tại depot.
- Tạo order tại depot (luồng offline/at depot).

### Admin

- Dashboard tổng quan.
- Renter management.
- Car manufacture management.
- Model management.
- Staff management.
- Amenities management.
- Depot management.
- Order booking management + xem chi tiết order và hồ sơ đi kèm.
- Car EV management.
- Membership config management (users/configs).
- System config management.
- Transactions.

## 3) Công nghệ sử dụng

- **React 19** + **TypeScript**
- **Vite 7**
- **React Router DOM 7** (`createBrowserRouter`, `RouterProvider`)
- **TanStack React Query 5** (+ Devtools)
- **Axios** (HTTP client chính, có interceptor)
- **Zustand** (auth state persist localStorage)
- **Tailwind CSS 4** + `@tailwindcss/vite`
- **Radix UI** + các UI component tùy biến trong `src/components/ui`
- **React Hook Form** + `zod`/`yup` (validation/form handling)
- **Recharts** (biểu đồ dashboard)
- **Firebase** dependency + service worker `public/firebase-messaging-sw.js`
- **Cloudinary** (upload ảnh biên bản theo biến môi trường)

## 4) Kiến trúc và tổ chức code

### Điều hướng và phân quyền

- Router tập trung tại `src/router/router.tsx`.
- Chia route theo layout/role:
  - `AppLayout` cho luồng chung
  - `AdminLayout` cho `/admin/*`
  - `StaffLayout` cho `/staff/*`
- `AuthGuard` kiểm tra đăng nhập + role (`USER`/`STAFF`/`ADMIN`).

### Quản lý auth

- `useAuthStore` (Zustand + persist) lưu `accessToken`, `refreshToken`, `user`.
- `AuthProvider` cung cấp `login/logout/hasRole` cho UI.
- Axios interceptor:
  - Tự gắn `Authorization: Bearer ...`
  - Tự refresh token khi nhận `403`
  - Queue request trong lúc refresh token

### Tầng API và dữ liệu

- API clients nằm trong `src/apis/*.ts` (auth, car-ev, order-booking, depot, transaction, membership, system-config, feedback, giấy tờ...).
- HTTP base URL lấy từ `VITE_API_URL`.
- React Query dùng cho fetch/cache/retry ở nhiều module; `QueryProvider` cấu hình retry/staleTime/gcTime toàn cục.

### Kiểu dữ liệu và domain

- Type definitions tập trung ở `src/@types` (auth, car, order, payment, response, enum...).
- Các trang theo domain/role nằm trong `src/page`.
- Hooks nghiệp vụ dùng lại trong `src/hooks` và `hooks` theo từng module con.

## 5) Cấu trúc thư mục chính

```txt
src/
  apis/            # API clients theo domain
  @types/          # TypeScript types/interfaces/enums
  components/      # shared components + ui
  context/         # React context (auth)
  hooks/           # custom hooks dùng chung
  layouts/         # App/Admin/Staff layouts + guard
  lib/             # axios, utils, zustand store, helpers
  page/
    admin/         # màn hình quản trị
    renter/        # màn hình người thuê
    staff/         # màn hình nhân viên
    paper/         # hợp đồng/biên bản/settlement
    public-page/   # policy/security pages
  router/          # cấu hình route
```

## 6) Biến môi trường đang được sử dụng trong source

- `VITE_API_URL`
- `VITE_CLOUDINARY_CLOUD_NAME`
- `VITE_CLOUDINARY_UPLOAD_PRESET`

## 7) Chạy dự án local

### Cài đặt

```bash
npm install
```

### Chạy dev

```bash
npm run dev
```

### Build

```bash
npm run build
```

### Preview bản build

```bash
npm run preview
```

---

Nếu bạn muốn, mình có thể tách README thành 2 phần rõ hơn: **README người mới vào dự án** (onboarding) và **README kỹ thuật cho dev** (kiến trúc + conventions + flow auth/API).  

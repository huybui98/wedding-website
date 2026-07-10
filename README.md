# Hướng dẫn kết nối Firebase cho Wedding Website (Ngọc Huy & Quỳnh Hương)

Trang web đã được nối sẵn code để dùng **Firebase Firestore** (lưu RSVP + Lời chúc, real-time thật)
và **Firebase Authentication** (đăng nhập admin bằng email/mật khẩu thật, khách được cấp danh tính ẩn danh
để giữ quyền sửa lời chúc lâu dài). Bạn chỉ cần làm theo các bước dưới đây — không cần biết code.

---

## Bước 1 — Tạo project Firebase (miễn phí)

1. Vào https://console.firebase.google.com → **Add project** (Thêm dự án).
2. Đặt tên bất kỳ, ví dụ `ngochuy-quynhhuong-wedding`.
3. Bỏ chọn Google Analytics nếu không cần (không ảnh hưởng gì).
4. Bấm **Create project**, đợi vài giây.

## Bước 2 — Lấy cấu hình Web App

1. Trong Project, bấm biểu tượng **`</>`** (Web) để "Add app".
2. Đặt tên app (ví dụ "wedding-site"), **không cần** tick Firebase Hosting ở bước này.
3. Firebase sẽ hiện ra một đoạn `firebaseConfig` dạng:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "ngochuy-quynhhuong-wedding.firebaseapp.com",
     projectId: "ngochuy-quynhhuong-wedding",
     storageBucket: "ngochuy-quynhhuong-wedding.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef123456"
   };
   ```
4. Mở file `wedding-huy-linh.html`, tìm đoạn `const firebaseConfig = {` ở gần đầu file
   (trong `<head>`), **dán đè** các giá trị bạn vừa copy vào đúng 6 dòng tương ứng.

## Bước 3 — Bật đăng nhập Email/Password (để vào trang Admin)

1. Trong Firebase Console, vào menu **Authentication** → tab **Sign-in method**.
2. Bấm **Email/Password** → **Enable** → Save.
3. Qua tab **Users** → **Add user** → nhập email + mật khẩu của cô dâu/chú rể
   (ví dụ `admin@ngochuy-quynhhuong.wedding` — bạn có thể dùng email thật của bạn cũng được).
4. Mở lại file HTML, tìm dòng:
   ```js
   window.ADMIN_EMAIL = "admin@ngochuy-quynhhuong.wedding";
   ```
   Sửa thành **chính xác** email bạn vừa tạo ở bước 3.

⚠️ Địa chỉ này còn được dùng để kiểm tra quyền trong Security Rules ở Bước 5 — phải khớp 100%.

## Bước 4 — Tạo Firestore Database

1. Vào menu **Firestore Database** → **Create database**.
2. Chọn **Start in production mode** → chọn khu vực gần bạn (ví dụ `asia-southeast1`) → Enable.

## Bước 5 — Dán Security Rules (bảo vệ dữ liệu)

Vào tab **Rules** trong Firestore, xoá hết nội dung mặc định, dán đoạn sau:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // RSVP: ai cũng gửi được (mỗi khách 1 bản ghi duy nhất theo ownerUid ẩn danh,
    // gửi lại sẽ CẬP NHẬT thay vì tạo trùng). Chỉ admin mới ĐỌC/XOÁ được danh sách.
    match /rsvps/{docId} {
      allow create: if true;
      allow update: if request.auth != null
        && request.auth.uid == resource.data.ownerUid;
      allow read, delete: if request.auth != null
        && request.auth.token.email == "admin@ngochuy-quynhhuong.wedding";
    }

    // Lời chúc: ai cũng đọc/gửi được (bảng lời chúc công khai).
    // Sửa: chỉ người tạo ra lời chúc đó (theo ownerUid ẩn danh).
    // Xoá: chỉ admin.
    match /wishes/{docId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update: if request.auth != null
        && request.auth.uid == resource.data.ownerUid;
      allow delete: if request.auth != null
        && request.auth.token.email == "admin@ngochuy-quynhhuong.wedding";
    }
  }
}
```

**Nhớ sửa** `admin@ngochuy-quynhhuong.wedding` trong đoạn Rules này thành đúng email bạn tạo ở Bước 3
(2 chỗ). Bấm **Publish** để lưu.

## Bước 6 — Bật Anonymous Sign-in (bắt buộc, để khách sửa được lời chúc của họ)

1. Vẫn ở **Authentication → Sign-in method**.
2. Bấm **Add new provider** → **Anonymous** → Enable → Save.

(Đây là cách khách được cấp một "danh tính ẩn danh" riêng, giữ nguyên trên máy/trình duyệt của họ,
để hệ thống biết lời chúc nào là của họ mà cho phép sửa — không cần khách đăng nhập hay để lại thông tin gì thêm.)

## Bước 7 — Deploy trang web

Bạn có thể deploy file `wedding-huy-linh.html` ở **bất kỳ nơi nào** — Vercel, Netlify, hoặc Firebase Hosting
đều được, vì Firestore/Authentication hoạt động độc lập với nơi bạn host file tĩnh:

- **Netlify/Vercel**: kéo-thả file (đổi tên thành `index.html`) vào là chạy.
- **Firebase Hosting** (nếu muốn dùng luôn trong cùng project):
  ```
  npm install -g firebase-tools
  firebase login
  firebase init hosting   (chọn project vừa tạo, thư mục public chứa index.html)
  firebase deploy
  ```

## Bước 8 — Kiểm tra lại

1. Mở trang đã deploy → thử gửi 1 lời chúc → nó phải hiện ngay trong danh sách.
2. Mở trang đó ở **một thiết bị/trình duyệt khác** → lời chúc bạn vừa gửi phải xuất hiện (real-time thật).
3. Vào `/#/admin` → đăng nhập bằng email/mật khẩu đã tạo ở Bước 3 → phải thấy RSVP + nút xoá lời chúc.
4. Gửi thử 1 RSVP → mở tab Admin đang mở sẵn → phản hồi phải tự hiện ra, không cần bấm gì thêm.

---

## Lưu ý quan trọng

- **Free tier (Spark plan)** của Firebase đủ dùng cho một trang cưới thông thường (khoảng 50.000 lượt đọc
  + 20.000 lượt ghi/ngày) — hầu như không tốn phí trừ khi có hàng chục nghìn khách truy cập cùng lúc.
- Quyền "sửa lời chúc của tôi" dựa trên **Anonymous Auth UID**, được trình duyệt lưu lại — nếu khách xoá dữ liệu
  trình duyệt (clear site data) hoặc dùng chế độ ẩn danh (Incognito) rồi đóng tab, họ sẽ mất quyền sửa lời chúc cũ
  (vẫn xem được, chỉ không sửa được nữa). Đây là giới hạn hợp lý cho một trang không yêu cầu khách đăng nhập.
- Nếu bạn xem thử file này ngay trong Claude (dưới dạng artifact/preview) mà chưa deploy, Firebase có thể
  **không tải được** do giới hạn mạng của môi trường preview — hãy deploy thật (Netlify/Vercel/Firebase Hosting)
  để kiểm tra đầy đủ.
- Muốn đổi mật khẩu admin: vào Firebase Console → Authentication → Users → chọn user → Reset password.
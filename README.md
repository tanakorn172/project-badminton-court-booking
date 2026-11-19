# ระบบจองสนามแบดมินตัน

Laravel 12 · Blade + Tailwind (CDN) + Alpine.js · MySQL 8 · Session auth + สิทธิ์แบบ permission

เอกสารออกแบบ (ขอบเขต, หน้าจอ, คู่มือโค้ด) อยู่ใน Claude Project "badminton booking" — `scope.md` คือสรุปทั้งระบบ

## วิดีโอสาธิต

- [วิดีโอฝั่ง Admin (admin.zip)](https://github.com/tanakorn172/project-badminton-court-booking/raw/main/admin.zip)
- [วิดีโอฝั่ง User (user.zip)](https://github.com/tanakorn172/project-badminton-court-booking/raw/main/user.zip)

ดาวน์โหลดแล้วแตกไฟล์ zip เพื่อดูวิดีโอ

## สถานะ

| Milestone | งาน | สถานะ |
|---|---|---|
| **M1** | Auth + RBAC (เข้าสู่ระบบด้วยเบอร์/อีเมล, สมัคร, ลืมรหัสผ่าน, รหัสชั่วคราว, PDPA, permission) | **ทำแล้ว** |
| **M2** | สนาม / ราคารายสนาม / เวลาเปิด-ปิด / วันหยุดร้าน / ปิดชั่วคราว (Admin: `/admin/settings`) | **ทำแล้ว** |
| **M3** | จอง + กันซ้ำ + หมดเวลา + กฎการจอง (หน้าจองลูกค้า `/`, หน้าชำระเงิน แสดงสรุป+นับถอยหลัง) | **ทำแล้ว** (รอทดสอบบนเครื่อง) |
| **M4 (รอบ 1)** | จ่ายด้วย QR พร้อมเพย์ + อัปโหลดสลิป + ตรวจสลิป (Thunder / โหมดจำลอง) + หน้า "ตรวจสลิป" ของพนักงาน + การ์ดวิธีรับเงิน | **ทำแล้ว** (รอทดสอบบนเครื่อง) · Omise ทำรอบถัดไป |
| **M4 (รอบ 2)** | Payment Gateway (Omise) + webhook | **เอาออกแล้ว 2026-10-14** — ชำระเงินเหลือ 2 ทาง: Thunder ตรวจสลิปอัตโนมัติ · พนักงานกดยืนยันเอง |
| **M5** | ตารางสนามหลังร้าน + drawer รายละเอียด/ประวัติ · walk-in เงินสด · ~~กะเงินสด~~ (เลิกใช้ 2026-10-21) · เช็คอิน/ไม่มา (มือ + อัตโนมัติ) · เปลี่ยนสนาม · ยกเลิก (Admin) · รายการจ่ายช้า · ปลดระงับลูกค้า · รายการการจอง/ค้นหา | **ทำแล้ว** (รอทดสอบบนเครื่อง) |
| **M6** | การจองของฉัน (ลูกค้า) · สรุปยอดรายวัน/เดือน (ดูอย่างเดียว) · จัดการลูกค้า (รีเซ็ตรหัสชั่วคราว, แก้ไข, ระงับ) และพนักงาน (เพิ่ม, ตำแหน่ง, ลบ) | **ทำแล้ว** (รอทดสอบบนเครื่อง) |
| **M7** | คู่มือเดโม + คู่มือดีพลอย + รายการตรวจก่อนใช้จริง (ด้านล่าง) | **ทำแล้ว** · เหลือทดสอบบนมือถือ/สลิป/QR จริงโดยเจ้าของ |

## ติดตั้ง (Windows + XAMPP)

ต้องมี **PHP 8.2 ขึ้นไป** (เช็คด้วย `php -v` — XAMPP รุ่นเก่าเป็น 8.0/8.1 ให้โหลด XAMPP รุ่นใหม่กว่า), **Composer**, และ **MySQL 8**
(XAMPP มี MariaDB มาให้ ถ้าใช้ MySQL 8 แยก อย่าเปิดสองตัวพร้อมกันบนพอร์ต 3306)
ใน `php.ini` ต้องเปิด `extension=pdo_mysql`, `extension=pdo_sqlite` (ใช้ตอนรันเทส), `extension=mbstring`, `extension=fileinfo`, `extension=openssl`

เปิด Command Prompt ในโฟลเดอร์โปรเจกต์ แล้วรันทีละบรรทัด:

```bat
composer install
copy .env.example .env
php artisan key:generate
```

1. สร้างฐานข้อมูลว่าง ๆ ชื่อ `badminton_booking` (collation `utf8mb4_unicode_ci`) ใน phpMyAdmin หรือ MySQL Workbench
2. เปิดไฟล์ `.env` แก้ `DB_USERNAME` / `DB_PASSWORD` ให้ตรงกับเครื่อง, แก้ `ADMIN_EMAIL` / `ADMIN_PASSWORD` เป็นของคุณ

```bat
php artisan migrate --seed
php artisan serve
```

ถ้าอัปเดตเป็นรุ่น 2026-10-24 (ตรวจทั้งระบบ): `php artisan migrate` (มี migration 2026-10-21 สามไฟล์ถ้ายังไม่ได้รัน) แล้ว `php artisan test` ต้องผ่าน **235** ข้อ · สิ่งที่เปลี่ยน: ลูกค้าเห็นการจองที่ "ไม่มา" พร้อมรหัสเช็คอินจนหมดเวลา · Admin ยกเลิก/ย้ายการจองที่ "ไม่มา" ได้ · ป้าย "รอตรวจสลิป" · พนักงาน 2 คนกดยืนยัน/ปฏิเสธสลิปพร้อมกันไม่พัง · ปฏิเสธสลิปหลังเลยเวลาเล่น = หมดเวลาชำระ ปล่อยช่อง · ต่อเวลาระหว่างรอ Thunder · จำกัดการกดถี่ตามชื่อ (login/register/forgot/reset/booking/slip) · แบ่งหน้าภาษาไทย · แก้ XSS ในกล่องยืนยันหน้าผู้ใช้ · วันหยุดนับการจองที่ชนเฉพาะที่ยังไม่จบ

ถ้าอัปเดตเป็นรุ่น 2026-10-21 (เลิกใช้กะเงินสด): `php artisan migrate` (ลบตาราง cash_shifts + payments.shift_id + สิทธิ์ shift.*) แล้ว `php artisan test`

ถ้าอัปเดตจาก M5 มา M6: ไม่ต้อง composer require เพิ่ม · แล้ว `php artisan test` (ต่อมา 2026-10-21 เอา export Excel / พิมพ์ ออกจากสรุปยอด: `php artisan migrate` ลบสิทธิ์ report.export)

ถ้าอัปเดตจาก M4 มา M5: `php artisan migrate` (ตาราง cash_shifts) แล้ว `php artisan test` · งานอัตโนมัติใหม่ `bookings:no-show` และ `bookings:complete` ทำงานผ่าน scheduler ทุกนาที (`php artisan schedule:work` ตอนพัฒนา)


ถ้าอัปเดตจาก M3 มา M4 (รอบ 1): รัน 1) `composer require bacon/bacon-qr-code` (ตัวสร้างรูป QR ไม่ต้องมี GD) 2) `php artisan migrate` 3) เพิ่มใน `.env` (ดู `.env.example`): `SLIP_DRIVER=fake`, `SHOP_PROMPTPAY_ID=...` (พร้อมเพย์ร้าน), `SHOP_PROMPTPAY_NAME=...` แล้ว `php artisan config:clear` 4) `php artisan test`
- ตัวจำลอง (`SLIP_DRIVER=fake`) ใช้ได้เฉพาะตอน `php artisan test` (phpunit.xml ตั้งให้) · เปิดเว็บจริงตรวจด้วย Thunder เสมอ · ชื่อไฟล์จำลองในเทสต์: `unreadable`, `wrongamount`, `wrongname`, `old`, `unavailable`
- ใช้งานจริงแต่ยังไม่มี key Thunder: `SLIP_DRIVER=thunder` (ไม่ใส่ key) → ทุกสลิปไป "รอพนักงานตรวจ" ที่เมนู ตรวจสลิป (`/admin/payments`) พนักงานดูรูปแล้วกดยืนยัน
- ตรวจจริง: `SLIP_DRIVER=thunder` + `THUNDER_API_KEY=...` ใน `.env` (ห้าม commit `.env` ห้ามส่ง key ในแชต) · ลองสลิปจริง 1 ใบก่อน แล้วเทียบชื่อผู้รับที่ขึ้นในสลิปกับ `SHOP_PROMPTPAY_NAME`

ถ้าอัปเดตจาก M2 มา M3: รัน `php artisan migrate` (สร้างตาราง bookings, booking_slots, payments) แล้วรัน `php artisan test`
ตั้งเวลาปล่อยช่องที่หมดเวลาชำระ: ตอนพัฒนารัน `php artisan schedule:work` ค้างไว้อีกหน้าต่าง (หรือ `php artisan bookings:expire` ด้วยมือ) · หน้าจองปล่อยช่องที่หมดเวลาให้เองทุกครั้งที่เปิด จึงไม่ติดแม้ยังไม่ได้รัน scheduler

ถ้าอัปเดตจาก M1 มา M2: รัน `php artisan migrate` แล้ว `php artisan db:seed` (สร้างสนาม 1–4 และค่าตั้งต้น ไม่ทับของเดิม)

เปิด http://localhost:8000

## บัญชีสำหรับลอง

| บทบาท | เข้าด้วย | รหัสผ่าน |
|---|---|---|
| Admin | อีเมลจาก `ADMIN_EMAIL` ใน .env | `ADMIN_PASSWORD` (ระบบบังคับเปลี่ยนรหัสตอนเข้าครั้งแรก) → เมนู "ตั้งค่าสนาม" |
| Staff (เฉพาะ local) | `staff@example.com` | `password` |
| ลูกค้า (เฉพาะ local) | เบอร์ `0811111111` หรืออีเมล `customer@example.com` | `password` |

ลูกค้าสมัครเองได้ที่ `/register` · พนักงานเข้าได้เฉพาะด้วยอีเมล (เบอร์ไว้ติดต่อเท่านั้น)

## ลองใช้ทั้งระบบ (ขั้นตอนเดโมสำหรับส่งอาจารย์)

ตั้ง `.env`: `SLIP_DRIVER=thunder` + `THUNDER_API_KEY`, `SHOP_PROMPTPAY_ID`, `SHOP_PROMPTPAY_NAME` แล้ว `php artisan config:clear` · เปิด 2 หน้าต่าง terminal: `php artisan serve` และ `php artisan schedule:work` (ตัวหลังทำงานทุกนาที: ปล่อยช่องหมดเวลา, ไม่มา, เล่นเสร็จ)

1. Admin เข้า `/login` (อีเมล `ADMIN_EMAIL`) → ตั้งรหัสใหม่ → "ตั้งค่าสนาม" ดู/แก้ราคา เวลาเปิด-ปิด เพิ่มวันหยุด (รับเงินออนไลน์แบบโอน + สลิปอย่างเดียว · ตัวเลือก Payment Gateway เอาออกจากหน้านี้แล้ว 2026-10-13)
2. ลูกค้า: สมัครที่ `/register` (เบอร์ + รหัสผ่าน + ติ๊กนโยบาย) → หน้าแรกเลือกวัน/สนาม/1–2 ชม./ช่องเวลา → ดูสรุป → จอง
   - สแกน QR พร้อมเพย์ด้วยแอปธนาคาร โอนจริง → อัปโหลดสลิป → Thunder ตรวจผ่านยืนยันทันที · ตรวจไม่ได้ → "รอพนักงานตรวจ" แล้วพนักงานกด "ยืนยันสลิป" ที่เมนู ตรวจสลิป
   - ปล่อยให้ครบเวลา (ค่าเริ่มต้น 10 นาที + 2) เพื่อดูการหมดเวลา · หมดเวลาครบ 3 ครั้ง/วันจะถูกระงับ
3. พนักงาน (`staff@example.com` / `password` เฉพาะ local): "ตารางสนาม" กดช่องว่าง (+) บันทึก walk-in ใส่เงินสดที่รับแล้วกดบันทึกได้เลย (ไม่มีกะ) → กดการจองเพื่อเปิด drawer: เช็คอิน / ไม่มา / เปลี่ยนสนาม / หมายเหตุ / ดูประวัติ → "ตรวจสลิป" ตัดสินสลิปที่รอ · ยอดเงินสด walk-in ดูที่ "สรุปยอด" (Admin)
4. Admin: ยกเลิกการจอง (ต้องใส่เหตุผล) → "สรุปยอด" รายวัน/รายเดือน (ดูอย่างเดียว) → "ผู้ใช้" รีเซ็ตรหัสลูกค้า ระงับบัญชี เพิ่มพนักงาน

## ทดสอบ

```bat
php artisan test
```

เทสใช้ SQLite ในหน่วยความจำ ไม่แตะฐานข้อมูล MySQL ของคุณ · ผลที่ถูกต้อง: **235 passed** (2026-10-24) · Thunder ทดสอบด้วย `Http::fake` ไม่ยิงจริง

## ลิงก์รีเซ็ตรหัสผ่านตอนยังไม่ตั้งค่าอีเมล

`MAIL_MAILER=log` ในไฟล์ `.env.example` คือไม่ส่งอีเมลจริง ลิงก์จะถูกเขียนลง `storage/logs/laravel.log` (ค้นหา `reset-password`)
ถ้าจะดูเป็นอีเมลจริง ใช้ Mailtrap: ตั้ง `MAIL_MAILER=smtp` แล้วใส่ username/password จากหน้า SMTP Settings ของ Mailtrap

## โครงโค้ด (ดูตามหน้าจอ)

| อยากแก้ | เปิดไฟล์ |
|---|---|
| หน้าเข้าสู่ระบบ / สมัคร / ลืมรหัส | `resources/views/auth/*.blade.php` · `app/Http/Controllers/Auth/*` |
| กฎตรวจข้อมูลตอนสมัคร | `app/Http/Requests/Auth/RegisterRequest.php` |
| วิธีหาบัญชีตอนเข้าระบบ (เบอร์/อีเมล) | `app/Http/Controllers/Auth/LoginController.php` → `login()` |
| ใครทำอะไรได้ (ตารางสิทธิ์) | `database/seeders/RolePermissionSeeder.php` (แก้แล้วรัน `php artisan db:seed --class=RolePermissionSeeder`) |
| ผูกสิทธิ์เข้ากับ `can:` / `@can` | `app/Providers/AppServiceProvider.php` (`Gate::before`) |
| ด่านตรวจ: บัญชีถูกระงับ / บังคับเปลี่ยนรหัส | `app/Http/Middleware/*` ลงทะเบียนใน `bootstrap/app.php` |
| route ทั้งหมด | `routes/web.php` (ทุก route หลังร้านต้องมี `can:...`) |
| สี / ฟอนต์ / โครงหน้า | `resources/views/layouts/*.blade.php` |
| ชื่อสนาม เบอร์สนาม | `.env` → `SHOP_NAME`, `SHOP_PHONE` |
| ข้อความภาษาไทยตอนตรวจข้อมูลผิด | `lang/th/validation.php` |
| หน้าตั้งค่าสนาม (ตารางสนาม · เวลาร้าน · วันหยุดร้าน) | `resources/views/admin/settings/index.blade.php` · `Admin/SettingsController.php` |
| ฟอร์มเพิ่ม/แก้ไขสนาม (ราคา, รายละเอียด, ปิดชั่วคราว) | `resources/views/admin/courts/form.blade.php` · `Admin/CourtController.php` · `Requests/Admin/CourtRequest.php` |
| คิดราคา ก่อน/หลัง 17:00 | `app/Services/PricingService.php` (ตัดที่ `config/booking.php` → `price_cutoff_hour`) |
| การจองที่ชนตอนปิดสนาม/ตั้งวันหยุด | `app/Services/ClosureService.php` (อ่าน `booking_slots`, ขึ้นรหัสการจอง + ชื่อลูกค้า) |
| กติกาการจอง (วันล่วงหน้า, ปิดจอง 15 นาที, ค้างจ่าย 1 รายการ, หมดเวลา 3 ครั้ง/วัน) | `app/Services/BookingService.php` · ค่าใน `config/booking.php` + หน้าตั้งค่าสนาม |
| กันจองซ้ำ | `database/migrations/2026_10_09_000100_create_bookings_tables.php` (UNIQUE `booking_slots_unique_active`) + จับ error ใน `BookingService::create()` |
| ปล่อยช่องที่หมดเวลา | `BookingService::releaseExpired()` · `app/Console/Commands/ExpireBookings.php` · `routes/console.php` (ทุก 1 นาที) |
| หน้าจองลูกค้า (เลือกวัน/สนาม/เวลา) | `app/Http/Controllers/BookingPageController.php` · `resources/views/booking/index.blade.php` · `public/js/booking.js` |
| QR พร้อมเพย์ (ข้อความ + CRC) / รูป QR | `app/Support/PromptPayQr.php` · `app/Support/QrSvg.php` |
| ตรวจสลิป: ตัดสินผ่าน/ไม่ผ่าน, ครั้งสูงสุด, ส่งพนักงาน | `app/Services/SlipService.php` · ค่าใน `config/slip.php` |
| ตัวตรวจสลิป (Thunder · ตัวจำลองเฉพาะตอนเทสต์) | `app/Services/Slip/ThunderSlipVerifier.php` · `FakeSlipVerifier.php` · เลือกใน `AppServiceProvider` |
| ยืนยันว่าจ่ายแล้ว + จ่ายช้า | `BookingService::confirmPaid()` |
| หน้าตรวจสลิปของพนักงาน | `app/Http/Controllers/Admin/PaymentReviewController.php` · `resources/views/admin/payments/index.blade.php` |
| การ์ดการรับเงินออนไลน์ (คำอธิบายอย่างเดียว · ค่าอยู่ใน .env) | `resources/views/admin/settings/index.blade.php` |
| หน้าชำระเงิน / จองสำเร็จ / สถานะ | `app/Http/Controllers/BookingController.php` · `resources/views/booking/{pay,success,_summary}.blade.php` |
| ตารางสนามหลังร้าน + drawer (walk-in / รายละเอียด / ประวัติ) | `Admin/TimelineController.php` · `resources/views/admin/timeline.blade.php` · `public/js/admin.js` (`timelinePage`) |
| walk-in, เช็คอิน, ไม่มา, เปลี่ยนสนาม, ยกเลิก, หมายเหตุ, จ่ายช้า, ปลดระงับ | `app/Services/BookingAdminService.php` · `Admin/BookingAdminController.php` · `Requests/Admin/WalkInRequest.php` |
| ไม่มา / เล่นเสร็จ อัตโนมัติ | `app/Console/Commands/{MarkNoShows,CompleteBookings}.php` · `routes/console.php` |
| ข้อความในส่วน "ประวัติ" | `app/Support/AuditLabels.php` |
| เงินสด walk-in (ไม่มีกะ — ลบ ShiftService/ShiftController/หน้า shift แล้ว 2026-10-21) | `BookingAdminService::walkIn()` → `payments` (provider = cash) · ยอดดูที่ `SummaryService` |
| ข้อมูลสดหลายเครื่อง (ตัวเลขบนเมนู, แจ้งเตือน, โหลดหน้าใหม่เอง) | `app/Support/LiveState.php` · route `admin.live` · `public/js/admin.js › startAdminLive()` · หน้าไหนให้โหลดเองใส่ `@section('live', 'reload')` |
| รายการการจอง + ค้นหา + จ่ายช้า | `BookingAdminController::index()/search()` · `resources/views/admin/bookings/index.blade.php` |
| การจองของฉัน + รหัสเช็คอินเต็มจอ | `app/Http/Controllers/MyBookingsController.php` · `resources/views/my-bookings/*` |
| สรุปยอด (ดูอย่างเดียว ไม่มี Excel/พิมพ์) | `app/Services/SummaryService.php` · `Admin/SummaryController.php` · `resources/views/admin/summary/index.blade.php` |
| ผู้ใช้: ลูกค้า (รีเซ็ตรหัส/แก้/ระงับ) และพนักงาน (เพิ่ม/ตำแหน่ง/ลบ) | `app/Services/UserManagementService.php` · `Admin/UserController.php` · `resources/views/admin/users/index.blade.php` |
| ค่าตั้งต้นเวลาร้าน ราคา | `config/booking.php` · `database/seeders/SettingSeeder.php` · `CourtSeeder.php` |
| JS ของหน้าหลังร้าน (Alpine) | `public/js/admin.js` |

## ดีพลอยขึ้นเซิร์ฟเวอร์จริง (รายการตรวจ)

1. เซิร์ฟเวอร์: PHP 8.2+ (extension: mbstring, openssl, pdo_mysql, zip, ctype, fileinfo, xml, curl), MySQL 8, Composer, เว็บเซิร์ฟเวอร์ชี้ document root ไปที่โฟลเดอร์ `public/` (ห้ามชี้ที่รากโปรเจกต์)
2. `composer install --no-dev --optimize-autoloader` · คัดลอก `.env.example` เป็น `.env` แล้วตั้ง: `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL=https://โดเมนจริง`, `APP_KEY` (`php artisan key:generate`), `DB_*`, `MAIL_*` (SMTP จริง), `SHOP_NAME`, `SHOP_PHONE`, `ADMIN_*`
3. `php artisan migrate --force` · `php artisan db:seed --force` (สร้างบทบาท/สิทธิ์/สนามตัวอย่าง/Admin คนแรก — สนามตัวอย่างให้แก้หรือลบในหน้าตั้งค่า) · `php artisan config:cache && php artisan route:cache && php artisan view:cache` · `php artisan storage:link`
4. **ตั้งเวลาให้ scheduler ทำงานทุกนาที (สำคัญมาก ไม่ตั้ง = ช่องที่หมดเวลาไม่ถูกปล่อย):** Linux cron `* * * * * cd /path/to/app && php artisan schedule:run >> /dev/null 2>&1` · Windows Task Scheduler: งานรันทุก 1 นาที `php.exe artisan schedule:run` โดยตั้ง Start in เป็นโฟลเดอร์โปรเจกต์
5. เปลี่ยนจากโหมดทดสอบเป็นของจริง: `SLIP_DRIVER=thunder` + `THUNDER_API_KEY` · `SHOP_PROMPTPAY_ID` / `SHOP_PROMPTPAY_NAME` ให้ตรงชื่อที่ขึ้นบนสลิปจริง · ระบบตรวจด้วย Thunder เสมอ (ไม่มี key = ทุกใบไปรอพนักงานยืนยัน)
6. ทดสอบก่อนเปิดให้ลูกค้า: สแกน QR ด้วยแอปธนาคารจริงว่ายอดและชื่อถูก · ส่งสลิปจริง 1 ใบ · จองบนมือถือจริง (iOS/Android) · ตรวจว่า `storage/` และ `bootstrap/cache/` เขียนได้ · สำรองฐานข้อมูลสม่ำเสมอ (`mysqldump`)
7. ก่อนใช้จริง: เปลี่ยนรหัส Admin, ลบบัญชีตัวอย่างถ้าเผลอ seed บน production, ให้เจ้าของสนามตรวจข้อความนโยบายความเป็นส่วนตัว, ใส่ชื่อ/เบอร์สนามจริง, เปลี่ยน `public/images/logo.png`

## หมายเหตุความปลอดภัย

- ห้าม commit ไฟล์ `.env` (อยู่ใน `.gitignore` แล้ว) — มี key ของ Thunder (อย่าส่ง key ในแชตหรือโพสต์ที่ไหน)
- ข้อความนโยบายความเป็นส่วนตัวใน `resources/views/partials/policy-sheet.blade.php` เป็นร่าง ให้เจ้าของสนามตรวจก่อนใช้จริง

# CAB System - API Documentation Guide

## Cau truc thu muc

`openapi.yaml` la file NGUON (modular, dung `$ref` tro vao `paths/` va `components/`).
`swagger-final.yaml` la file BUNDLE (da gop het `$ref` thanh 1 file phang) - dung file
nay de paste nhanh vao Swagger Editor online. Cac file trong `paths/` va `components/`
chi la fragment, KHONG the mo/test doc lap (xem muc "Test file .yaml rieng le" o duoi).

```
api-docs/
|-- openapi.yaml              <- FILE NGUON: sua o day, dung $ref toi paths/ + components/
|-- swagger-final.yaml        <- FILE BUNDLE (auto-generate): paste vao Swagger UI de test nhanh
|-- paths/
|   |-- auth/
|   |   |-- register.yaml     <- POST /auth/register
|   |   |-- login.yaml        <- POST /auth/login
|   |   `-- logout.yaml       <- POST /auth/logout
|   |-- customers/
|   |   `-- me.yaml           <- GET/PATCH /customers/me
|   |-- drivers/
|   |   |-- me.yaml           <- GET/PATCH /drivers/me
|   |   |-- availability.yaml <- PATCH /drivers/me/availability
|   |   `-- location.yaml     <- PUT /drivers/me/location
|   |-- trips/
|   |   |-- trips.yaml        <- POST/GET /trips
|   |   |-- trip-detail.yaml  <- GET /trips/{tripId}
|   |   |-- trip-status.yaml  <- PATCH /trips/{tripId}/status
|   |   |-- trip-accept.yaml  <- POST /trips/{tripId}/accept
|   |   |-- trip-reject.yaml  <- POST /trips/{tripId}/reject
|   |   `-- trip-review.yaml  <- POST /trips/{tripId}/review
|   |-- payments/
|   |   |-- payment.yaml      <- GET/POST /payments/{tripId}
|   |   `-- payment-retry.yaml<- POST /payments/{tripId}/retry
|   |-- notifications/
|   |   `-- notifications.yaml<- GET /notifications
|   `-- admin/
|       |-- users.yaml        <- GET /admin/users
|       |-- user-detail.yaml  <- GET/PATCH /admin/users/{userId}
|       |-- trips.yaml        <- GET /admin/trips
|       |-- trip-cancel.yaml  <- POST /admin/trips/{tripId}/cancel
|       |-- reports.yaml      <- GET /admin/reports
|       `-- logs.yaml         <- GET /admin/logs
`-- components/
    |-- schemas/
    |   `-- _index.yaml       <- Customer, Driver, Vehicle, Location, Trip, Payment,
    |                             Review, Notification, AuditLog, ErrorResponse
    |-- responses/
    |   `-- _index.yaml       <- Unauthorized, Forbidden, NotFound, BadRequest
    `-- securitySchemes/
        `-- _index.yaml       <- BearerAuth (JWT)
```

## Sua API va tao lai file bundle

1. Chi sua noi dung trong `openapi.yaml`, `paths/**/*.yaml` hoac `components/**/*.yaml`
   (KHONG sua tay `swagger-final.yaml`, file nay se bi ghi de).
2. Kiem tra cu phap + $ref hop le:
   ```bash
   npx @redocly/cli@latest lint openapi.yaml
   ```
3. Gop lai (bundle) thanh 1 file phang de test/nop bai:
   ```bash
   npx @redocly/cli@latest bundle openapi.yaml -o swagger-final.yaml --ext yaml
   ```

## Test file .yaml rieng le (paths/*.yaml) - VI SAO KHONG PASTE TRUC TIEP DUOC

Cac file trong `paths/` (VD `paths/auth/login.yaml`) chi la **fragment** - noi dung chi la
mot Path Item Object (`post:`/`get:`...), khong co khai bao `openapi: 3.0.x` o dau file va
co dung `$ref` tuong doi (`../../components/...`) chi hoat dong khi duoc nap tu `openapi.yaml`.
Vi vay dan rieng 1 file trong `paths/` vao Swagger Editor se luon bao loi
"does not specify a valid version field". Muon test, luon dung `openapi.yaml` (sau khi
`redocly bundle`) hoac `swagger-final.yaml` co san.

## Huong dan su dung voi VS Code

### Cach 1: Dung Swagger Viewer Extension (Don gian nhat)

1. Mo VS Code, cai extension: **Swagger Viewer** (Arjun G)
   hoac **OpenAPI (Swagger) Editor** (42Crunch)

2. Mo file `api-docs/openapi.yaml`

3. Bam Ctrl+Shift+P -> chon **Preview Swagger**
   hoac bam nut Preview (icon mat kinh) o goc tren phai

4. Swagger UI hien thi ngay trong VS Code!

### Cach 2: Up len Swagger Editor Online

1. Truy cap: https://editor.swagger.io
2. Copy toan bo noi dung file `openapi.yaml`
3. Dan vao khung ben trai -> Swagger UI tu dong hien thi ben phai
4. Co the test truc tiep cac API tren do

### Cach 3: Dung Swagger UI qua Docker (Local Server)

```bash
docker run -p 8081:8080 -e SWAGGER_JSON=/api/openapi.yaml \
  -v ./api-docs:/api swaggerapi/swagger-ui
# Truy cap: http://localhost:8081
```

## Danh sach API Endpoints (27 endpoints)

| Method | Path | Mo ta | FR/BR |
|--------|------|-------|-------|
| POST | /auth/register | Dang ky tai khoan | FR-CUS-01, BR-01 |
| POST | /auth/login | Dang nhap - nhan JWT | FR-CUS-01, BR-31 |
| POST | /auth/logout | Dang xuat | - |
| GET | /customers/me | Xem ho so khach hang | FR-CUS-02 |
| PATCH | /customers/me | Cap nhat ho so | FR-CUS-02 |
| GET | /drivers/me | Xem ho so tai xe | FR-DRV-01 |
| PATCH | /drivers/me | Cap nhat ho so | FR-DRV-01 |
| PATCH | /drivers/me/availability | Bat/tat san sang | FR-DRV-02, BR-07 |
| PUT | /drivers/me/location | Cap nhat GPS (~3s/lan) | FR-DRV-05, BR-10 |
| POST | /trips | Dat xe moi | FR-CUS-03, FR-SYS-01 |
| GET | /trips | Lich su chuyen di | FR-CUS-07 |
| GET | /trips/{tripId} | Chi tiet + Tracking | FR-CUS-05 |
| PATCH | /trips/{tripId}/status | Cap nhat trang thai | FR-DRV-04 |
| POST | /trips/{tripId}/accept | Tai xe chap nhan | FR-DRV-03 |
| POST | /trips/{tripId}/reject | Tai xe tu choi | FR-SYS-02 |
| POST | /trips/{tripId}/review | Danh gia tai xe | FR-CUS-08 |
| GET | /payments/{tripId} | Xem hoa don | FR-SYS-04 |
| POST | /payments/{tripId} | Thanh toan | FR-SYS-05 |
| POST | /payments/{tripId}/retry | Thu lai thanh toan | BR-17 |
| GET | /notifications | Danh sach thong bao | FR-SYS-06 |
| GET | /admin/users | DS nguoi dung | FR-ADM-01 |
| GET | /admin/users/{userId} | Chi tiet nguoi dung | FR-ADM-01 |
| PATCH | /admin/users/{userId} | Duyet ho so / Khoa / Mo khoa | FR-ADM-01 |
| GET | /admin/trips | Giam sat chuyen | FR-ADM-02 |
| POST | /admin/trips/{tripId}/cancel | Huy khan cap | FR-ADM-02 |
| GET | /admin/reports | Bao cao thong ke | FR-ADM-05 |
| GET | /admin/logs | Audit log | FR-ADM-06 |

## Ghi chu ve Bao mat

- Tat ca API (tru /auth/register va /auth/login) yeu cau: `Authorization: Bearer <JWT_TOKEN>`
- Phan quyen: Customer < Driver < Operator < Admin
- Thao tac Admin duoc luu vao Audit Log (RULE-10)
- Khong luu thong tin the thanh toan trong he thong (RULE-04)

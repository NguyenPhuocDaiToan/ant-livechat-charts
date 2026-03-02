# Ant-Livechat Helm Chart

![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.0.0](https://img.shields.io/badge/AppVersion-1.0.0-informational?style=flat-square)

Biểu đồ Helm này được thiết kế để triển khai hệ thống **Ant-Livechat** (nền tảng BeeIQ) lên cụm Kubernetes của bạn một cách nhanh chóng và tối ưu.

---

## 🚀 Triển khai nhanh

### 1. Thêm Helm Repository của AntBuddy
```bash
helm repo add antbuddy https://antbuddy.github.io/ant-livechat-charts/
helm repo update
```

### 2. Cài đặt Chart
Để hệ thống hoạt động, bạn cần cung cấp ít nhất thông tin Database và Secret Key. Thay thế các giá trị dưới đây bằng thông tin thực tế của bạn:

```bash
helm install ant-livechat antbuddy/ant-livechat \
    --set secrets.SECRET_KEY_BASE="tạo_một_chuỗi_ngẫu_nhiên_64_ký_tự" \
    --set secrets.DATABASE_URL="ecto://postgres:password@your-db-host:5432/ant_livechat_db" \
    --set configMap.BACKEND_URL="https://chat.yourdomain.com"
```

---

## 📋 Yêu cầu tiên quyết (Prerequisites)

### Cơ sở dữ liệu (PostgreSQL)
Hệ thống yêu cầu PostgreSQL phiên bản 12 trở lên. Bạn có thể sử dụng database có sẵn hoặc cài đặt mới thông qua gói Bitnami:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install ant-livechat-db bitnami/postgresql \
    --set postgresqlUsername=antbuddy \
    --set postgresqlPassword=antbuddy_pass \
    --set postgresqlDatabase=ant_livechat_db
```

---

## ⚙️ Các tham số cấu hình chính (Values)

Bạn có thể tùy chỉnh cấu hình thông qua file `values.yaml`:

| Khóa (Key) | Mặc định | Mô tả |
|:---|:---|:---|
| `replicaCount` | `1` | Số lượng Pod chạy đồng thời. |
| `image.repository` | `antbuddyinc/ant-livechat` | Docker Image chính thức của AntBuddy. |
| `image.tag` | `prod_release1.0.0` | Tag phiên bản sản phẩm. |
| `service.port` | `4000` | Cổng nội bộ của ứng dụng. |
| `ingress.enabled` | `false` | Bật/Tắt truy cập từ domain bên ngoài. |
| `ingress.hosts[0].host` | `chat.beeiq.vn` | Domain chính thức của hệ thống. |
| `configMap.BACKEND_URL` | `https://chat.beeiq.vn` | URL gốc xử lý WebSocket và Assets. |
| `secrets.SECRET_KEY_BASE` | `dvPPvOjpg...` | Khóa bảo mật Phoenix (Bắt buộc đổi khi chạy Prod). |
| `secrets.DATABASE_URL` | `ecto://...` | Đường dẫn kết nối Database PostgreSQL. |
| `migration.enabled` | `true` | Tự động cập nhật Database khi cài đặt/nâng cấp. |
| `initialize_database.enabled` | `true` | Tự động tạo cấu trúc DB nếu chưa có. |

---

## ⚠️ Lưu ý quan trọng

### 1. Hỗ trợ WebSocket (Real-time)
Để khung chat hoạt động thời gian thực, Ingress Controller của bạn (ví dụ Nginx) cần được cấu hình để không ngắt kết nối WebSocket. Các annotation sau đã được tích hợp sẵn:
- `nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"`
- `nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"`

### 2. Bảo mật (Security)
Tuyệt đối không sử dụng `SECRET_KEY_BASE` mặc định cho môi trường Production. Bạn có thể tạo chuỗi ngẫu nhiên bằng lệnh:
```bash
openssl rand -base64 64
```

---

## 🌐 Istio Integration

### Thêm vào values.yaml:

  istio:
    enabled: false
    gateway: "longchau-gateway"
    hosts:
      - "chat-sdk-omni.frt.vn"

### Thêm file templates/virtualservice.yaml:

  (Đã tạo sẵn, bạn chỉ cần chỉnh sửa nếu muốn route nâng cao)

### Khi deploy cho Long Châu, dùng lệnh:

  helm install ... --set istio.enabled=true --set ingress.enabled=false

### Nếu muốn tuỳ chỉnh gateway hoặc hosts, sửa trong values hoặc dùng --set.

### Đảm bảo cluster đã có Istio và gateway đúng tên.

---

**Phát triển và quản lý bởi Thông Nguyễn - AntBuddy Team.** *Mọi chi tiết xin liên hệ: [thong.nguyen@antbuddy.com](mailto:thong.nguyen@antbuddy.com)*
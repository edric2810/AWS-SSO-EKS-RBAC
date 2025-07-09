
# 🔐 Luồng Xác Thực & Phân Quyền RBAC trong Amazon EKS với AWS SSO

Dưới đây là luồng chuẩn mô tả quá trình **xác thực và phân quyền người dùng AWS SSO** khi truy cập vào **Kubernetes EKS Cluster** sử dụng RBAC:

---

## 📋 Sơ đồ luồng hoạt động

```
┌────────────────────────────┐
│     1. Đăng nhập AWS SSO   │
│   (aws sso login)          │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ 2. Nhận temporary IAM Role │
│    (assumed-role từ SSO)   │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────────────────┐
│ 3. Gửi yêu cầu đến EKS (kubectl/eksctl)│
│    dùng IAM Role làm identity          │
└────────────┬───────────────────────────┘
             │
             ▼
┌────────────────────────────────────────┐
│ 4. API Server tra `aws-auth` ConfigMap │
│   để map IAM Role → username + groups  │
└────────────┬───────────────────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│ 5. Kiểm tra RBAC (RoleBinding/CRB): │
│    - Có group `system:masters`?     │
│    → ✅ Full quyền (`cluster-admin`)│
│    - Không có?                      │
│    → ❌ Chỉ có quyền nếu được RBAC  │
│         gán riêng                   │
└─────────────────────────────────────┘
             │
             ▼
┌──────────────────────────────┐
│ 6. Cho phép hoặc từ chối     │
│    truy cập theo RBAC        │
└──────────────────────────────┘
```

---

## ✅ Vai trò các thành phần chính

| Thành phần        | Vai trò |
|-------------------|--------|
| **AWS SSO**           | Xác thực người dùng (Authentication) |
| **IAM Role**          | Danh tính IAM sau khi đăng nhập |
| **aws-auth ConfigMap**| Map IAM Role → Kubernetes username/group |
| **system:masters**    | Nhóm đặc biệt, có full quyền |
| **RBAC**              | Kiểm soát ai được làm gì trong cụm |
| **API Server**        | Thực thi xác thực và phân quyền |

---

## 📌 Ghi chú quan trọng

- `aws-auth` là trung gian bắt buộc giữa IAM Role và RBAC.
- Nếu IAM Role được gán group `system:masters` → có toàn quyền, kể cả khi không có `ClusterRoleBinding`.
- Nếu không, bạn cần tự gán `RoleBinding` hoặc `ClusterRoleBinding` tương ứng.

---

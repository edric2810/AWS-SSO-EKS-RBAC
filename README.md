┌────────────────────────────┐
│     1. Đăng nhập AWS SSO   │
│   (aws sso login)          │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ 2. Nhận temporary IAM Role │
│    (assumed-role từ SSO)  │
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
│      → ✅ Full quyền (`cluster-admin`)│
│    - Không có?                      │
│      → ❌ Chỉ có quyền nếu được RBAC │
│           gán riêng                 │
└─────────────────────────────────────┘
             │
             ▼
┌──────────────────────────────┐
│ 6. Cho phép hoặc từ chối     │
│    truy cập theo RBAC        │
└──────────────────────────────┘

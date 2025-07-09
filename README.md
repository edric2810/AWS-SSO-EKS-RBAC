# 🔐 Tổng Quan & Luồng Chi Tiết RBAC Trong Kubernetes (Kèm Group)

Tài liệu này cung cấp kiến thức nền tảng và quy trình chi tiết về **Role-Based Access Control (RBAC)** trong Kubernetes, đặc biệt khi sử dụng **Group** để phân quyền (qua AWS SSO, OIDC, IAM...).

---

## 📘 1. Giới thiệu về RBAC

**RBAC (Role-Based Access Control)** là cơ chế phân quyền trong Kubernetes cho phép bạn kiểm soát **ai được làm gì với tài nguyên nào**.

RBAC sử dụng 4 đối tượng chính:

| Thành phần       | Mô tả |
|------------------|------|
| `Role`           | Tập hợp các quyền (verbs như `get`, `list`, `create`,...) giới hạn trong **một namespace**. |
| `ClusterRole`    | Tập hợp các quyền trên **toàn cụm** (cluster-wide). Có thể áp dụng cho cả namespace hoặc tài nguyên cụm. |
| `RoleBinding`    | Gán `Role` cho `User`/`Group`/`ServiceAccount` trong **namespace cụ thể**. |
| `ClusterRoleBinding` | Gán `ClusterRole` cho `User`/`Group`/`ServiceAccount` trên **toàn cụm**. |

---

## 🧱 2. Mối quan hệ giữa các thành phần

```
[User / Group / ServiceAccount]
           │
           ▼
  [RoleBinding / ClusterRoleBinding]
           │
           ▼
    [Role / ClusterRole]
           │
           ▼
     [Quyền truy cập tài nguyên]
```

---

## 🧩 3. Luồng RBAC kết hợp Group (Authentication → Authorization)

```
┌────────────────────────────┐
│   Người dùng đăng nhập     │
│  (VD: AWS SSO / OIDC SSO)  │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│  Nhận IAM Role + thông tin │
│     group (claim) từ IdP   │
└────────────┬───────────────┘
             ▼
┌─────────────────────────────────────────────┐
│  Gửi request đến Kubernetes API Server      │
│  → truyền identity: username + group        │
└────────────┬────────────────────────────────┘
             ▼
┌──────────────────────────────────────────────┐
│  RBAC Authorizer tra RoleBinding/CRBinding   │
│  với subject là user hoặc group              │
└────────────┬─────────────────────────────────┘
             ▼
┌──────────────────────────────────────────────┐
│  Kiểm tra Role/ClusterRole có verb + resource│
│  phù hợp yêu cầu không                       │
└────────────┬─────────────────────────────────┘
     ┌───────┴────────┐
     ▼                ▼
┌─────────────┐   ┌────────────┐
│ ✅ Cho phép │   │ ❌ Từ chối │
└─────────────┘   └────────────┘
```

---

## ✅ 4. Ví dụ: Gán quyền qua Group

### 🎯 Mục tiêu: Group `monitor_admin` được xem mọi thứ (cluster-wide)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitor-admin-binding
subjects:
- kind: Group
  name: monitor_admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔍 5. Kiểm tra quyền

- Kiểm tra quyền của user hiện tại:
```bash
kubectl auth can-i get pods
```

- Giả lập user khác:
```bash
kubectl auth can-i get pods --as=alice
```

- Kubernetes **không cho phép --as-group**, nhưng RBAC vẫn dùng `subjects.kind: Group`.

---

## 🧠 6. Lưu ý khi dùng Group trong RBAC

- **Kubernetes không quản lý user/group** – nó chỉ **nhận thông tin từ hệ thống xác thực (SSO/OIDC)**.
- RBAC chỉ thực hiện phân quyền dựa trên `subject.kind: User|Group`.
- Không thể `kubectl get users` hoặc `get groups`.
- Hệ thống xác thực quyết định user thuộc group nào.

---

## 📌 Tổng kết

| Bước | Diễn giải |
|------|-----------|
| 1. Xác thực | AWS SSO / OIDC cung cấp thông tin user + group |
| 2. `aws-auth` hoặc webhook | Map IAM Role/OIDC → Kubernetes group |
| 3. RBAC | Dùng RoleBinding hoặc ClusterRoleBinding để phân quyền |
| 4. Role / ClusterRole | Định nghĩa rõ ai được làm gì với resource nào |

---

## 📎 Tài liệu tham khảo

- [Kubernetes RBAC Docs](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [AWS EKS aws-auth configmap](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)

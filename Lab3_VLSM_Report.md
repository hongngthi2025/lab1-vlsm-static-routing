# BÁO CÁO BÀI THỰC HÀNH SỐ 1
# Variable Length Subnet Mask và Định tuyến tĩnh

---

## THÔNG TIN SINH VIÊN

| STT | Họ và Tên | MSSV |
|-----|-----------|------|
| 1   | [Họ tên SV1] | [MSSV1] |
| 2   | [Họ tên SV2] | [MSSV2] |
| 3   | [Họ tên SV3] | [MSSV3] |

**Mã lớp:** NT132.M11.ANTT.1

---

## 1. MỤC TIÊU BÀI LAB

- Làm quen và sử dụng phần mềm Packet Tracer để giả lập mô hình mạng
- Hiểu rõ cách chia địa chỉ IP với số host không bằng nhau (VLSM)
- Xây dựng mô hình mạng cơ bản và cấu hình thiết bị Cisco
- Hiểu nguyên tắc định tuyến và cấu hình định tuyến tĩnh với đường chính và đường dự phòng

---

## 2. SƠ ĐỒ MẠNG (TOPOLOGY)

> **📌 Hướng dẫn:** Chèn ảnh chụp topology từ Packet Tracer vào đây

**Mô tả topology:**
- **Nhánh 1:** PC-A (100 hosts) nối với R1
- **Nhánh 2:** ServerB1 (10 hosts) nối với R3
- **Nhánh 3:** ServerB2 (15 hosts) nối với R3
- **Kết nối Router:** R1 ↔ R2, R1 ↔ R4, R2 ↔ R3, R4 ↔ R3 (tạo thành vòng tròn)

---

## 3. CHIA ĐỊA CHỈ IP (VLSM)

### Yêu cầu 1: Phân chia mạng con

**Địa chỉ gốc:** `10.81.x.0/24` (với x là số thứ tự nhóm)

**Ví dụ:** Sử dụng `10.81.1.0/24` cho nhóm 1

**Các subnet cần chia:**
- LAN PC-A: 100 hosts
- LAN ServerB2: 15 hosts  
- LAN ServerB1: 10 hosts
- WAN R1-R2: 2 hosts
- WAN R1-R4: 2 hosts
- WAN R2-R3: 2 hosts
- WAN R4-R3: 2 hosts

### Bảng phân chia Subnet (VLSM)

| Số hosts | Network | Subnet Mask | Dải IP | Broadcast |
|----------|---------|-------------|--------|-----------|
| 100 | 10.81.1.0/25 | 255.255.255.128 | 10.81.1.1 - 10.81.1.126 | 10.81.1.127 |
| 15 | 10.81.1.128/27 | 255.255.255.224 | 10.81.1.129 - 10.81.1.158 | 10.81.1.159 |
| 10 | 10.81.1.160/28 | 255.255.255.240 | 10.81.1.161 - 10.81.1.174 | 10.81.1.175 |
| 2 (WAN R1-R2) | 10.81.1.176/30 | 255.255.255.252 | 10.81.1.177 - 10.81.1.178 | 10.81.1.179 |
| 2 (WAN R1-R4) | 10.81.1.180/30 | 255.255.255.252 | 10.81.1.181 - 10.81.1.182 | 10.81.1.183 |
| 2 (WAN R2-R3) | 10.81.1.184/30 | 255.255.255.252 | 10.81.1.185 - 10.81.1.186 | 10.81.1.187 |
| 2 (WAN R4-R3) | 10.81.1.188/30 | 255.255.255.252 | 10.81.1.189 - 10.81.1.190 | 10.81.1.191 |

**Giải thích cách tính:**
- 100 hosts → 2^7 - 2 = 126 hosts > 100 → cần 7 bits host → /25
- 15 hosts → 2^5 - 2 = 30 hosts > 15 → cần 5 bits host → /27
- 10 hosts → 2^4 - 2 = 14 hosts > 10 → cần 4 bits host → /28
- WAN (2 hosts) → 2^2 - 2 = 2 hosts → cần 2 bits host → /30

---

### Yêu cầu 2: Bảng địa chỉ IP các thiết bị

| Thiết bị | Interface | Địa chỉ IP | Subnet Mask | Default Gateway |
|----------|-----------|------------|-------------|-----------------|
| **R1** | G0/0/1 | 10.81.1.1 | 255.255.255.128 | N/A |
|  | S0/1/0 | 10.81.1.177 | 255.255.255.252 | N/A |
|  | S0/1/1 | 10.81.1.181 | 255.255.255.252 | N/A |
| **R2** | S0/1/0 | 10.81.1.178 | 255.255.255.252 | N/A |
|  | S0/1/1 | 10.81.1.185 | 255.255.255.252 | N/A |
| **R3** | G0/0/0 | 10.81.1.161 | 255.255.255.240 | N/A |
|  | G0/0/1 | 10.81.1.129 | 255.255.255.224 | N/A |
|  | S0/1/0 | 10.81.1.186 | 255.255.255.252 | N/A |
|  | S0/1/1 | 10.81.1.190 | 255.255.255.252 | N/A |
| **R4** | S0/1/0 | 10.81.1.182 | 255.255.255.252 | N/A |
|  | S0/1/1 | 10.81.1.189 | 255.255.255.252 | N/A |
| **PC-A** | NIC | 10.81.1.126 | 255.255.255.128 | 10.81.1.1 |
| **ServerB1** | NIC | 10.81.1.174 | 255.255.255.240 | 10.81.1.161 |
| **ServerB2** | NIC | 10.81.1.158 | 255.255.255.224 | 10.81.1.129 |

---

## 4. CẤU HÌNH CƠ BẢN CHO THIẾT BỊ (Yêu cầu 3)

### 4.1 Cấu hình Router R1

```
Router> enable
Router# configure terminal
Router(config)# hostname R1

! Cấu hình mật khẩu
R1(config)# enable password cisco
R1(config)# service password-encryption

! Cấu hình console
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit

! Cấu hình telnet
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 4.2 Cấu hình Router R2

```
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# enable password cisco
R2(config)# service password-encryption
R2(config)# line console 0
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# exit
R2(config)# line vty 0 4
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# exit
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 4.3 Cấu hình Router R3

```
Router> enable
Router# configure terminal
Router(config)# hostname R3
R3(config)# enable password cisco
R3(config)# service password-encryption
R3(config)# line console 0
R3(config-line)# password cisco
R3(config-line)# login
R3(config-line)# exit
R3(config)# line vty 0 4
R3(config-line)# password cisco
R3(config-line)# login
R3(config-line)# exit
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 4.4 Cấu hình Router R4

```
Router> enable
Router# configure terminal
Router(config)# hostname R4
R4(config)# enable password cisco
R4(config)# service password-encryption
R4(config)# line console 0
R4(config-line)# password cisco
R4(config-line)# login
R4(config-line)# exit
R4(config)# line vty 0 4
R4(config-line)# password cisco
R4(config-line)# login
R4(config-line)# exit
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

## 5. CẤU HÌNH INTERFACE (Yêu cầu 4)

### 5.1 Cấu hình Interface trên R1

```
R1(config)# interface GigabitEthernet 0/0/1
R1(config-if)# ip address 10.81.1.1 255.255.255.128
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface Serial 0/1/0
R1(config-if)# ip address 10.81.1.177 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface Serial 0/1/1
R1(config-if)# ip address 10.81.1.181 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Kiểm tra:**
```
R1# show ip interface brief
```

> **📌 Screenshot:** Chèn ảnh kết quả `show ip interface brief` trên R1

---

### 5.2 Cấu hình Interface trên R2

```
R2(config)# interface Serial 0/1/0
R2(config-if)# ip address 10.81.1.178 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# interface Serial 0/1/1
R2(config-if)# ip address 10.81.1.185 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
```

**Kiểm tra:**
```
R2# show ip interface brief
```

> **📌 Screenshot:** Chèn ảnh kết quả `show ip interface brief` trên R2

---

### 5.3 Cấu hình Interface trên R3

```
R3(config)# interface GigabitEthernet 0/0/0
R3(config-if)# ip address 10.81.1.161 255.255.255.240
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface GigabitEthernet 0/0/1
R3(config-if)# ip address 10.81.1.129 255.255.255.224
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface Serial 0/1/0
R3(config-if)# ip address 10.81.1.186 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface Serial 0/1/1
R3(config-if)# ip address 10.81.1.190 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
```

**Kiểm tra:**
```
R3# show ip interface brief
```

> **📌 Screenshot:** Chèn ảnh kết quả `show ip interface brief` trên R3

---

### 5.4 Cấu hình Interface trên R4

```
R4(config)# interface Serial 0/1/0
R4(config-if)# ip address 10.81.1.182 255.255.255.252
R4(config-if)# no shutdown
R4(config-if)# exit

R4(config)# interface Serial 0/1/1
R4(config-if)# ip address 10.81.1.189 255.255.255.252
R4(config-if)# no shutdown
R4(config-if)# exit
```

**Kiểm tra:**
```
R4# show ip interface brief
```

> **📌 Screenshot:** Chèn ảnh kết quả `show ip interface brief` trên R4

---

### 5.5 Cấu hình IP cho PC và Server

**PC-A:**
- IP Address: 10.81.1.126
- Subnet Mask: 255.255.255.128
- Default Gateway: 10.81.1.1

> **📌 Screenshot:** Chèn ảnh cấu hình IP PC-A

**ServerB1:**
- IP Address: 10.81.1.174
- Subnet Mask: 255.255.255.240
- Default Gateway: 10.81.1.161

> **📌 Screenshot:** Chèn ảnh cấu hình IP ServerB1

**ServerB2:**
- IP Address: 10.81.1.158
- Subnet Mask: 255.255.255.224
- Default Gateway: 10.81.1.129

> **📌 Screenshot:** Chèn ảnh cấu hình IP ServerB2

---

## 6. CẤU HÌNH ĐỊNH TUYẾN TĨNH (Yêu cầu 5)

### Yêu cầu đường đi:
- **Đường chính:** PC-A → R1 → R2 → R3 → ServerB1, ServerB2
- **Đường dự phòng:** PC-A → R1 → R4 → R3 → ServerB1, ServerB2

### 6.1 Cấu hình Static Route trên R1

```
! Đường chính qua R2 (AD mặc định = 1)
R1(config)# ip route 10.81.1.160 255.255.255.240 10.81.1.178
R1(config)# ip route 10.81.1.128 255.255.255.224 10.81.1.178

! Đường dự phòng qua R4 (AD = 10)
R1(config)# ip route 10.81.1.160 255.255.255.240 10.81.1.182 10
R1(config)# ip route 10.81.1.128 255.255.255.224 10.81.1.182 10
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 6.2 Cấu hình Static Route trên R2

```
! Route đến các mạng LAN qua R3
R2(config)# ip route 10.81.1.160 255.255.255.240 10.81.1.186
R2(config)# ip route 10.81.1.128 255.255.255.224 10.81.1.186

! Route đến mạng PC-A qua R1
R2(config)# ip route 10.81.1.0 255.255.255.128 10.81.1.177
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 6.3 Cấu hình Static Route trên R3

```
! Đường chính về PC-A qua R2 (AD mặc định = 1)
R3(config)# ip route 10.81.1.0 255.255.255.128 10.81.1.185

! Đường dự phòng về PC-A qua R4 (AD = 10)
R3(config)# ip route 10.81.1.0 255.255.255.128 10.81.1.189 10
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 6.4 Cấu hình Static Route trên R4

```
! Route đến các mạng LAN qua R3
R4(config)# ip route 10.81.1.160 255.255.255.240 10.81.1.190
R4(config)# ip route 10.81.1.128 255.255.255.224 10.81.1.190

! Route đến mạng PC-A qua R1
R4(config)# ip route 10.81.1.0 255.255.255.128 10.81.1.181
```

> **📌 Screenshot:** Chèn ảnh minh chứng

---

### 6.5 Lưu cấu hình

```
R1# copy running-config startup-config
R2# copy running-config startup-config
R3# copy running-config startup-config
R4# copy running-config startup-config
```

---

## 7. KIỂM TRA VÀ XÁC NHẬN

### 7.1 Kiểm tra Routing Table

**R1:**
```
R1# show ip route
```
> **📌 Screenshot:** Chèn ảnh kết quả

**R3:**
```
R3# show ip route
```
> **📌 Screenshot:** Chèn ảnh kết quả

---

### 7.2 Kiểm tra kết nối đường chính

**Từ PC-A ping đến ServerB1:**
```
C:\> ping 10.81.1.174
```
Kết quả mong đợi: Reply from 10.81.1.174

> **📌 Screenshot:** Chèn ảnh kết quả ping

**Từ PC-A tracert đến ServerB1:**
```
C:\> tracert 10.81.1.174
```
Kết quả mong đợi:
```
1   10.81.1.1      (R1)
2   10.81.1.178    (R2)
3   10.81.1.186    (R3)
4   10.81.1.174    (ServerB1)
```

> **📌 Screenshot:** Chèn ảnh kết quả tracert (đường chính qua R2)

---

### 7.3 Kiểm tra đường dự phòng

**Bước 1:** Tắt Router R2

**Bước 2:** Từ PC-A tracert đến ServerB1:
```
C:\> tracert 10.81.1.174
```
Kết quả mong đợi (đi qua R4):
```
1   10.81.1.1      (R1)
2   10.81.1.182    (R4)
3   10.81.1.190    (R3)
4   10.81.1.174    (ServerB1)
```

> **📌 Screenshot:** Chèn ảnh kết quả tracert (đường dự phòng qua R4)

---

## 8. KẾT LUẬN

- ✅ Đã hoàn thành chia địa chỉ IP bằng VLSM cho 7 subnet
- ✅ Đã cấu hình hostname và password cho 4 router (R1, R2, R3, R4)
- ✅ Đã cấu hình IP address cho tất cả các interface
- ✅ Đã cấu hình Static Routing với đường chính (qua R2) và đường dự phòng (qua R4)
- ✅ Đã kiểm tra kết nối thành công bằng ping và tracert
- ✅ Đã kiểm tra đường dự phòng hoạt động khi R2 bị tắt

---

**Ngày hoàn thành:** ___/___/2026

**Chữ ký sinh viên:**

_________________________

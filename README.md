<<<<<<< HEAD
# UC12 – Xem lịch sử chỉnh sửa điểm (Sequence Diagram)

Sơ đồ tuần tự cho use case **Xem lịch sử chỉnh sửa điểm**, actor **Hội đồng chấm thi**, vẽ theo kiến trúc **BCE (Boundary – Control – Entity)**.

## Các đối tượng

| Loại | Tên |
|---|---|
| Actor | `HoiDongChamThi` |
| Boundary | `GiaoDienXemLichSuChinhSuaDiem` |
| Control | `XemLichSuChinhSuaDiemController` |
| Entity | `KyThi`, `MonThi`, `KetQuaChamThi`, `LichSuChinhSuaDiem` |

## Sơ đồ (ảnh)

![UC12 Sequence Diagram](UC12_Sequence_XemLichSuChinhSuaDiem.png)

## Sơ đồ (Mermaid – GitHub tự render)

```mermaid
sequenceDiagram
    actor HD as :HoiDongChamThi
    participant GD as «boundary»<br/>:GiaoDienXemLichSuChinhSuaDiem
    participant CT as «control»<br/>:XemLichSuChinhSuaDiemController
    participant KT as «entity»<br/>:KyThi
    participant MT as «entity»<br/>:MonThi
    participant KQ as «entity»<br/>:KetQuaChamThi
    participant LS as «entity»<br/>:LichSuChinhSuaDiem

    Note over HD,GD: Bước 1–2: Mở chức năng
    HD->>GD: 1: chonXemLichSuChinhSuaDiem()
    activate GD
    GD->>CT: 1.1: layTieuChiTraCuu()
    activate CT
    CT->>KT: 1.1.1: layDanhSachKyThi()
    KT-->>CT: dsKyThi
    CT->>MT: 1.1.2: layDanhSachMonThi()
    MT-->>CT: dsMonThi
    CT-->>GD: dsKyThi, dsMonThi
    deactivate CT
    GD->>GD: 1.2: hienThiTieuChiTraCuu(dsKyThi, dsMonThi)

    Note over HD,CT: Bước 3–4: Chọn và kiểm tra điều kiện
    HD->>GD: 2: chonDieuKienTraCuu(maKyThi, maMonThi, maPhach)
    GD->>CT: 2.1: kiemTraDieuKienTraCuu(maKyThi, maMonThi, maPhach)
    activate CT
    CT->>CT: 2.1.1: kiemTraDayDu()
    CT-->>GD: hopLe
    deactivate CT

    alt [AF 4.1] Chưa chọn đủ điều kiện
        GD->>GD: 2.2a: hienThiThongBao("Vui lòng chọn đầy đủ điều kiện tra cứu.")
        Note over HD,GD: Quay lại bước 3
    else Điều kiện hợp lệ
        Note over HD,KQ: Bước 5–7: Xác nhận và tìm kết quả chấm thi
        HD->>GD: 3: xacNhanTraCuu()
        GD->>CT: 3.1: traCuuKetQua(maKyThi, maMonThi, maPhach)
        activate CT
        CT->>KQ: 3.1.1: timKetQua(maKyThi, maMonThi, maPhach)
        KQ-->>CT: dsKetQua
        CT-->>GD: dsKetQua
        deactivate CT

        alt [EF 6.2] Không truy xuất được dữ liệu
            GD->>GD: 3.2a: hienThiThongBao("Không thể truy xuất dữ liệu kết quả chấm thi.")
            HD->>GD: 4a: xacNhanThongBao()
            Note over HD,GD: Kết thúc use case
        else [AF 6.1] dsKetQua rỗng
            GD->>GD: 3.2b: hienThiThongBao("Không tìm thấy kết quả chấm thi phù hợp.")
            Note over HD,GD: Quay lại bước 3
        else Có kết quả
            GD->>GD: 3.2c: hienThiDanhSachKetQua(dsKetQua)

            Note over HD,LS: Bước 8–11: Xem lịch sử chỉnh sửa
            HD->>GD: 4: chonKetQua(maKetQua)
            GD->>CT: 4.1: xemLichSuChinhSua(maKetQua)
            activate CT
            CT->>LS: 4.1.1: layLichSuTheoKetQua(maKetQua)
            LS-->>CT: dsLichSu
            CT-->>GD: dsLichSu
            deactivate CT

            alt [AF 9.1] dsLichSu rỗng
                GD->>GD: 4.2a: hienThiThongBao("Kết quả chấm thi chưa có lịch sử chỉnh sửa.")
                HD->>GD: 5a: xacNhanThongBao()
                Note over HD,GD: Quay lại bước 8 hoặc kết thúc
            else Có lịch sử
                GD->>GD: 4.2b: hienThiLichSu(dsLichSu)
                Note right of GD: điểm trước, điểm sau,<br/>thời gian, người thực hiện, lý do
                HD->>GD: 5: xemThongTinLichSu()
            end
        end
    end
    deactivate GD
```

## Quy ước đánh số message

- Message từ Actor: số nguyên `1, 2, 3…`
- Message được gọi lồng bên trong: thêm cấp `1.1`, `1.1.1`…
- Các nhánh loại trừ nhau trong `alt`: cùng số, thêm hậu tố `a, b, c` (ví dụ `3.2a`, `3.2b`, `3.2c`)
- Return message (nét đứt) không đánh số
- Guard của fragment ghi theo luồng trong đặc tả: `AF` = Alternative Flow, `EF` = Exception Flow

## Ánh xạ luồng đặc tả → fragment

| Luồng | Fragment |
|---|---|
| AF 4.1 – Chưa chọn đủ điều kiện | `alt` → thông báo, quay lại bước 3 |
| EF 6.2 – Không truy xuất được dữ liệu | `alt` → thông báo, kết thúc use case |
| AF 6.1 – Không tìm thấy kết quả | `alt` → thông báo, quay lại bước 3 |
| AF 9.1 – Chưa có lịch sử chỉnh sửa | `alt` → thông báo, quay lại bước 8 hoặc kết thúc |

## Các file

| File | Mô tả |
|---|---|
| `UC12_Sequence_XemLichSuChinhSuaDiem.png` | Ảnh sơ đồ |
| `UC12_XemLichSuChinhSuaDiem.puml` | Mã PlantUML (dán vào plantuml.com / VS Code) |
| `UC12_XemLichSuChinhSuaDiem.mmd` | Mã Mermaid gốc (có icon boundary/control/entity) |
=======
# -sequence-diagram
>>>>>>> 267cfefeb8888c1613390ecb60c3f5946c3d362d

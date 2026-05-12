## Bài tập Hệ quản trị cơ sở dữ liệu  
# Thông tin sinh viên:  
Họ và tên: Lù Đình Hưng  
Lớp: K59KMT.K01  
Mã số sinh viên: K235480106033  
Trường: Đại học Kỹ thuật Công nghiệp Thái Nguyên  

## Nhiệm vụ 1: Thiết kế CSDL  
- Tạo database `CamDo59KMT_ThaiNguyen`  
```sql  
CREATE DATABASE CamDo59KMT_ThaiNguyen  
GO  
USE CamDo59KMT_ThaiNguyen  
GO  
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/7cddad46-1c85-4fea-b4c8-b76cd11568a7" />  

- tạo bảng khách hàng `NguoiCamDo`
```sql
CREATE TABLE NguoiCamDo
(
    MaNguoiCam INT IDENTITY(1,1) PRIMARY KEY,
    TenKhach NVARCHAR(100),
    SDT VARCHAR(15),
    DiaChi NVARCHAR(200),
    CCCD VARCHAR(20)
)
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/b160a070-4b73-4387-8ab1-9f8067cae1cf" />  
- Tạo bảng nhân viên - tức người thu tiền `NguoiThuTien`
  
```sql
CREATE TABLE NguoiThuTien
(
    MaNguoiThu INT IDENTITY(1,1) PRIMARY KEY,
    TenNhanVien NVARCHAR(100),
    SoDienThoai VARCHAR(15)
)
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/7b342a1f-5b58-42d1-976b-823a820c01d2" />  
- Tạo bảng hợp đồng cầm đồ `PhieuCamDo`  

```sql
CREATE TABLE PhieuCamDo
(
    MaPhieuCam INT IDENTITY(1,1) PRIMARY KEY,
    MaNguoiCam INT,
    TienVayGoc DECIMAL(18,2),
    NgayCam DATE DEFAULT GETDATE(),
    HanLaiDon DATE,
    HanThanhLy DATE,
    TrangThaiPhieu NVARCHAR(50)
        DEFAULT N'Đang giữ đồ',
    TienConNo DECIMAL(18,2),
    FOREIGN KEY (MaNguoiCam)
        REFERENCES NguoiCamDo(MaNguoiCam)
)
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/6ee6db37-fc90-435c-afb9-83727e5ffe61" />  
- Tạo bảng đồ đang cầm, hiện vật, tiền mặt - `DoCam`  

```sql
CREATE TABLE DoCam
(
    MaDoCam INT IDENTITY(1,1) PRIMARY KEY,
    MaPhieuCam INT,
    TenDo NVARCHAR(200),
    GiaTriUocTinh DECIMAL(18,2),
    TrangThaiDo NVARCHAR(50)
        DEFAULT N'Đang giữ',
    DaBanThanhLy BIT DEFAULT 0,
    FOREIGN KEY (MaPhieuCam)
        REFERENCES PhieuCamDo(MaPhieuCam)
)
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/bd89346c-817d-408f-84b4-46b075f4c663" />  
- Tạo bảng nhật ký dòng tiền nhằm kiểm soát tiền đã thu và tiền còn nợ - NhatKyDongTien  

```sql
CREATE TABLE NhatKyDongTien
(
    MaGiaoDich INT IDENTITY(1,1) PRIMARY KEY,
    MaPhieuCam INT,
    MaNguoiThu INT,
    NgayDong DATETIME
        DEFAULT GETDATE(),
    SoTienKhachTra DECIMAL(18,2),
    SoTienNoConLai DECIMAL(18,2),
    FOREIGN KEY (MaPhieuCam)
        REFERENCES PhieuCamDo(MaPhieuCam),
    FOREIGN KEY (MaNguoiThu)
        REFERENCES NguoiThuTien(MaNguoiThu)
)
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/fd0aee1f-1bde-4713-8fcf-2bdee01679a1" />  

- Add một vài khách hàng có nhu cầu cầm đồ  

```sql
INSERT INTO NguoiCamDo
(
    TenKhach,
    SDT,
    DiaChi,
    CCCD
)
VALUES
    (N'Nguyễn Văn An',  '0911111111', N'Hà Nội',     '001111111111'),
    (N'Trần Trung Kiên',  '0922222222', N'Thái Nguyên','002222222222'),
    (N'Trần Nhất Nam',   '0933333333', N'Bắc Giang',  '003333333333'),
    (N'Phạm Quang Huy',  '0944444444', N'Hà Nội',     '004444444444'),
    (N'Đỗ Ngọc Hải',  '0955555555', N'Nam Định',   '005555555555');
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/c2f41a46-c030-416e-90e5-8c37d4586404" />  

## Nhiệm vụ 2: Cài đặt SQL (Yêu cầu viết Scripts)  
# Event 1: Đăng ký hợp đồng mới (Vay tiền)  
Viết Store Procedure tiếp nhận hợp đồng: Lưu thông tin khách hàng, danh sách tài sản (kèm giá trị định giá), số tiền vay gốc và thiết lập 2 mốc Deadline1, Deadline2.  
+Tạo `sp_TaoHopDong` - lưu thông tin khách hàng, danh sách tài sản (kèm định giá), số tiền vay gốc và thiết lập 2 mốc Deadline1, Deadline2.  

```sql
CREATE PROC sp_TaoHopDong
(
    @MaNguoiCam INT,
    @TienVayGoc DECIMAL(18,2),
    @HanLaiDon DATE,
    @HanThanhLy DATE,
    @TenDo NVARCHAR(200),
    @GiaTriUocTinh DECIMAL(18,2)
)
AS
BEGIN
    -- Tạo phiếu cầm đồ
    INSERT INTO PhieuCamDo
    (
        MaNguoiCam,
        TienVayGoc,
        HanLaiDon,
        HanThanhLy,
        TrangThaiPhieu,
        TienConNo
    )
    VALUES
    (
        @MaNguoiCam,
        @TienVayGoc,
        @HanLaiDon,
        @HanThanhLy,
        N'Đang giữ đồ',
        @TienVayGoc
    )
    -- Lấy mã phiếu vừa tạo
    DECLARE @MaPhieuCamMoi INT
    SET @MaPhieuCamMoi = SCOPE_IDENTITY()
    -- Thêm tài sản cầm cố
    INSERT INTO DoCam
    (
        MaPhieuCam,
        TenDo,
        GiaTriUocTinh,
        TrangThaiDo
    )
    VALUES
    (
        @MaPhieuCamMoi,
        @TenDo,
        @GiaTriUocTinh,
        N'Đang giữ'
    )
    PRINT N'Tạo hợp đồng thành công'
END
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/0e9fce6e-e5de-4d29-9ff8-14c9af04d905" />  
- Kiểm tra Hợp đồng + Đồ cầm
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/acbd20e5-d710-404a-8528-ae7b51ae0fb8" />
 

```sql
SELECT * FROM PhieuCamDo
GO 
SELECT * FROM DoCam
GO 
```

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/37f6abbf-0f71-4779-b26e-7d03f799b17e" />

# Event 2: Tính toán công nợ thời gian thực  
1, Viết một Function fn_CalcMoneyContract(ContractID, TargetDate) để tính tổng số tiền khách(ContractID) phải trả (Gốc + Lãi đơn + Lãi kép) tính đến ngày TargetDate.  


```sql
CREATE FUNCTION fn_CalcMoneyContract
(
    @ContractID INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE
        @TienGoc DECIMAL(18,2),
        @NgayCam DATE,
        @Deadline1 DATE,
        @SoNgay INT,
        @SoNgayQuaHan INT,
        @LaiDon DECIMAL(18,2),
        @TongNo DECIMAL(18,2)
    SELECT
        @TienGoc = TienVayGoc,
        @NgayCam = NgayCam,
        @Deadline1 = HanLaiDon
    FROM PhieuCamDo
    WHERE MaPhieuCam = @ContractID
    -- Nếu chưa quá hạn lãi đơn
    IF @TargetDate <= @Deadline1
    BEGIN
        SET @SoNgay =
            DATEDIFF
            (
                DAY,
                @NgayCam,
                @TargetDate
            )
        -- Lãi đơn:
        -- 5.000 / 1.000.000 / ngày
        -- = 0.005
        SET @LaiDon =
            @TienGoc * 0.005 * @SoNgay
        SET @TongNo =
            @TienGoc + @LaiDon
    END
    -- Nếu đã quá Deadline1
    ELSE
    BEGIN
        -- Tính lãi đơn trước
        SET @SoNgay =
            DATEDIFF
            (
                DAY,
                @NgayCam,
                @Deadline1
            )
        SET @LaiDon =
            @TienGoc * 0.005 * @SoNgay
        -- Số ngày quá hạn
        SET @SoNgayQuaHan =
            DATEDIFF
            (
                DAY,
                @Deadline1,
                @TargetDate
            )
        -- Lãi kép
        SET @TongNo =
            (@TienGoc + @LaiDon)
            * POWER(1.005, @SoNgayQuaHan)
    END
    RETURN @TongNo
END
GO
```
Test function  

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/2207539e-4af7-4d1a-890a-3c1a37e7d6b1" />  
2, Viết một Function fn_CalcMoneyTransaction(TransactionID, TargetDate) để tính số tiền phải trả của TransactionID này cho đến ngày TargetDate  

```sql
CREATE FUNCTION fn_CalcMoneyTransaction
(
    @TransactionID INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @ContractID INT
    DECLARE @TongNo DECIMAL(18,2)
    SELECT
        @ContractID = MaPhieuCam
    FROM NhatKyDongTien
    WHERE MaGiaoDich = @TransactionID
    SET @TongNo =
        dbo.fn_CalcMoneyContract
        (
            @ContractID,
            @TargetDate
        )
    RETURN @TongNo
END
GO
```

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/cb27b884-9b46-4178-8d7e-d85883826f42" />  
- Test function  

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/7c129668-3b49-4b32-bad1-cc6cfff4f30c" />  
# Event 3: Xử lý trả nợ và hoàn trả tài sản  
 Viết Store Procedure xử lý khi khách mang tiền đến: Nếu tài sản đã bị thanh lý (sau Deadline 2 và có cờ IsSold): Thông báo không thu tiền, không trả đồ. Nếu tài sản chưa bị thanh lý: Tính tổng nợ, trừ số tiền khách trả vào hệ thống. Nếu trả hết tiền, trả hết đồ và cập nhật trạng thái hợp đồng thành “Đã thanh toán đủ”; Nếu chưa trả hết tiền gốc+lãi: cập nhật trạng thái hợp đồng thành “Đang trả góp”, ghi nhận vào LOG số tiền đã trả, và số tiền còn nợ. Đưa ra danh sách gợi ý trả lại cho khách hàng này dựa trên điều kiện:  
Giá trị tài sản còn lại >= Dư nợ còn lại.  

```sql
CREATE PROC sp_ThanhToanHopDong
(
    @MaPhieuCam INT,
    @SoTienTra DECIMAL(18,2)
)
AS
BEGIN
    DECLARE
        @TongNo DECIMAL(18,2),
        @ConNo DECIMAL(18,2)
    -- Kiểm tra tài sản đã thanh lý chưa
    IF EXISTS
    (
        SELECT *
        FROM DoCam
        WHERE
            MaPhieuCam = @MaPhieuCam
            AND DaBanThanhLy = 1
    )
    BEGIN
        PRINT N'Tài sản đã bị thanh lý, không thể thanh toán'
        RETURN
    END
    -- Tính tổng nợ hiện tại
    SET @TongNo =
        dbo.fn_CalcMoneyContract
        (
            @MaPhieuCam,
            GETDATE()
        )
    -- Tính số nợ còn lại
    SET @ConNo =
        @TongNo - @SoTienTra
    -- Ghi log thanh toán
    INSERT INTO NhatKyDongTien
    (
        MaPhieuCam,
        SoTienKhachTra,
        SoTienNoConLai
    )
    VALUES
    (
        @MaPhieuCam,
        @SoTienTra,
        @ConNo
    )
    -- Cập nhật số nợ còn lại
    UPDATE PhieuCamDo
    SET TienConNo = @ConNo
    WHERE MaPhieuCam = @MaPhieuCam
    -- Nếu trả hết nợ
    IF @ConNo <= 0
    BEGIN
        -- Cập nhật trạng thái hợp đồng
        UPDATE PhieuCamDo
        SET TrangThaiPhieu = N'Đã thanh toán đủ'
        WHERE MaPhieuCam = @MaPhieuCam
        -- Trả tài sản cho khách
        UPDATE DoCam
        SET TrangThaiDo = N'Đã trả khách'
        WHERE MaPhieuCam = @MaPhieuCam
        PRINT N'Khách đã thanh toán đủ'
    END
    -- Nếu chưa trả hết
    ELSE
    BEGIN
        UPDATE PhieuCamDo
        SET TrangThaiPhieu = N'Đang trả góp'
        WHERE MaPhieuCam = @MaPhieuCam
        PRINT N'Khách chưa thanh toán hết'
    END
    -- Gợi ý tài sản có thể trả
    SELECT
        TenDo,
        GiaTriUocTinh
    FROM DoCam
    WHERE
        MaPhieuCam = @MaPhieuCam
        AND GiaTriUocTinh >= @ConNo
END
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/7c8d2b5b-e8c4-4ac3-b4a5-67fe440a9561" />

- Test procedure  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/90e6e244-4240-4444-b323-1343351d0186" />

- Xem log thanh toán  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/3211634a-c419-4ce9-8761-9b5ee957facd" />
 
- Xem hợp đồng  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/40685256-73b2-499a-bc9e-126f225141d3" />
 
- Xem tài sản  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/896bdc96-e8d8-42b0-9fd7-5fd7954c0fd4" />


# Event 4: Truy vấn danh sách nợ xấu (Nợ khó đòi)  

Xuất danh sách các khách hàng đã quá Deadline 1 mà chưa thanh toán. Yêu cầu các cột: Tên KH, Số điện thoại, Số tiền vay gốc, Số ngày quá hạn, Tổng tiền phải trả hiện tại (đến ngày hiện tại), Tổng số tiền phải trả sau 1 tháng nữa.  

```sql
CREATE VIEW vw_DanhSachNoXau
AS
SELECT
    nc.TenKhach,
    nc.SDT,
    pc.TienVayGoc,
    DATEDIFF
    (
        DAY,
        pc.HanLaiDon,
        GETDATE()
    ) AS SoNgayQuaHan,
    dbo.fn_CalcMoneyContract
    (
        pc.MaPhieuCam,
        GETDATE()
    ) AS TongTienHienTai,
    dbo.fn_CalcMoneyContract
    (
        pc.MaPhieuCam,
        DATEADD(MONTH, 1, GETDATE())
    ) AS TongTienSau1Thang
FROM PhieuCamDo pc
JOIN NguoiCamDo nc
ON pc.MaNguoiCam = nc.MaNguoiCam
WHERE
    GETDATE() > pc.HanLaiDon
    AND pc.TrangThaiPhieu
        <> N'Đã thanh toán đủ'
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/3fe67e3a-2385-4238-a818-4b536cd50143" />  
- Xem danh sách nợ xấu  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/e221fe29-e2f4-4a97-a00f-e1be9c96c914" />

# Event 5: Quản lý thanh lý tài sản  

1, Viết một Trigger tự động chuyển trạng thái hợp đồng sang "Quá hạn (nợ xấu)" sau khi hợp đồng đang ở trạng thái "Đang vay" mà ngày vượt quá Deadline 1.   

  ```sql
CREATE TRIGGER trg_ChuyenQuaHan
ON PhieuCamDo
AFTER INSERT, UPDATE
AS
BEGIN
    UPDATE PhieuCamDo
    SET TrangThaiPhieu = N'Quá hạn'
    WHERE
        GETDATE() > HanLaiDon
        AND TrangThaiPhieu = N'Đang vay'
END
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/e0547aef-3a4a-4526-8c93-37e79cbbd132" />  
- Test Trigger  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/ddb5dfd8-988c-422a-983f-23ebc2017386" />  
2, Viết một Trigger tự động chuyển trạng thái tài sản sang "Sẵn sàng thanh lý" sau khi hợp đồng đang ở trạng thái "Quá hạn (nợ xấu)" mà ngày vượt quá Deadline 2.  

```sql
CREATE TRIGGER trg_SanSangThanhLy
ON PhieuCamDo
AFTER INSERT, UPDATE
AS
BEGIN
    UPDATE DoCam
    SET TrangThaiDo = N'Sẵn sàng thanh lý'
    WHERE MaPhieuCam IN
    (
        SELECT MaPhieuCam
        FROM PhieuCamDo
        WHERE
            GETDATE() > HanThanhLy
            AND TrangThaiPhieu = N'Quá hạn'
    )
END
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/03d35678-7740-4719-99de-f2def05b5c5e" />  
Test Trigger  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/149adf00-024b-40d9-bda4-166a1cbcfa9a" />  
3, Viết một Trigger tự động chuyển trạng thái tài sản thành “Đã bán thanh lý” sau khi trạng thái của hợp đồng chuyển sang "Đã thanh lý". Chú ý: Mỗi tài sản cũng được theo dõi trạng thái: đang cầm cố, đã trả khách, đã bán thanh lý  

```sql
CREATE TRIGGER trg_DaBanThanhLy
ON PhieuCamDo
AFTER UPDATE
AS
BEGIN
    UPDATE DoCam
    SET
        TrangThaiDo = N'Đã bán thanh lý',
        DaBanThanhLy = 1
    WHERE MaPhieuCam IN
    (
        SELECT MaPhieuCam
        FROM inserted
        WHERE TrangThaiPhieu = N'Đã thanh lý'
    )
END
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/2cb06cc0-9227-4233-9b9e-ec27630b0fe0" />  
- Test Trigger  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/2808983c-7f60-45f1-ba68-055c451dce08" />  
4, Kết quả  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/4e868a38-d0a3-4b6e-8ad9-fcce30f3a3f5" />  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/0955aa9b-6563-413f-88d2-0d46d5453481" />  
# Các sự kiện bổ sung:  

- Tạo sp_GiaHanHopDong  

```sql
CREATE PROC sp_GiaHanHopDong
(
    @MaPhieuCam INT,
    @SoNgayGiaHan INT
)
AS
BEGIN
    DECLARE
        @TienLai DECIMAL(18,2),
        @TienConNo DECIMAL(18,2)
    -- Tính tiền lãi hiện tại
    SET @TienLai =
        dbo.fn_CalcMoneyContract
        (
            @MaPhieuCam,
            GETDATE()
        )
    SELECT
        @TienConNo = TienVayGoc
    FROM PhieuCamDo
    WHERE MaPhieuCam = @MaPhieuCam
    -- Chỉ lấy phần lãi
    SET @TienLai =
        @TienLai - @TienConNo
    -- Ghi log gia hạn
    INSERT INTO NhatKyDongTien
    (
        MaPhieuCam,
        SoTienKhachTra,
        SoTienNoConLai
    )
    VALUES
    (
        @MaPhieuCam,
        @TienLai,
        @TienConNo
    )
    -- Dời deadline
    UPDATE PhieuCamDo
    SET
        HanLaiDon =
            DATEADD
            (
                DAY,
                @SoNgayGiaHan,
                HanLaiDon
            ),
        HanThanhLy =
            DATEADD
            (
                DAY,
                @SoNgayGiaHan,
                HanThanhLy
            ),
        TrangThaiPhieu = N'Đang vay'
    WHERE MaPhieuCam = @MaPhieuCam
    PRINT N'Gia hạn hợp đồng thành công'
END
GO
```
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/786eb5ad-811e-406e-a341-71d8439855f9" />  
- Test gia hạn  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/08c20924-f7f1-43a6-a418-81bfd5fca9c8" />  
- Kiểm tra hợp đồng  
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/70b74c82-f95e-43d7-bd33-54966b0323af" />  



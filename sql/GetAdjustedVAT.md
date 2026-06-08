# GetAdjustedVAT — Tarihe Göre KDV Oranı Hesaplama Fonksiyonu

## Açıklama

Türkiye'de KDV oranları zaman içinde birçok kez değişmiştir. Bu fonksiyon, bir malzemenin **belirli bir tarihteki geçerli KDV oranını** döndürür. Malzemenin mevcut KDV oranı yerine, işlem tarihine göre doğru oranı hesaplar.

Özellikle geçmişe dönük fatura, irsaliye veya sipariş sorgularında KDV tutarlarının doğru hesaplanması için kullanılır.

---

## Parametreler

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| `@Bitis` | DATE | İşlem tarihi (fatura/irsaliye tarihi) |
| `@LogicalRef` | INT | Malzemenin `LG_001_ITEMS.LOGICALREF` değeri |

---

## Dönen Alanlar

| Alan | Açıklama |
|------|----------|
| `LOGICALREF` | Malzeme referansı |
| `CODE` | Malzeme kodu |
| `NAME3` | Malzeme adı |
| `STGRPCODE` | Stok grup kodu |
| `SPECODE` | Özel kod |
| `VAT` | Malzemenin güncel KDV oranı |
| `VAT_ADJUSTED` | Tarihe göre hesaplanmış KDV oranı |

---

## KDV Değişim Takvimi

| Tarih | Açıklama |
|-------|----------|
| 13.02.2022 | Gıda ve bazı ürünlerde KDV düzenlemesi |
| 31.03.2022 | Çeşitli ürün gruplarında KDV düzenlemesi |
| 18.07.2022 | Specode 41 grubunda KDV düzenlemesi |
| 09.07.2023 | Genel KDV artışı (8→10, 8→20, 18→20) |
| 12.11.2024 | Specode 42 grubunda KDV düzenlemesi (10→20) |
| 31.12.2024 | Karbonat (Specode 72) KDV düzenlemesi |

---

## SQL

```sql
CREATE FUNCTION [dbo].[GetAdjustedVAT]
(
    @Bitis DATE,        -- Kullanıcıdan alınan tarih parametresi
    @LogicalRef INT     -- İlgili kaydın LogicalRef değeri
)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        ITM.LOGICALREF,
        ITM.CODE,
        ITM.NAME3,
        ITM.STGRPCODE,
        ITM.SPECODE,
        ITM.VAT,
        CASE 
            -- 13.02.2022 KDV UYARLAMASI
            WHEN ITM.STGRPCODE = '07' AND ITM.SPECODE NOT IN ('34','72') AND @Bitis BETWEEN '2019-01-01' AND '2022-02-12' THEN 8 
            WHEN ITM.STGRPCODE = '03' AND ITM.SPECODE = '18' AND @Bitis BETWEEN '2019-01-01' AND '2022-02-12' THEN 8 
            
            -- 31.03.2022 KDV UYARLAMASI
            WHEN ITM.SPECODE IN ('94', '95', '33', '44', '43', '12', '102', '103', '104', '31') AND @Bitis BETWEEN '2019-01-01' AND '2022-03-30' THEN 18 
            
            -- 18.07.2022 KDV UYARLAMASI
            WHEN ITM.SPECODE IN ('41') AND @Bitis BETWEEN '2019-01-01' AND '2022-07-17' THEN 8  
            
            -- 09.07.2023 KDV UYARLAMASI
            -- 8 → 10
            WHEN ITM.SPECODE IN ('34') AND @Bitis BETWEEN '2019-01-01' AND '2023-07-08' THEN 8
            WHEN ITM.SPECODE IN ('01', '42', '03', '29', '36', '86', '126', '06', '76', '39', '32') AND @Bitis BETWEEN '2019-01-01' AND '2023-07-08' THEN 8
            WHEN ITM.SPECODE = '103' AND @Bitis BETWEEN '2022-03-31' AND '2023-07-08' THEN 8
            
            -- 8 → 20
            WHEN ITM.SPECODE IN ('12', '102', '104', '33', '44', '43', '94', '95', '31') AND @Bitis BETWEEN '2022-03-31' AND '2023-07-08' THEN 8
            
            -- 18 → 20
            WHEN ITM.SPECODE IN ('41') AND @Bitis BETWEEN '2022-07-18' AND '2023-07-08' THEN 18
            WHEN ITM.SPECODE IN (
                '02', '121', '13', '14', '25', '40', '09', '15', '19', '20', '21', '22', '26', '87', '88', '89', 
                '91', '96', '101', '106', '118', '131', '132', '139', '04', '45', '46', '47', '11', '35', '79', 
                '80', '83', '84', '122', '137', '07', '10', '28', '50', '51', '52', '53', '54', '55', '81', '82', 
                '85', '97', '98', '99', '100', '105', '107', '108', '117', '127', '128', '133', '135', '27', 
                '48', '49', '38'
            ) AND @Bitis BETWEEN '2019-01-01' AND '2023-07-08' THEN 18
            
            -- 12.11.2024 KDV UYARLAMASI
            -- 10 → 20
            WHEN ITM.SPECODE IN ('42') AND @Bitis BETWEEN '2023-07-09' AND '2024-11-11' THEN 10
            
            -- 31.12.2024 Karbonat KDV UYARLAMASI
            WHEN ITM.SPECODE IN ('72') AND @Bitis BETWEEN '2019-01-01' AND '2020-12-15' THEN 18
            WHEN ITM.SPECODE IN ('72') AND @Bitis BETWEEN '2020-12-16' AND '2024-12-30' THEN 1
            
            -- Varsayılan KDV
            ELSE ITM.VAT
        END AS VAT_ADJUSTED
    FROM LG_001_ITEMS ITM
    WHERE 
        ITM.LOGICALREF = @LogicalRef 
        AND ITM.CODE <> 'ÿ' 
        AND ISNULL(ITM.NAME3, '') <> '' 
        AND ITM.NAME4 <> 'Merkez'
);
```

---

## Kullanım Örneği

```sql
-- Belirli bir malzemenin 01.01.2023 tarihindeki KDV oranını getir
SELECT * FROM dbo.GetAdjustedVAT('2023-01-01', 12345)

-- Fatura satırlarıyla birlikte CROSS APPLY kullanımı
SELECT 
    STL.AMOUNT,
    STL.PRICE,
    VAT.VAT_ADJUSTED,
    STL.AMOUNT * STL.PRICE * VAT.VAT_ADJUSTED / 100 AS KDV_TUTARI
FROM LG_001_01_STLINE STL
CROSS APPLY dbo.GetAdjustedVAT('2023-01-01', STL.STOCKREF) VAT
```

---

## Notlar

- Fonksiyon `RETURNS TABLE` olarak tanımlanmıştır, `CROSS APPLY` ile kullanılır.
- `SPECODE` alanı Logo Tiger'da malzemenin KDV grubunu belirler.
- `STGRPCODE` stok grup kodudur, bazı KDV kuralları grup bazında uygulanır.
- Yeni KDV değişikliklerinde `CASE` bloğuna yeni satır eklemek yeterlidir.

---

## Ortam

- **ERP:** Logo Tiger 3
- **Veritabanı:** SQL Server (T-SQL)
- **Kapsam:** Türkiye KDV mevzuatı (2019–2024)

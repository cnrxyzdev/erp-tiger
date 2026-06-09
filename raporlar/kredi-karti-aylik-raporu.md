# Kredi Kartı Aylık Raporu

## Açıklama

Şirkete ait kurumsal ve bireysel kredi kartlarının aylık harcama tutarlarını getirir. Banka hesap kartı tipine göre filtreleme yapar, geçmiş 1 ay ile gelecek 18 aylık dönemi kapsar.

---

## Kullanılan Tablolar

| Tablo | Açıklama |
|-------|----------|
| `LG_001_BANKACC` | Banka hesap kartları |
| `LG_001_01_PAYTRANS` | Ödeme işlemleri |
| `LG_001_01_BNFICHE` | Banka fişi başlıkları |
| `LG_001_01_BNFLINE` | Banka fişi satırları |
| `LG_001_BNCARD` | Banka kartları |

---

## Filtre Kriterleri

| Alan | Değer | Açıklama |
|------|-------|----------|
| `BACC.CARDTYPE` | 5 | Kredi kartı tipi |
| `BACC.SPECODE` | '003' | Özel kod filtresi |
| `BACC.ACTIVE` | 0 | Aktif kartlar |

---

## Kart Tipi Ayrımı

| Banka Kodu | Kart Tipi |
|------------|-----------|
| 30 | Bireysel |
| Diğer | Kurumsal |

---

## SQL

```sql
CREATE PROC [dbo].[SP_KrediKartiAylik]
AS
WITH KrediKartiAylik AS
(
    SELECT
        BACC.BANKREF                                                        AS 'Banka',
        BACC.LOGICALREF                                                     AS 'Hesap',
        BACC.DEFINITION_                                                    AS 'Kart',
        YEAR(PYT.DATE_)                                                     AS 'Yil',
        MONTH(PYT.DATE_)                                                    AS 'Ay',
        CASE 
            WHEN BNF.TRCODE = 16 
            THEN IIF(BNF.DATE_ < DATEADD(mm, DATEDIFF(mm, 0, GETDATE()), 0), 0, CREDITTOT) 
            ELSE ISNULL(SUM(ISNULL(PYT.TOTAL, 0)), 0) - ISNULL(SUM(ISNULL(BNL.AMOUNT, 0)), 0) 
        END                                                                 AS TUTAR
    FROM LG_001_BANKACC BACC
    LEFT JOIN (
        SELECT 
            ISNULL(SUM(TOTAL), 0)   AS TOTAL,
            BNFCHREF,
            BNFLNREF,
            PYT.DATE_,
            PYT.BANKACCREF
        FROM LG_001_01_PAYTRANS PYT
        GROUP BY PYT.BNFCHREF, PYT.BNFLNREF, PYT.DATE_, PYT.BANKACCREF
    ) PYT ON PYT.BANKACCREF = BACC.LOGICALREF
    LEFT JOIN LG_001_01_BNFICHE BNF ON BNF.LOGICALREF = PYT.BNFCHREF
    LEFT JOIN LG_001_01_BNFLINE BNL ON BNL.LOGICALREF = PYT.BNFLNREF
    WHERE 
        BACC.CARDTYPE = 5
        AND BACC.SPECODE = '003'
        AND BACC.ACTIVE = 0
        AND PYT.DATE_ BETWEEN DATEADD(month, -1, GETDATE()) AND DATEADD(month, 18, GETDATE())
    GROUP BY 
        BACC.BANKREF, BACC.LOGICALREF, BNF.TRCODE, PYT.DATE_,
        BNF.CREDITTOT, BNF.DATE_, BACC.DEFINITION_,
        YEAR(PYT.DATE_), MONTH(PYT.DATE_)
)
SELECT 
    CASE WHEN KKA.Banka <> 30 THEN 'Kurumsal' ELSE 'Bireysel' END  AS 'KartTip',
    KKA.Banka,
    Hesap,
    Kart,
    Yil,
    Ay,
    ISNULL(SUM(TUTAR), 0)                                           AS 'Tutar'
FROM KrediKartiAylik KKA
WHERE Banka IN (
    SELECT BNC.LOGICALREF 
    FROM LG_001_BNCARD BNC 
   
)
GROUP BY KKA.Banka, KKA.Hesap, KKA.Kart, KKA.Yil, KKA.Ay
```

---

## Kullanım

```sql
EXEC SP_KrediKartiAylik
```

---

## Notlar

- `TRCODE = 16` olan fişler banka hizmet faturasıdır. Bu fişler için cari ay kontrolü yapılır, geçmiş aylardaki tutarlar 0 olarak alınır.

- Tarih aralığı dinamiktir: geçmiş 1 ay ile gelecek 18 ay.

  

---

## Ortam

- **ERP:** Logo Tiger 3
- **Veritabanı:** SQL Server (T-SQL)
- **Nesne Tipi:** Stored Procedure

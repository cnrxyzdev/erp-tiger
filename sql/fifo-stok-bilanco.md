# FIFO Stok Bilanço Hesaplama (Çoklu Birim + KDV)

## Açıklama

Logo Tiger'da stok bilanço değerini **FIFO (First In First Out)** yöntemine göre hesaplar. Çoklu birim seti ve KDV düzenlemelerini destekler. `GetAdjustedVAT` fonksiyonu ile entegre çalışarak tarihe göre doğru KDV oranını uygular.

Bilanço raporlarında stok değerlemesi için kullanılır.

---

## Özellikler

- Çoklu birim seti desteği (`CONVFACT1` / `CONVFACT2` ile birim dönüşümü)
- FIFO sıralı giriş/çıkış eşleştirmesi
- `GetAdjustedVAT` fonksiyonu ile tarihe göre KDV hesabı
- KDVli ve KDVsiz tutar ayrımı
- Depo bazında hesaplama

---

## Kullanılan Tablolar

| Tablo | Açıklama |
|-------|----------|
| `LG_001_01_STLINE` | Stok hareketleri |
| `LG_001_ITMUNITA` | Malzeme birim dönüşüm faktörleri |
| `dbo.GetAdjustedVAT` | Tarihe göre KDV oranı fonksiyonu ([bkz.](./GetAdjustedVAT.md)) |

---

## Parametreler

| Parametre | Tip | Açıklama |
|-----------|-----|----------|
| `@EndDate` | DATE | Bilanço tarihi |

---

## TRCODE Filtresi

| İşlem | TRCODE | Yön |
|-------|--------|-----|
| Satınalma İrsaliyesi | 1 | Giriş |
| Perakende İade İrsaliyesi | 2 | Giriş |
| Devir Fişi | 14 | Giriş |
| Ambar Fişi | 25 | Giriş/Çıkış |
| Sayım Fazlası | 50 | Giriş |
| Sayım Eksiği | 51 | Çıkış |
| Toptan Satış İrsaliyesi | 7 | Çıkış |
| Satınalma İade İrsaliyesi | 6 | Çıkış |

---

## IOCODE Değerleri

| IOCODE | Açıklama |
|--------|----------|
| 0 | Nötr |
| 1, 2 | Giriş |
| 3, 4 | Çıkış |

---

## Algoritma Akışı

```
1. #UomFact      → Birim dönüşüm faktörlerini geçici tabloya yükle
2. #StockFifo    → Tüm stok hareketlerini giriş/çıkış miktar ve fiyatlarıyla yükle
3. OrderedIn     → Giriş hareketlerini tarih/saat/stficheref sırasına göre numaralandır
4. RunningTotals → Her giriş satırı için kümülatif toplam hesapla
5. TotalOut      → Malzeme+depo bazında toplam çıkış miktarını hesapla
6. FifoSatir     → FIFO'ya göre kalan miktarı hesapla
7. FifoSatirKDV  → GetAdjustedVAT ile KDV uygula, KDVli tutarı hesapla
8. Sonuç         → Toplam KDVli stok değerini döndür
```

---

## SQL

```sql
/* TEMİZLİK */
DROP TABLE IF EXISTS #FifoEnvanterRaporBilanco;
DROP TABLE IF EXISTS #StockFifo;
IF OBJECT_ID('tempdb..#UomFact') IS NOT NULL
    DROP TABLE #UomFact;

/* BİRİM ÇEVİRİ FAKTÖRLERİ */
SELECT
    ITEMREF,
    UNITLINEREF,
    CONVFACT1,
    CONVFACT2
INTO #UomFact
FROM LG_001_ITMUNITA;

CREATE CLUSTERED INDEX IX_UomFact ON #UomFact(ITEMREF, UNITLINEREF);

/* STOCHFIFO TABLOSU */
CREATE TABLE #StockFifo
(
    item        INT           NOT NULL,
    warehouse   INT           NOT NULL,
    [date]      DATETIME      NOT NULL,
    [time_]     TIME          NOT NULL,
    trcode      SMALLINT      NULL,
    stficheref  INT           NULL,
    in_qty      DECIMAL(18,5) NULL,
    out_qty     DECIMAL(18,5) NULL,
    in_Price    DECIMAL(19,6) NULL,
    out_Price   DECIMAL(19,6) NULL
);

/* HAREKETLERİ YÜKLE */
INSERT INTO #StockFifo(
    item, warehouse, [date], [time_], trcode, stficheref,
    in_qty, out_qty, in_Price, out_Price
)
SELECT 
    STOCKREF        AS item,
    SOURCEINDEX     AS warehouse,
    DATE_           AS [date],
    [dbo].[LG_INTTOTIME](FTIME) AS [time_],
    TRCODE,
    STFICHEREF,

    /* GİRİŞ MİKTAR */
    ISNULL(
        CASE 
            WHEN TRCODE IN (14,50,1,2,25) AND IOCODE IN (0,1,2) THEN 
                ISNULL(
                    CASE  
                        WHEN UOMREF = 23 THEN AMOUNT
                        WHEN UOMREF = 24 THEN AMOUNT
                        WHEN UOMREF = 26 THEN ISNULL(u26.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 38 THEN AMOUNT
                        WHEN UOMREF = 39 THEN ISNULL(u39.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 40 THEN ISNULL(u40.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 41 THEN AMOUNT
                        WHEN UOMREF = 42 THEN AMOUNT / NULLIF(u42.CONVFACT1,0)
                        WHEN UOMREF = 43 THEN AMOUNT
                        WHEN UOMREF = 44 THEN ISNULL(u44.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 45 THEN AMOUNT
                        WHEN UOMREF = 46 THEN ISNULL(u46.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 47 THEN AMOUNT / ISNULL(u47.CONVFACT1,0)
                        WHEN UOMREF = 48 THEN AMOUNT
                        WHEN UOMREF = 49 THEN ISNULL(u49.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 50 THEN AMOUNT / ISNULL(u50.CONVFACT1,0)
                        WHEN UOMREF = 51 THEN AMOUNT
                        WHEN UOMREF = 52 THEN AMOUNT / ISNULL(u52.CONVFACT1,0)
                        WHEN UOMREF = 56 THEN AMOUNT
                        WHEN UOMREF = 57 THEN AMOUNT / ISNULL(u57.CONVFACT1,0)
                        WHEN UOMREF = 53 THEN AMOUNT
                        WHEN UOMREF = 54 THEN AMOUNT / ISNULL(u54.CONVFACT1,0)
                        WHEN UOMREF = 55 THEN AMOUNT / ISNULL(u55.CONVFACT1,0)
                        WHEN UOMREF = 58 THEN AMOUNT
                        WHEN UOMREF = 59 THEN ISNULL(u59.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 60 THEN ISNULL(u60.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 61 THEN AMOUNT
                        WHEN UOMREF = 62 THEN ISNULL(u62.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 64 THEN AMOUNT
                        WHEN UOMREF = 65 THEN AMOUNT / ISNULL(u65.CONVFACT1,0)
                        ELSE AMOUNT
                    END, 0
                )
        END
    , 0) AS in_qty,

    /* ÇIKIŞ MİKTAR */
    ISNULL(
        CASE 
            WHEN TRCODE IN (7,51,6,25) AND IOCODE IN (0,3,4) THEN 
                ISNULL(
                    CASE  
                        WHEN UOMREF = 23 THEN AMOUNT
                        WHEN UOMREF = 24 THEN AMOUNT
                        WHEN UOMREF = 26 THEN ISNULL(u26.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 38 THEN AMOUNT
                        WHEN UOMREF = 39 THEN ISNULL(u39.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 40 THEN ISNULL(u40.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 41 THEN AMOUNT
                        WHEN UOMREF = 42 THEN AMOUNT / ISNULL(u42.CONVFACT1,0)
                        WHEN UOMREF = 43 THEN AMOUNT
                        WHEN UOMREF = 44 THEN ISNULL(u44.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 45 THEN AMOUNT
                        WHEN UOMREF = 46 THEN ISNULL(u46.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 47 THEN AMOUNT / ISNULL(u47.CONVFACT1,0)
                        WHEN UOMREF = 48 THEN AMOUNT
                        WHEN UOMREF = 49 THEN ISNULL(u49.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 50 THEN AMOUNT / ISNULL(u50.CONVFACT1,0)
                        WHEN UOMREF = 51 THEN AMOUNT
                        WHEN UOMREF = 52 THEN AMOUNT / ISNULL(u52.CONVFACT1,0)
                        WHEN UOMREF = 56 THEN AMOUNT
                        WHEN UOMREF = 57 THEN AMOUNT / ISNULL(u57.CONVFACT1,0)
                        WHEN UOMREF = 53 THEN AMOUNT
                        WHEN UOMREF = 54 THEN AMOUNT / ISNULL(u54.CONVFACT1,0)
                        WHEN UOMREF = 55 THEN AMOUNT / ISNULL(u55.CONVFACT1,0)
                        WHEN UOMREF = 58 THEN AMOUNT
                        WHEN UOMREF = 59 THEN ISNULL(u59.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 60 THEN ISNULL(u60.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 61 THEN AMOUNT
                        WHEN UOMREF = 62 THEN ISNULL(u62.CONVFACT2,0) * AMOUNT
                        WHEN UOMREF = 64 THEN AMOUNT
                        WHEN UOMREF = 65 THEN AMOUNT / ISNULL(u65.CONVFACT1,0)
                        ELSE AMOUNT
                    END, 0
                )
        END
    , 0) AS out_qty,

    /* GİRİŞ FİYAT */
    ISNULL(
        CASE 
            WHEN TRCODE IN (14,50,1,2) AND VATINC = 1 THEN
            (
                CASE 
                    WHEN UOMREF = 23 THEN 
                        CASE WHEN TRCODE IN (14,50,1) 
                             THEN ROUND(PRICE / (1 + VAT/100) - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                             WHEN TRCODE IN (2) AND ISNULL(PARENTLNREF,0) = 0 
                             THEN ROUND(ISNULL(CASE WHEN OUTCOST <> 0 THEN OUTCOST ELSE RETCOST END, 0), 5)
                        END
                    WHEN UOMREF = 26 THEN 
                        ROUND(PRICE / (1 + VAT/100) - (DISTCOST/NULLIF(AMOUNT,0)), 5) / ISNULL(u26.CONVFACT2, 0)
                    ELSE ROUND(PRICE / (1 + VAT/100) - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                END
            )
            WHEN TRCODE IN (14,50,1,25) AND VATINC = 0 AND IOCODE IN (0,1,2) THEN
            (
                CASE
                    WHEN UOMREF = 23 THEN ROUND(CASE WHEN TRCODE=25 THEN OUTCOST ELSE PRICE END - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                    WHEN UOMREF = 26 THEN ROUND(CASE WHEN TRCODE=25 THEN OUTCOST ELSE PRICE END - (DISTCOST/NULLIF(AMOUNT,0)), 5) / ISNULL(u26.CONVFACT2, 0)
                    ELSE ROUND(CASE WHEN TRCODE=25 THEN OUTCOST ELSE PRICE END - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                END
            )
        END
    , 0) AS in_Price,

    /* ÇIKIŞ FİYAT */
    ISNULL(
        CASE 
            WHEN TRCODE IN (7,51) AND VATINC = 1 THEN
            (
                CASE
                    WHEN UOMREF = 23 THEN ROUND(PRICE / (1 + VAT/100) - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                    WHEN UOMREF = 26 THEN ROUND(PRICE / (1 + VAT/100) - (DISTCOST/NULLIF(AMOUNT,0)), 5) / ISNULL(u26.CONVFACT2, 0)
                    ELSE ROUND(PRICE / (1 + VAT/100) - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                END
            )
            WHEN TRCODE IN (7,51,25) AND VATINC = 0 AND IOCODE IN (0,3,4) THEN
            (
                CASE
                    WHEN UOMREF = 23 THEN ROUND(CASE WHEN TRCODE=25 THEN OUTCOST ELSE PRICE END - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                    WHEN UOMREF = 26 THEN ROUND(CASE WHEN TRCODE=25 THEN OUTCOST ELSE PRICE END - (DISTCOST/NULLIF(AMOUNT,0)), 5) / ISNULL(u26.CONVFACT2, 0)
                    ELSE ROUND(CASE WHEN TRCODE=25 THEN OUTCOST ELSE PRICE END - (DISTCOST/NULLIF(AMOUNT,0)), 5)
                END
            )
        END
    , 0) AS out_Price

FROM LG_001_01_STLINE s
LEFT JOIN #UomFact u26 ON u26.ITEMREF = s.STOCKREF AND u26.UNITLINEREF = 26
LEFT JOIN #UomFact u39 ON u39.ITEMREF = s.STOCKREF AND u39.UNITLINEREF = 39
LEFT JOIN #UomFact u40 ON u40.ITEMREF = s.STOCKREF AND u40.UNITLINEREF = 40
LEFT JOIN #UomFact u42 ON u42.ITEMREF = s.STOCKREF AND u42.UNITLINEREF = 42
LEFT JOIN #UomFact u44 ON u44.ITEMREF = s.STOCKREF AND u44.UNITLINEREF = 44
LEFT JOIN #UomFact u46 ON u46.ITEMREF = s.STOCKREF AND u46.UNITLINEREF = 46
LEFT JOIN #UomFact u47 ON u47.ITEMREF = s.STOCKREF AND u47.UNITLINEREF = 47
LEFT JOIN #UomFact u49 ON u49.ITEMREF = s.STOCKREF AND u49.UNITLINEREF = 49
LEFT JOIN #UomFact u50 ON u50.ITEMREF = s.STOCKREF AND u50.UNITLINEREF = 50
LEFT JOIN #UomFact u52 ON u52.ITEMREF = s.STOCKREF AND u52.UNITLINEREF = 52
LEFT JOIN #UomFact u53 ON u53.ITEMREF = s.STOCKREF AND u53.UNITLINEREF = 53
LEFT JOIN #UomFact u54 ON u54.ITEMREF = s.STOCKREF AND u54.UNITLINEREF = 54
LEFT JOIN #UomFact u55 ON u55.ITEMREF = s.STOCKREF AND u55.UNITLINEREF = 55
LEFT JOIN #UomFact u56 ON u56.ITEMREF = s.STOCKREF AND u56.UNITLINEREF = 56
LEFT JOIN #UomFact u57 ON u57.ITEMREF = s.STOCKREF AND u57.UNITLINEREF = 57
LEFT JOIN #UomFact u59 ON u59.ITEMREF = s.STOCKREF AND u59.UNITLINEREF = 59
LEFT JOIN #UomFact u60 ON u60.ITEMREF = s.STOCKREF AND u60.UNITLINEREF = 60
LEFT JOIN #UomFact u62 ON u62.ITEMREF = s.STOCKREF AND u62.UNITLINEREF = 62
LEFT JOIN #UomFact u65 ON u65.ITEMREF = s.STOCKREF AND u65.UNITLINEREF = 65
WHERE 
    CANCELLED = 0
    AND SOURCEINDEX <> 2
    AND STOCKREF <> 0
    AND DATE_ <= @EndDate
ORDER BY 
    DATE_ ASC,
    CASE 
        WHEN TRCODE IN (50,1,2) AND IOCODE IN (0,1,2) THEN '05.00.00'
        WHEN TRCODE IN (14) THEN STFICHEREF
        ELSE [dbo].[LG_INTTOTIME](FTIME)
    END ASC;

/* INDEX */
CREATE CLUSTERED INDEX IX_StockFifo_ItemWhDate
ON #StockFifo (item, warehouse, [date], [time_], stficheref);


/* FIFO + KDV TOPLAM HESAP */
;WITH OrderedIn AS (
    SELECT
        sf.item,
        sf.warehouse,
        sf.in_qty,
        sf.in_Price AS price,
        ROW_NUMBER() OVER (
            PARTITION BY sf.item, sf.warehouse
            ORDER BY sf.[date] ASC, sf.[time_] ASC, sf.stficheref
        ) AS S_No
    FROM #StockFifo sf
    WHERE sf.in_qty <> 0
),
RunningTotals AS (
    SELECT
        oi.item,
        oi.warehouse,
        oi.in_qty,
        oi.price,
        -- Kümülatif toplam giriş miktarı
        SUM(oi.in_qty) OVER (
            PARTITION BY oi.item, oi.warehouse
            ORDER BY oi.S_No
            ROWS UNBOUNDED PRECEDING
        ) AS Total,
        -- Bir önceki satıra kadar olan toplam
        SUM(oi.in_qty) OVER (
            PARTITION BY oi.item, oi.warehouse
            ORDER BY oi.S_No
            ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
        ) AS PrevTotal
    FROM OrderedIn oi
),
TotalOut AS (
    SELECT
        item,
        warehouse,
        SUM(out_qty) AS Qty
    FROM #StockFifo
    WHERE out_qty <> 0
    GROUP BY item, warehouse
),
FifoSatir AS (
    SELECT
        rt.item,
        rt.warehouse,
        CASE
            WHEN COALESCE(rt.PrevTotal, 0) > COALESCE(o.Qty, 0)
                THEN rt.in_qty
            ELSE rt.Total - COALESCE(o.Qty, 0)
        END AS KalanMiktar,
        rt.price
    FROM RunningTotals rt
    LEFT JOIN TotalOut o
        ON rt.item = o.item
       AND rt.warehouse = o.warehouse
    WHERE rt.Total > COALESCE(o.Qty, 0)
),
FifoSatirKDV AS (
    SELECT
        f.item,
        f.warehouse,
        f.KalanMiktar,
        f.price,
        kdv.VAT_ADJUSTED                                        AS KDV,
        (f.KalanMiktar * f.price) * (1 + kdv.VAT_ADJUSTED / 100.0) AS TutarKDVli
    FROM FifoSatir f
    CROSS APPLY dbo.GetAdjustedVAT(@EndDate, f.item) kdv
)
SELECT
    SUM(TutarKDVli) AS Toplam
FROM FifoSatirKDV;
```

---

## Birim Dönüşüm Mantığı

| Durum | Hesap |
|-------|-------|
| `CONVFACT2` ile çarpım | `AMOUNT * CONVFACT2` |
| `CONVFACT1` ile bölüm | `AMOUNT / CONVFACT1` |
| Temel birim (23, 24, 38...) | `AMOUNT` direkt kullanılır |

---

## Fiyat Hesaplama Mantığı

| Durum | Formül |
|-------|--------|
| KDV dahil fiyat (`VATINC=1`) | `PRICE / (1 + VAT/100) - (DISTCOST/AMOUNT)` |
| KDV hariç fiyat (`VATINC=0`) | `PRICE - (DISTCOST/AMOUNT)` |
| İade satırı (`TRCODE=2`) | `OUTCOST` veya `RETCOST` kullanılır |
| Ambar fişi (`TRCODE=25`) | `OUTCOST` kullanılır |

---

## Notlar

- `SOURCEINDEX <> 2` filtresi ile belirli bir depo hariç tutulur, ihtiyaca göre değiştirilebilir.
- `LG_INTTOTIME` Logo Tiger'a özgü bir fonksiyondur, integer formatındaki saati TIME'a çevirir.
- `GetAdjustedVAT` fonksiyonu için [bkz.](./GetAdjustedVAT.md)
- Büyük veri setlerinde performans için `#StockFifo` üzerindeki clustered index kritiktir.

---

## Ortam

- **ERP:** Logo Tiger 3
- **Veritabanı:** SQL Server (T-SQL)
- **Yöntem:** FIFO (First In First Out)
- **Bağımlılık:** `dbo.GetAdjustedVAT`, `dbo.LG_INTTOTIME`

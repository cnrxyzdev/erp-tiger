# Fatura, İrsaliye ve Sipariş Metreküp & Ağırlık Raporu

## Açıklama

Logo Tiger'da fatura, irsaliye ve sipariş satırlarındaki malzemelerin birim bazında brüt ağırlık (kg) ve brüt hacim (m³) bilgilerini tek sorguda getirir. Hacim hesabında ft³ → m³ dönüşümü de dahildir.

---

## Kullanılan Tablolar

| Tablo | Açıklama |
|-------|----------|
| `LG_001_01_INVOICE` | Fatura başlıkları |
| `LG_001_01_STLINE` | Fatura / irsaliye satırları |
| `LG_001_01_STFICHE` | İrsaliye başlıkları |
| `LG_001_01_ORFICHE` | Sipariş başlıkları |
| `LG_001_01_ORFLINE` | Sipariş satırları |
| `LG_001_ITEMS` | Malzeme kartları |
| `LG_001_ITMUNITA` | Malzeme birim ağırlık/hacim bilgileri |
| `LG_001_UNITSETL` | Birim setleri |
| `LG_001_CLCARD` | Cari hesap kartları |

---

## Hacim Birimi Referansları (GROSSVOLREF)

| GROSSVOLREF | Birim | Dönüşüm |
|-------------|-------|---------|
| 17 | m³ | Direkt kullanılır |
| 16 | ft³ | × 0.028317 → m³ |

---

## SQL

```sql
------ Faturalar ----------------------
SELECT
    INV.DATE_                                                               AS 'Tarih',
    INV.FICHENO                                                             AS 'Fiş No',
    (SELECT TOP 1 CLC.DEFINITION_
     FROM LG_001_CLCARD CLC
     WHERE CLC.LOGICALREF = INV.CLIENTREF)                                  AS 'Cari',
    dbo.fn_trcode('Invoice', INV.TRCODE, 0, 0)                             AS 'Fiş Türü',
    INV.BRANCH                                                              AS 'İşyeri',
    INV.SOURCEINDEX                                                         AS 'Depo',
    (SELECT TOP 1 NAME3
     FROM LG_001_ITEMS ITM
     WHERE ITM.LOGICALREF = STL.STOCKREF)                                   AS 'Malzeme',
    STL.AMOUNT                                                              AS 'Miktar',
    (SELECT TOP 1 UNTS.CODE
     FROM LG_001_UNITSETL UNTS
     WHERE UNTS.LOGICALREF = STL.UOMREF)                                    AS 'Birim',
    ISNULL(STL.AMOUNT, 0) *
        (SELECT TOP 1 ISNULL(ITMA.GROSSWEIGHT, 0)
         FROM LG_001_ITMUNITA ITMA
         WHERE ITMA.ITEMREF = STL.STOCKREF
           AND ITMA.UNITLINEREF = STL.UOMREF)                               AS 'Brüt KG',
    ISNULL(STL.AMOUNT, 0) *
        (SELECT CASE
                    WHEN ITMA.GROSSVOLREF = 17 THEN ISNULL(ITMA.GROSSVOLUME, 0)
                    WHEN ITMA.GROSSVOLREF = 16 THEN ISNULL(ITMA.GROSSVOLUME, 0) * 0.028317
                END
         FROM LG_001_ITMUNITA ITMA
         WHERE ITMA.ITEMREF = STL.STOCKREF
           AND ITMA.UNITLINEREF = STL.UOMREF)                               AS 'Brüt M³'
FROM LG_001_01_STLINE STL
LEFT JOIN LG_001_01_INVOICE INV ON STL.INVOICEREF = INV.LOGICALREF
WHERE STL.STOCKREF <> 0
  AND INV.TRCODE IN (7, 2, 51, 1, 8, 50, 6)
  AND INV.CANCELLED = 0

UNION ALL

-------- Siparişler ---------------------------
SELECT
    ORF.DATE_                                                               AS 'Tarih',
    ORF.FICHENO                                                             AS 'Fiş No',
    (SELECT TOP 1 CLC.DEFINITION_
     FROM LG_001_CLCARD CLC
     WHERE CLC.LOGICALREF = ORF.CLIENTREF)                                  AS 'Cari',
    dbo.fn_trcode('Orfiche', ORF.TRCODE, 0, 0)                            AS 'Fiş Türü',
    ORF.BRANCH                                                              AS 'İşyeri',
    ORF.SOURCEINDEX                                                         AS 'Depo',
    (SELECT TOP 1 NAME
     FROM LG_001_ITEMS ITM
     WHERE ITM.LOGICALREF = ORL.STOCKREF)                                   AS 'Malzeme',
    ORL.AMOUNT                                                              AS 'Miktar',
    (SELECT TOP 1 UNTS.CODE
     FROM LG_001_UNITSETL UNTS
     WHERE UNTS.LOGICALREF = ORL.UOMREF)                                    AS 'Birim',
    ISNULL(ORL.AMOUNT, 0) *
        (SELECT TOP 1 ISNULL(ITMA.GROSSWEIGHT, 0)
         FROM LG_001_ITMUNITA ITMA
         WHERE ITMA.ITEMREF = ORL.STOCKREF
           AND ITMA.UNITLINEREF = ORL.UOMREF)                               AS 'Brüt KG',
    ISNULL(ORL.AMOUNT, 0) *
        (SELECT CASE
                    WHEN ITMA.GROSSVOLREF = 17 THEN ISNULL(ITMA.GROSSVOLUME, 0)
                    WHEN ITMA.GROSSVOLREF = 16 THEN ISNULL(ITMA.GROSSVOLUME, 0) * 0.028317
                END
         FROM LG_001_ITMUNITA ITMA
         WHERE ITMA.ITEMREF = ORL.STOCKREF
           AND ITMA.UNITLINEREF = ORL.UOMREF)                               AS 'Brüt M³'
FROM LG_001_01_ORFLINE ORL
LEFT JOIN LG_001_01_ORFICHE ORF ON ORL.ORDFICHEREF = ORF.LOGICALREF
WHERE ORF.CANCELLED = 0
  AND ORL.STOCKREF <> 0

UNION ALL

---------- İrsaliyeler ------------------------
SELECT
    STFIC.DATE_                                                             AS 'Tarih',
    STFIC.FICHENO                                                           AS 'Fiş No',
    (SELECT TOP 1 CLC.DEFINITION_
     FROM LG_001_CLCARD CLC
     WHERE CLC.LOGICALREF = STFIC.CLIENTREF)                                AS 'Cari',
    dbo.fn_trcode('Stfiche', STFIC.TRCODE, 0, 0)                          AS 'Fiş Türü',
    STFIC.BRANCH                                                            AS 'İşyeri',
    STFIC.SOURCEINDEX                                                       AS 'Depo',
    (SELECT TOP 1 NAME3
     FROM LG_001_ITEMS ITM
     WHERE ITM.LOGICALREF = STLL.STOCKREF)                                  AS 'Malzeme',
    STLL.AMOUNT                                                             AS 'Miktar',
    (SELECT TOP 1 UNTS.CODE
     FROM LG_001_UNITSETL UNTS
     WHERE UNTS.LOGICALREF = STLL.UOMREF)                                   AS 'Birim',
    ISNULL(STLL.AMOUNT, 0) *
        (SELECT TOP 1 ISNULL(ITMA.GROSSWEIGHT, 0)
         FROM LG_001_ITMUNITA ITMA
         WHERE ITMA.ITEMREF = STLL.STOCKREF
           AND ITMA.UNITLINEREF = STLL.UOMREF)                              AS 'Brüt KG',
    ISNULL(STLL.AMOUNT, 0) *
        (SELECT CASE
                    WHEN ITMA.GROSSVOLREF = 17 THEN ISNULL(ITMA.GROSSVOLUME, 0)
                    WHEN ITMA.GROSSVOLREF = 16 THEN ISNULL(ITMA.GROSSVOLUME, 0) * 0.028317
                END
         FROM LG_001_ITMUNITA ITMA
         WHERE ITMA.ITEMREF = STLL.STOCKREF
           AND ITMA.UNITLINEREF = STLL.UOMREF)                              AS 'Brüt M³'
FROM LG_001_01_STLINE STLL
LEFT JOIN LG_001_01_STFICHE STFIC ON STLL.STFICHEREF = STFIC.LOGICALREF
WHERE STLL.STOCKREF <> 0
  AND STFIC.TRCODE IN (7, 2, 51, 1, 8, 50, 6)
  AND STFIC.CANCELLED = 0
```

---

## Notlar

- `TRCODE IN (7, 2, 51, 1, 8, 50, 6)` — Toptan/perakende satış, iade ve satınalma fişlerini kapsar. TRCODE açıklamaları için [trcode-reference.md](../sql/trcode-reference.md) dosyasına bak.
- Siparişlerde TRCODE filtresi uygulanmamıştır, tüm siparişler getirilir.
- `GROSSVOLREF = 16` (ft³) değeri otomatik olarak m³'e çevrilir (`× 0.028317`).
- Malzeme birim bilgisi `LG_001_ITMUNITA` tablosunda tanımlı değilse sonuç `NULL` döner.

---

## Ortam

- **ERP:** Logo Tiger 3
- **Veritabanı:** SQL Server (T-SQL)

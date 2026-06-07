# Fatura ile İrsaliye İşyerinin Farklı Olması

## Problem

E-arşiv faturası ile ilgili irsaliyenin **farklı işyerlerine** ait olması durumunda fatura Logo Tiger üzerinden direkt silinemez. Kaydın silinebilmesi için önce SQL seviyesinde e-arşiv statüsünün sıfırlanması gerekir.

> **Not:** E-fatura kaydını silmek için `Invoice` tablosunda `ESTATUS` alanını `0` yapmak yeterlidir. Sonrasında kayıt silinebilir.

---

## Çözüm Adımları

### Adım 1 — Fatura Referanslarını Al

Herhangi bir değişiklik yapmadan önce faturanın `GUID`, `LOGICALREF` ve `FICHENO` değerlerini not al.

```sql
SELECT ESTATUS, GUID, LOGICALREF, *
FROM LG_001_01_INVOICE
WHERE FICHENO = 'CCG2020000000017'
```

**Not alınacak örnek değerler:**
- `GUID` = `AD55C7FA-5C8C-4EE3-9997-742F1300E4BE`
- `LOGICALREF` = `6904`
- `FICHENO` = `CCG2020000000017`

---

### Adım 2 — E-Arşiv Statüsünü Gönderilmedi'ye Çek

Fatura ve ilgili e-arşiv kayıtlarını gönderilmedi statüsüne al.

```sql
UPDATE LG_001_01_INVOICE
SET ESTATUS = 0
WHERE FICHENO = 'CCG2020000000017'

UPDATE LG_001_01_EARCHIVEDET
SET EARCHIVESTATUS = 0
WHERE INVOICEREF = '6904'

UPDATE LG_001_01_DATAEXCHHISTORY
SET TRANSTYPE = 0
WHERE TRANSTYPE = 7
AND DOCREF = '6904'
```

---

### Adım 3 — Faturayı Tiger'dan Çıkar (İrsaliyeye Dokunma)

Logo Tiger'a gir, ilgili faturayı **Çıkar** işlemi ile sil.

> ⚠️ **İrsaliyeyi kesinlikle silme.** Sadece faturayı çıkar.

---

### Adım 4 — İrsaliyeyi Yeniden Faturala

İrsaliyeyi Logo Tiger üzerinden önceki fatura ayarlarıyla aynı şekilde yeniden faturala.

---

### Adım 5 — Yeni Faturaya Orijinal GUID ve Gönderildi Statüsü Yaz

Yeni fatura oluşturulduktan sonra orijinal GUID'i yaz ve gönderildi statüsüne al.

```sql
-- Yeni faturanın e-arşiv statüsünü gönderildi yap
UPDATE LG_001_01_INVOICE
SET ESTATUS = 2
WHERE FICHENO = 'CCG2020000000017'

-- Yeni fatura ref ile e-arşiv detay statüsünü gönderildi yap
UPDATE LG_001_01_EARCHIVEDET
SET EARCHIVESTATUS = 6
WHERE INVOICEREF = '7036'

-- Orijinal GUID'i yeni faturaya yaz
UPDATE LG_001_01_INVOICE
SET GUID = 'AD55C7FA-5C8C-4EE3-9997-742F1300E4BE'
WHERE FICHENO = 'CCG2020000000017'
```

---

### Adım 6 — Kontrol

```sql
SELECT ESTATUS, GUID, *
FROM LG_001_01_INVOICE
WHERE FICHENO = 'CCG2020000000017'

SELECT EARCHIVESTATUS, *
FROM LG_001_01_EARCHIVEDET
WHERE INVOICEREF = '129'

SELECT TRANSTYPE, *
FROM LG_001_01_DATAEXCHHISTORY
WHERE DOCREF = '129'
```

---

## Alan Referansları

| Tablo | Alan | Açıklama |
|-------|------|----------|
| `LG_001_01_INVOICE` | `ESTATUS` | E-arşiv statüsü (0 = gönderilmedi, 2 = gönderildi) |
| `LG_001_01_INVOICE` | `GUID` | GİB tarafından kullanılan benzersiz tanımlayıcı |
| `LG_001_01_EARCHIVEDET` | `EARCHIVESTATUS` | E-arşiv detay statüsü (0 = gönderilmedi, 6 = gönderildi) |
| `LG_001_01_DATAEXCHHISTORY` | `TRANSTYPE` | İşlem tipi (0 = sıfırlandı, 7 = e-arşiv) |

---

## Ortam

- **ERP:** Logo Tiger 3
- **Modül:** E-Arşiv
- **Veritabanı:** SQL Server (T-SQL)

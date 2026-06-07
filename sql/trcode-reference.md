# Logo Tiger - TRCODE Reference Guide

## What is TRCODE?

In Logo Tiger ERP, every transaction has a **TRCODE** value that identifies the document type.
The same TRCODE number can mean different things depending on which table it belongs to.
This guide maps each table's TRCODE values to their corresponding document types.

---

## Invoice Table

| TRCODE | Turkish | English |
|--------|---------|---------|
| 1 | Satınalma Faturası | Purchase Invoice |
| 2 | Perakende İade Faturası | Retail Return Invoice |
| 3 | Toptan Satış İade Faturası | Wholesale Return Invoice |
| 4 | Alınan Hizmet Faturası | Received Service Invoice |
| 5 | Alınan Proforma Faturası | Received Proforma Invoice |
| 6 | Satınalma İade Faturası | Purchase Return Invoice |
| 7 | Perakende Satış Faturası | Retail Sales Invoice |
| 8 | Toptan Satış Faturası | Wholesale Sales Invoice |
| 9 | Verilen Hizmet Faturası | Issued Service Invoice |
| 10 | Verilen Proforma Faturası | Issued Proforma Invoice |
| 13 | Alınan Fiyat Farkı Faturası | Received Price Difference Invoice |
| 14 | Verilen Fiyat Farkı Faturası | Issued Price Difference Invoice |
| 26 | Müstahsil Makbuzu | Producer Receipt |

---

## Clfiche / Clfline Table (Current Account Transactions)

| TRCODE | Turkish | English |
|--------|---------|---------|
| 1 | Nakit Tahsilat | Cash Collection |
| 2 | Nakit Ödeme | Cash Payment |
| 3 | Borç Dekontu | Debit Note |
| 4 | Alacak Dekontu | Credit Note |
| 5 | Virman İşlemi | Internal Transfer |
| 6 | Kur Farkı İşlemi | Exchange Rate Difference |
| 12 | Özel İşlem | Special Transaction |
| 14 | Açılış Fişi | Opening Slip |
| 20 | Gelen Havaleler | Incoming Wire Transfers |
| 21 | Gönderilen Havaleler | Outgoing Wire Transfers |
| 24 | Döviz Alış Belgesi | Foreign Currency Purchase Document |
| 25 | Döviz Satış Belgesi | Foreign Currency Sale Document |
| 28 | Banka Alınan Hizmet Faturası | Bank Received Service Invoice |
| 31 | Satınalma Faturası | Purchase Invoice |
| 32 | Perakende Satış İade Faturası | Retail Return Invoice |
| 33 | Toptan Satış İade Faturası | Wholesale Return Invoice |
| 34 | Alınan Hizmet Faturası | Received Service Invoice |
| 35 | Alınan Proforma Faturası | Received Proforma Invoice |
| 36 | Satınalma İade Faturası | Purchase Return Invoice |
| 37 | Perakende Satış Faturası | Retail Sales Invoice |
| 38 | Toptan Satış Faturası | Wholesale Sales Invoice |
| 39 | Verilen Hizmet Faturası | Issued Service Invoice |
| 40 | Verilen Proforma Faturası | Issued Proforma Invoice |
| 41 | Verilen Vade Farkı Faturası | Issued Maturity Difference Invoice |
| 42 | Alınan Vade Farkı Faturası | Received Maturity Difference Invoice |
| 43 | Alınan Fiyat Farkı Faturası | Received Price Difference Invoice |
| 44 | Verilen Fiyat Farkı Faturası | Issued Price Difference Invoice |
| 46 | Alınan Ser. Mes. Makbuzu | Received Freelance Receipt |
| 56 | Müstahsil Makbuzu | Producer Receipt |
| 61 | Çek Girişi | Cheque Entry |
| 62 | Senet Girişi | Promissory Note Entry |
| 63 | Çek Çıkış Cari Hesaba | Cheque Out to Current Account |
| 64 | Senet Çıkış Cari Hesaba | Promissory Note Out to Current Account |
| 70 | Kredi Kartı Fişi | Credit Card Slip |
| 71 | Kredi Kartı İade Fişi | Credit Card Return Slip |
| 81 | Ödemeli Satış Siparişi | Paid Sales Order |
| 82 | Ödemeli Satınalma Siparişi | Paid Purchase Order |

---

## Kslines Table (Cash Account Transactions)

| TRCODE | Turkish | English |
|--------|---------|---------|
| 11 | Cari Hesap Tahsilat | Current Account Collection |
| 12 | Cari Hesap Ödeme | Current Account Payment |
| 21 | Bankaya Yatırılan | Deposited to Bank |
| 22 | Bankadan Çekilen | Withdrawn from Bank |
| 31 | Mal Alım Faturası | Purchase Invoice |
| 32 | Perakende Sat İade Faturası | Retail Return Invoice |
| 33 | Toptan Sat İade Faturası | Wholesale Return Invoice |
| 34 | Alınan Hizmet Faturası | Received Service Invoice |
| 35 | Satınalma İade Faturası | Purchase Return Invoice |
| 36 | Perakende Sat Faturası | Retail Sales Invoice |
| 37 | Toptan Sat Faturası | Wholesale Sales Invoice |
| 38 | Verilen Hizmet Faturası | Issued Service Invoice |
| 39 | Müstahsil Makbuzu | Producer Receipt |
| 41 | Muhasebe Tahsil | Accounting Collection |
| 42 | Muhasebe Tediye | Accounting Payment |
| 61 | Çek Tahsili | Cheque Collection |
| 62 | Senet Tahsili | Promissory Note Collection |
| 63 | Çek Ödemesi | Cheque Payment |
| 64 | Senet Ödemesi | Promissory Note Payment |
| 71 | Açılış (Borç) | Opening (Debit) |
| 72 | Açılış (Alacak) | Opening (Credit) |
| 73 | Virman (Borç) | Transfer (Debit) |
| 74 | Virman (Alacak) | Transfer (Credit) |
| 75 | Gider Pusulası | Expense Receipt |
| 76 | Verilen Serbest Meslek Makbuzu | Issued Freelance Receipt |
| 77 | Alınan Serbest Meslek Makbuzu | Received Freelance Receipt |
| 79 | Kur Farkı (Borç) | Exchange Rate Difference (Debit) |
| 80 | Kur Farkı (Alacak) | Exchange Rate Difference (Credit) |

---

## Bnfiche Table (Bank Transactions)

| TRCODE | Turkish | English |
|--------|---------|---------|
| 1 | Banka İşlem Fişi | Bank Transaction Slip |
| 2 | Banka Virman Fişi | Bank Transfer Slip |
| 3 | Gelen Havale / EFT | Incoming Wire / EFT |
| 4 | Gönderilen Havale / EFT | Outgoing Wire / EFT |
| 5 | Banka Açılış Fişi | Bank Opening Slip |
| 6 | Banka Kur Farkı Fişi | Bank Exchange Rate Difference Slip |
| 7 | Döviz Alış Belgesi | Foreign Currency Purchase Document |
| 8 | Döviz Satış Belgesi | Foreign Currency Sale Document |
| 16 | Banka Alınan Hizmet Faturası | Bank Received Service Invoice |
| 17 | Banka Verilen Hizmet Faturası | Bank Issued Service Invoice |
| 18 | Bankadan Çek Ödemesi | Bank Cheque Payment |
| 19 | Bankadan Senet Ödemesi | Bank Promissory Note Payment |

---

## Stfiche Table (Inventory Slips)

| TRCODE | Turkish | English |
|--------|---------|---------|
| 1 | Satınalma İrsaliyesi | Purchase Dispatch Note |
| 2 | Perakende Satış İade İrsaliyesi | Retail Return Dispatch Note |
| 3 | Toptan Satış İade İrsaliyesi | Wholesale Return Dispatch Note |
| 4 | Konsinye Çıkış İade İrsaliyesi | Consignment Out Return Note |
| 5 | Konsinye Giriş İrsaliyesi | Consignment In Dispatch Note |
| 6 | Satınalma İade İrsaliyesi | Purchase Return Dispatch Note |
| 7 | Perakende Satış İrsaliyesi | Retail Sales Dispatch Note |
| 8 | Toptan Satış İrsaliyesi | Wholesale Sales Dispatch Note |
| 9 | Konsinye Çıkış İrsaliyesi | Consignment Out Dispatch Note |
| 10 | Konsinye Giriş İade İrsaliyesi | Consignment In Return Note |
| 11 | Fire Fişi | Waste/Loss Slip |
| 12 | Sarf Fişi | Consumption Slip |
| 13 | Üretimden Giriş Fişi | Production Entry Slip |
| 14 | Devir Fişi | Transfer Slip |
| 25 | Ambar Fişi | Warehouse Slip |
| 26 | Müstahsil İrsaliyesi | Producer Dispatch Note |
| 30 | Arızalı Giriş İrsaliyesi | Defective Goods Entry Note |
| 31 | Değişim Giriş İrsaliyesi | Exchange Entry Note |
| 32 | Sunum Giriş İrsaliyesi | Presentation Entry Note |
| 36 | Arızalı Çıkış İrsaliyesi | Defective Goods Exit Note |
| 37 | Değişim Çıkış İrsaliyesi | Exchange Exit Note |
| 38 | Sunum Çıkış İrsaliyesi | Presentation Exit Note |
| 50 | Sayım Fazlası Fişi | Inventory Surplus Slip |
| 51 | Sayım Eksiği Fişi | Inventory Shortage Slip |

---

## Stline Table (Inventory Lines)

> **Note:** Stline uses an additional parameter `STFICHEREF` to differentiate document types when the same TRCODE is shared.

| TRCODE | STFICHEREF | Turkish | English |
|--------|-----------|---------|---------|
| 1 | - | Satınalma Faturası | Purchase Invoice |
| 2 | - | Perakende İade Faturası | Retail Return Invoice |
| 3 | - | Toptan Satış İade Faturası | Wholesale Return Invoice |
| 4 | 0 | Alınan Hizmet Faturası | Received Service Invoice |
| 4 | > 0 | Konsinye Çıkış İade İrsaliyesi | Consignment Out Return Note |
| 5 | 0 | Alınan Proforma Faturası | Received Proforma Invoice |
| 5 | > 0 | Konsinye Giriş İrsaliyesi | Consignment In Dispatch Note |
| 6 | - | Satınalma İade Faturası | Purchase Return Invoice |
| 7 | - | Perakende Satış Faturası | Retail Sales Invoice |
| 8 | - | Toptan Satış Faturası | Wholesale Sales Invoice |
| 9 | 0 | Verilen Hizmet Faturası | Issued Service Invoice |
| 9 | > 0 | Konsinye Çıkış İrsaliyesi | Consignment Out Dispatch Note |
| 10 | 0 | Verilen Proforma Faturası | Issued Proforma Invoice |
| 10 | > 0 | Konsinye Giriş İade İrsaliyesi | Consignment In Return Note |
| 11 | - | Fire Fişi | Waste/Loss Slip |
| 12 | - | Sarf Fişi | Consumption Slip |
| 13 | 0 | Alınan Fiyat Farkı Faturası | Received Price Difference Invoice |
| 13 | > 0 | Üretimden Giriş Fişi | Production Entry Slip |
| 14 | 0 | Verilen Fiyat Farkı Faturası | Issued Price Difference Invoice |
| 14 | > 0 | Devir Fişi | Transfer Slip |
| 25 | - | Ambar Fişi | Warehouse Slip |
| 50 | - | Sayım Fazlası | Inventory Surplus |
| 51 | - | Sayım Eksiği | Inventory Shortage |

---

## Orfiche Table (Orders)

| TRCODE | Turkish | English |
|--------|---------|---------|
| 1 | Alınan Sipariş | Sales Order (Received) |
| 2 | Verilen Sipariş | Purchase Order (Issued) |

---

## Csroll Table (Cheque & Promissory Note Transactions)

| TRCODE | Turkish | English |
|--------|---------|---------|
| 1 | Çek Girişi | Cheque Entry |
| 2 | Senet Girişi | Promissory Note Entry |
| 3 | Çek Çıkış Cari Hesaba | Cheque Out to Current Account |
| 4 | Senet Çıkış Cari Hesaba | Promissory Note Out to Current Account |
| 5 | Çek Çıkış Banka Tahsil | Cheque Out for Bank Collection |
| 6 | Senet Çıkış Banka Tahsil | Promissory Note Out for Bank Collection |
| 7 | Çek Çıkış Banka Teminat | Cheque Out as Bank Guarantee |
| 8 | Senet Çıkış Banka Teminat | Promissory Note Out as Bank Guarantee |
| 9 | İşlem Bordrosu Müşteri Çeki | Transaction Payroll - Customer Cheque |
| 10 | İşlem Bordrosu Müşteri Senedi | Transaction Payroll - Customer Note |
| 11 | İşlem Bordrosu Kendi Çekimiz | Transaction Payroll - Our Own Cheque |
| 12 | İşlem Bordrosu Kendi Senedimiz | Transaction Payroll - Our Own Note |
| 13 | İşyerleri Arası İ.Bord. Müşteri Çeki | Inter-Branch Transaction - Customer Cheque |

---

## Key Points

- The same TRCODE value can represent **different document types** depending on the table.
- In **Stline**, the `STFICHEREF` column further differentiates the document type.
- In **Paytrans**, the `MODULENR` column determines the module context (see table below).

### Paytrans MODULENR Values

| MODULENR | Module |
|----------|--------|
| 3 | Orders |
| 4 | Invoice |
| 5 | Current Account (Cari) |
| 6 | Cheque & Promissory Notes |
| 7 | Bank |
| 10 | Cash |
| 61 | Current Account (Special) |

---

*Logo Tiger ERP - SQL Level Reference*  
*Source: fn_trcode function analysis*

# Anforderungsdefinition ERP-Webshop Schnittstelle

Dieses Dokument beschreibt die Anforderungen an die Schnittstelle zwischen dem neuen Webshop und unserem ERP-System.

---

## Inhaltsverzeichnis

1. [Allgemeine Anforderungen](#1-allgemeine-anforderungen)
2. [Preisabfrage Kunden-Nettopreise](#2-preisabfrage-kunden-nettopreise)
3. [Bestellungen und Anfragen](#3-bestellungen-und-anfragen)
4. [Änderungen und Stornierungen](#4-änderungen-und-stornierungen)
5. [ERP-Dokumente im Webshop (DMS)](#5-erp-dokumente-im-webshop-dms)
6. [Technologieentscheidung](#6-technologieentscheidung)
7. [Authentifizierung und Sicherheit](#7-authentifizierung-und-sicherheit)
8. [Logging und Fehlerbehandlung](#8-logging-und-fehlerbehandlung)
9. [Beispiele](#9-technische-beispiele)

---

## 1. Allgemeine Anforderungen

Die Schnittstelle soll eine bidirektionale Echtzeit-Kommunikation ermöglichen und folgende Kriterien erfüllen:

- Zugriff auf verschiedene ERP-Mandanten
- Sichere Kommunikation (HTTPS/TLS)
- Klare API-Versionierung
- Umfangreiche Logging- und Monitoring-Möglichkeiten

---

## 2. Preisabfrage Kunden-Nettopreise

### 2.1 Einzelartikel-Abfrage

**Request**

- Kundennummer
- Artikelnummer
- Menge
- Mandant (Firma)
- Datum (optional, Standard = aktuell)

**Response**

- Brutto-/Netto-Preis
- Rabattinfos
- Preisgültigkeit

### 2.2 Warenkorb-Abfrage

**Request**

- Kundennummer
- Artikelliste (Artikelnummer + Menge)
- Mandant

**Response**

- Artikelpreise (Netto/Brutto)
- Gesamtsumme (Netto/Brutto)
- Rabattinfos gesamt
- Statusmeldungen (z.B. Verfügbarkeit)

---

## 3. Bestellungen und Anfragen

### 3.1 Bestellung erfassen

Die Bestellung aus dem Webshop wird sofort im ERP angelegt.

**Request**

- Kundennummer oder Gastdaten
- Rechnungs- und Lieferadresse
- Artikelliste (Preis/Menge)
- Versand/Zahlungsart
- Mandant

**Response**

- ERP-Auftragsnummer
- Status/Fehlermeldungen

### 3.2 Anfrage als Angebot erfassen

Anfragen werden als Angebot im ERP gespeichert.

**Request/Response analog Bestellung**

- Rückgabe: ERP-Angebotsnummer und Angebotsgültigkeit

---

## 4. Änderungen und Stornierungen

Da Änderungen und Stornierungen abhängig vom Auftragsstatus eingeschränkt möglich sind, wird folgender Prozess empfohlen:

| Status im ERP | Änderungsmöglichkeit | Vorgeschlagener Prozess                  |
|---------------|----------------------|------------------------------------------|
| Offen         | ✅                   | Direkte Änderung via API möglich         |
| In Bearbeitung| ⚠️                   | Manuelle Prüfung durch Vertrieb (Workflow)|
| Versendet     | ❌                   | Nur Retoure-Prozess möglich              |

- Der Webshop zeigt Statusinformationen und ermöglicht Workflow-Anfragen.
- ERP gibt stets klaren Status und Feedback zurück.

---

## 5. ERP-Dokumente im Webshop (DMS)

Die Dokumente (Angebote, Aufträge, Lieferscheine, Rechnungen, Gutschriften) werden dem Kunden bereitgestellt.

### Empfehlung: Live-Abfrage (ohne Replikation)

- Abruf direkt über sichere URL aus dem ERP-DMS.
- Keine doppelte Datenhaltung erforderlich.
- Voraussetzung: ausreichende Performance.

### Optional: Replikation (bei Performanceproblemen)

- Synchronisation der Dokumente periodisch oder ereignisgesteuert in Cloud-Storage des Webshops.

---

## 6. Technologieentscheidung

REST-API (JSON-basiert) wird empfohlen.

| Merkmal              | REST-API                | XML-Dateiaustausch       |
|----------------------|-------------------------|--------------------------|
| Geschwindigkeit      | ✅ Echtzeit             | ⚠️ Verzögert             |
| Komplexität          | ✅ Einfach              | ⚠️ Hoch                  |
| Erweiterbarkeit      | ✅ Einfach              | ⚠️ Schwierig             |
| Validierung          | ✅ Einfach (JSON Schema)| ⚠️ Komplex (XSD)         |
| Technologie-Standard | ✅ Modern               | ⚠️ Legacy                |

---

## 7. Authentifizierung und Sicherheit

- HTTPS/TLS-Verschlüsselung
- Authentifizierung via OAuth2/JWT oder API-Key
- Optional: IP-Whitelisting

---

## 8. Logging und Fehlerbehandlung

- Einheitliches Error-Response-Format
- Zentrale Protokollierung im ERP
- Fehlerstatus und Meldungen im Webshop verständlich darstellen

---

## 9. Technische Beispiele

### 9.1 Warenkorb-Preisabfrage (Request)

```json
{
  "client_id": "12345",
  "company_code": "01",
  "items": [
    {"item_id": "ART12345", "quantity": 10},
    {"item_id": "ART67890", "quantity": 5}
  ]
}

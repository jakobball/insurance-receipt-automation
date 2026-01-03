# insurance-receipt-automation
# 🏥 AI Claims Automation 

> Eine intelligente Automatisierung zur Verarbeitung von Versicherungsanträgen mit OCR, DeepSeek LLM Analyse und automatisierter Kundenkommunikation.

![n8n](https://img.shields.io/badge/Workflow-n8n-red?style=flat-square&logo=n8n)
![DeepSeek](https://img.shields.io/badge/AI-DeepSeek_V3-blue?style=flat-square)
![OCR](https://img.shields.io/badge/Vision-OCR.space-orange?style=flat-square)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-green?style=flat-square&logo=google-sheets)

## 📖 Über das Projekt

Dieses Projekt demonstriert einen End-to-End-Prozess für das Einreichen und Prüfen von Rückerstattungsanträgen (z.B. Arztrechnungen). Es ersetzt manuelle Dateneingabe durch KI-gestützte Extraktion und Validierung.

Das System besteht aus zwei Haupt-Workflows:
1.  **Ingestion & Analysis:** Empfang der Dokumente, OCR, KI-Validierung und Speicherung.
2.  **Validation & Response:** Automatisierte, aber personalisierte E-Mail-Kommunikation basierend auf der Entscheidung (Genehmigt/Abgelehnt/Rückfrage).

## ⚙️ Architektur & Workflows

### 1. Claims Ingestion (` Decision.json`)
Dieser Workflow ist der Einstiegspunkt für neue Anträge.

* **Multi-Input:** Akzeptiert sowohl Bild-Uploads (Belege) als auch strukturierte JSON-Daten.
* **OCR Engine:** Nutzt die **OCR.space API** zur Texterkennung auf Belegen/Bildern.
* **AI Extraction (DeepSeek):** Ein AI-Agent extrahiert strukturierte Daten (Name, Betrag, Diagnose) aus dem OCR-Rohdaten.
* **AI Fraud & Risk Check:** Ein zweiter AI-Agent bewertet den Fall:
    * *Labeling:* Kategorisiert den Antrag (z.B. "Zahnreinigung", "Medikament").
    * *Confidence Score:* Berechnet eine Wahrscheinlichkeit (0.0 - 1.0) für die Korrektheit.
    * *Status Empfehlung:* Setzt Status auf `APPROVED` (<100€ & plausibel), `DECLINED` oder `NEEDS_TALK`.
* **Datenbank:** Speichert den Fall inkl. KI-Bewertung in **Google Sheets**.
* **Bestätigung:** Sendet automatisch eine Eingangsbestätigung per Gmail.

### 2. Decision Communicator (` Validator.json`)
Dieser Workflow kümmert sich um die finale Kommunikation mit dem Kunden.

* **Trigger:** Wird per Webhook ausgelöst (z.B. nach manueller Freigabe im Dashboard).
* **Context-Aware Emailing:** Nutzt **DeepSeek**, um eine passende E-Mail zu formulieren.
    * *Bei Genehmigung:* Formuliert eine freundliche Auszahlungsbestätigung.
    * *Bei Rückfragen:* Erklärt genau, welche Informationen fehlen (basierend auf dem Review-Kommentar).
    * *Bei Ablehnung:* Formuliert eine empathische, aber klare Absage.

## 🛠 Tech Stack

* **Orchestrator:** [n8n](https://n8n.io/)
* **LLM / AI:** DeepSeek Chat Model (via API) für Extraktion und Textgenerierung.
* **OCR:** [OCR.space](https://ocr.space/) API.
* **Datenbank:** Google Sheets.
* **Kommunikation:** Gmail API.

## 🚀 Installation & Setup

1.  **n8n Import:**
    * Importiere ` Decision.json` und ` Validator.json` in deine n8n Instanz.

2.  **Credentials hinterlegen:**
    Du benötigst folgende Accounts in n8n:
    * `DeepSeek API` (für die LLM Nodes).
    * `Google Sheets OAuth2`.
    * `Gmail OAuth2`.
    * *Hinweis:* Der OCR-Request im Workflow nutzt einen API-Key im Header/Body. Stelle sicher, dass dieser im `HTTP Request` Node aktualisiert wird.

3.  **Google Sheet Struktur:**
    Erstelle ein Tabellenblatt mit folgenden Spalten (Mapping siehe Node "Update row in sheet"):
    `id`, `date`, `name`, `email`, `amount`, `description`, `ai_status`, `ai_label`, `ai_topic`, `ai_conf`, `review_comment`

## 🔒 Datenschutz & Sicherheit

* Der Workflow filtert sensible Daten vor dem LLM-Prompting, wo möglich.
* API-Keys sind in diesem Repository **nicht** enthalten. Bitte eigene Keys in den n8n-Credentials hinterlegen.

---
*Entwickelt als Showcase für automatisierte Prozessoptimierung.*

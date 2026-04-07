# 🗳️ VoteBox – Digitale Abstimmungsplattform

Invite-basierte Online-Wahlen für Parteien, Verbände und Organisationen · *Node.js · Express 5 · React 19 · PostgreSQL · TOTP-Sicherung*

<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nodejs/nodejs-original.svg" width="40" title="Node.js" />&nbsp;
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/express/express-original.svg" width="40" title="Express" />&nbsp;
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/react/react-original.svg" width="40" title="React 19" />&nbsp;
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/vitejs/vitejs-original.svg" width="40" title="Vite 6" />&nbsp;
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/postgresql/postgresql-original.svg" width="40" title="PostgreSQL" />&nbsp;

---

## 📋 Inhaltsverzeichnis

- [Überblick](#überblick)
- [Wahlfluss im Detail](#wahlfluss-im-detail)
- [Kernfunktionen](#kernfunktionen)
- [Technische Architektur](#technische-architektur)
- [Tech Stack](#tech-stack)
- [Schnellstart](#schnellstart)
- [Roadmap](#roadmap)
- [Was dieses Projekt zeigt](#was-dieses-projekt-zeigt)
- [Autor](#autor)

---

## <a id="überblick"></a>📖 Überblick

VoteBox ist eine sichere, einladungsbasierte Abstimmungsplattform für interne Wahlen in Parteien, Verbänden und vergleichbaren Organisationen.

Der Standardfluss ist bewusst einfach gehalten:

```
E-Mail erhalten → persönlichen Link öffnen → abstimmen → fertig
```

Das Projekt legt besonderen Wert auf:

- Starke Admin-Absicherung mit TOTP und HttpOnly-Session
- Reibungslosen Wählerzugang ohne zusätzliche Registrierung
- Datenschutz durch Datenminimierung, saubere Systemtrennung und automatisches Retention-Cleanup
- Schutz gegen Doppeltstimmen durch serverseitige Vote-Session-Logik
- Erweiterbare Multi-Tenant-Architektur (SaaS-Foundation in Aufbau)

**Zielbereich:** Parteiinterne Wahlen, verbandliche Abstimmungen, Pilotwahlen unter realen Bedingungen.

**Nicht Zielbereich:** Staatliche Wahlen, Kandidatenaufstellungen für Volksvertretungen, Verfahren mit abweichendem Spezialwahlrecht.

---

## <a id="wahlfluss-im-detail"></a>🔄 Wahlfluss im Detail

Jede Abstimmung durchläuft zwei klar getrennte Seiten: Wahlleitung und Wahlberechtigte.

### 🛠 Seite der Wahlleitung

**1️⃣ Vorbereitung**

Die Wahlleitung legt eine Abstimmung an, definiert Antwortoptionen sowie Start- und Endzeitpunkt und importiert die Wahlberechtigten per CSV.

**2️⃣ Einladungsversand**

Einladungen werden generiert und per SMTP versendet. Jede wahlberechtigte Person erhält einen individuellen, einmalig verwendbaren Link.

**3️⃣ Monitoring**

Die Wahlleitung verfolgt Versandstatus und Abstimmungsstatus in Echtzeit. Für Personen, die noch nicht abgestimmt haben, kann ein Re-Invite ausgelöst werden.

**4️⃣ Ergebnisse**

Nach Wahlende werden Ergebnisse freigegeben und können als CSV exportiert werden. Während der laufenden Wahl sind keine Zwischenergebnisse einsehbar.

---

### 🗳 Seite der Wahlberechtigten

**1️⃣ Einladung erhalten**

Die wahlberechtigte Person erhält eine E-Mail mit einem persönlichen Einladungslink. Keine Registrierung, kein Passwort, kein TOTP erforderlich.

**2️⃣ Link öffnen**

Der Invite-Link wird serverseitig sofort in eine kurzlebige Vote-Session umgetauscht. Der ursprüngliche Token ist danach ungültig.

**3️⃣ Stimme abgeben**

Die Person wählt eine Option und gibt die Stimme verbindlich ab. Doppeltstimmen sind durch die Session-Logik ausgeschlossen.

**4️⃣ Bestätigung**

Eine Bestätigungsseite schließt den Vorgang ab. Die Stimme ist gespeichert — ohne direkten Personenbezug.

---

## <a id="kernfunktionen"></a>✨ Kernfunktionen

### 🔐 Admin-Absicherung

- Login mit TOTP-Pflicht und QR-Code-Enrollment
- HttpOnly-Session-Cookie
- Rate-Limiting auf allen sensiblen Endpunkten
- Audit-Log für alle Admin- und Systemereignisse

### 📥 Wählerverwaltung

- CSV-Import von Wahlberechtigten
- Versandstatus pro Wähler in Echtzeit
- Einzelne Wähler deaktivieren oder Re-Invite auslösen
- Statusübersicht: abgestimmt / noch offen / deaktiviert

### 📨 Einladungssystem

- Individuelle Invite-Links per SMTP
- Re-Invite nur solange noch keine Stimme abgegeben wurde
- Automatische Invalidierung alter Links bei Re-Invite oder Deaktivierung

### 🛡 Datenschutz & Retention

- Stimmen werden ohne direkten Personenbezug gespeichert
- Saubere Trennung von Wahlberechtigung, Einladung, Session und Stimme
- Automatisches Retention-Cleanup: Personendaten werden nach Ablauf bereinigt, Ergebnisdaten bleiben erhalten

---

## <a id="technische-architektur"></a>🧠 Technische Architektur

VoteBox ist als klar getrenntes Frontend/Backend-System aufgebaut mit modularen, single-responsibility Services auf der Serverseite.

```
Frontend (React)  →  Backend API (Express)  →  PostgreSQL
```

**Architekturprinzipien:**

- Single-Responsibility-Services (ein Service, eine Zuständigkeit)
- Zentralisierte Session- und Auth-Kontrolle
- Erweiterbare Modulstruktur für Multi-Tenant-Ausbau
- Konsequente Datentrennung: Identität, Einladung, Session und Stimme sind entkoppelt

---

## <a id="tech-stack"></a>🛠 Tech Stack

| Bereich      | Technologie                                  |
|--------------|----------------------------------------------|
| Frontend     | React 19, React Router 7, Vite 6             |
| Backend      | Node.js ≥ 22, Express 5                      |
| Datenbank    | PostgreSQL 15                                |
| Auth         | TOTP, bcrypt, JWT, HttpOnly-Cookie           |
| Mail         | Nodemailer (SMTP)                            |
| Security     | Helmet, Rate-Limiting, CORS                  |
| Deployment   | Docker, nginx                                |
| Tests        | Node.js built-in test runner                 |

---

## <a id="schnellstart"></a>🚀 Schnellstart

### Voraussetzungen

- [Node.js](https://nodejs.org/) ≥ 22
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Git

### Repository klonen

```bash
git clone git@github.com:Codenix-1349/VoteBox-Showcase.git
cd VoteBox-Showcase
```

### Datenbank starten

```bash
docker compose up -d postgres
```

### Backend einrichten & starten

```bash
cd backend
cp .env.example .env   # DB, SMTP und APP_BASE_URL eintragen
npm install
npm run migrate
npm start
```

### Frontend starten

```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

---

## <a id="roadmap"></a>🛣 Roadmap

### 🏗 Phase 1–3: Tenant- & User-Grundlage

- Self-Service-Registrierung für Wahlleiter
- E-Mail-Verifikation, Passwort-Setup, TOTP-Enrollment
- KV-/Organisationsanlage im Self-Service
- Vollständige Tenant-Isolation für Polls, Wähler und Ergebnisse

### 🧩 Phase 4–5: Plattformkonsole & Lizenzmodell

- Plattformkonsole für Kundenverwaltung, Lizenzen und Sperrungen
- Kontrollierter Supportzugriff mit Auditlog
- Jahrespläne pro KV, Trial-Modus, Upgrade-Pfad

### 🎯 Phase 6: Self-Service-Produktreife

- Testwahl und Wahl kopieren
- Onboarding-Checkliste für neue Wahlleiter
- Reminder-Flow und verbesserte Zustelldarstellung

### 🔒 Phase 7: Betriebs- & Vertrauensschicht

- Backup/Restore mit Restore-Test
- Datenschutzinfo, AVV/DPA, AGB
- Monitoring, Alarmierung und Incident-Runbook

### 🚀 Phase 8: Pilot & Marktstart

- Pilot mit 2–5 echten Kreisverbänden
- Auswertung und gezielte Nacharbeit an echten Blockern

---

## <a id="was-dieses-projekt-zeigt"></a>🎯 Was dieses Projekt zeigt

- Invite-basierte Authentifizierungsarchitektur ohne Wähler-Registrierung
- Zweiphasiger Admin-Login (Passwort + TOTP) mit sicherer Session-Verwaltung
- Datenschutzkonforme Systemtrennung: Wahlberechtigung, Einladung, Session und Stimme sind entkoppelt
- Automatisiertes Retention-Cleanup ohne Ergebnisverlust
- Erweiterbare Multi-Tenant-Grundlage für SaaS-Ausbau
- Produktreifes PoC mit klarer Pilot- und Vermarktungsperspektive

---

## <a id="autor"></a>👨‍💻 Autor

**Patrick Neumann**

GitHub: [github.com/Codenix-1349](https://github.com/Codenix-1349)

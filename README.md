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
- [Datenmodell](#datenmodell)
- [Tech Stack](#tech-stack)
- [Schnellstart](#schnellstart)
- [Nutzung](#nutzung)
- [API-Überblick](#api-überblick)
- [Roadmap](#roadmap)
- [Was dieses Projekt zeigt](#was-dieses-projekt-zeigt)
- [Autor](#autor)

---

## 📖 Überblick

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

## 🔄 Wahlfluss im Detail

Jede Abstimmung durchläuft zwei klar getrennte Seiten: Wahlleitung und Wahlberechtigte.

### 🛠 Seite der Wahlleitung

**1️⃣ Vorbereitung**

Die Wahlleitung legt eine Abstimmung an, definiert Antwortoptionen sowie Start- und Endzeitpunkt und importiert die Wahlberechtigten per CSV (`display_name`, `email`, `external_ref`).

**2️⃣ Einladungsversand**

Einladungen werden generiert und per SMTP versendet. Jede wahlberechtigte Person erhält einen individuellen, einmalig verwendbaren Link.

**3️⃣ Monitoring**

Die Wahlleitung verfolgt Versandstatus und Abstimmungsstatus in Echtzeit. Für Personen, die noch nicht abgestimmt haben, kann ein Re-Invite ausgelöst werden — so lange, bis die Stimme abgegeben wurde.

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

## ✨ Kernfunktionen

### 🔐 Admin-Absicherung

- Login mit TOTP-Pflicht (QR-Code-Enrollment beim ersten Login)
- HttpOnly-Session-Cookie
- Rate-Limiting auf allen Admin-Endpunkten
- Audit-Log für alle Admin- und Systemereignisse

### 📥 Wählerverwaltung

- CSV-Import mit `display_name`, `email`, `external_ref`
- Versandstatus pro Wähler: `erstellt → angefordert → versendet → fehlgeschlagen → benutzt`
- Einzelne Wähler deaktivieren oder Re-Invite auslösen
- Statusübersicht: abgestimmt / noch offen / deaktiviert

### 📨 Einladungssystem

- Individuelle Invite-Links per SMTP
- Serverseitiger Token-zu-Session-Tausch
- Re-Invite nur solange noch keine Stimme abgegeben wurde
- Automatische Invalidierung alter Links bei Re-Invite oder Deaktivierung

### 🛡 Datenschutz & Retention

- Stimmen werden ohne direkten Personenbezug gespeichert
- Saubere Trennung: Wahlberechtigung, Invite-Artefakte, Vote-Sessions, Stimmen
- Keine Roh-Tokens in Logs
- Automatisches Retention-Cleanup: poll-lokale Personendaten werden nach Ablauf bereinigt, Ergebnisdaten bleiben erhalten

---

## 🧠 Technische Architektur

```
VoteBox/
├── backend/
│   ├── src/
│   │   ├── routes/          # adminAuth, polls, voters, results, voteAuth, votes
│   │   ├── services/        # adminAuthService, adminSessionService,
│   │   │                    # emailService, pollRetentionService,
│   │   │                    # voteAccessService, voteSessionService,
│   │   │                    # userAuthService
│   │   ├── middleware/      # adminSessionGuard, adminRateLimiter, rateLimiter,
│   │   │                    # authGuard, userSessionGuard
│   │   └── utils/           # crypto, logger, pollLifecycle, totp,
│   │                        # pollRetention, proxy, publicError
│   ├── migrations/          # 016 SQL-Migrationen (inkl. SaaS-Foundation)
│   ├── scripts/             # retentionCleanup.js
│   ├── seed.js
│   └── migrate.js
│
├── frontend/
│   └── src/
│       ├── pages/           # AdminLogin, AdminDashboard, AdminAccess,
│       │                    # UserAccess, UserHome, VotePage,
│       │                    # ConfirmPage, VerifyEmailPage
│       ├── components/      # CreatePollForm, PollCard, PollDetail,
│       │                    # VoterList, VoterImportPanel, InviteActions,
│       │                    # VoterStatusBadge
│       └── api/             # client.js
│
├── docs/                    # Roadmaps, Runbook, Retention-Policy
├── Konzept_Pitch/           # Pitchdeck, Projektplan, DSGVO-Konzept
└── docker-compose.yml
```

**Architekturprinzipien:**

- Single-Responsibility-Services (ein Service, eine Zuständigkeit)
- Zentralisierte Session- und Auth-Kontrolle
- Erweiterbare Modulstruktur für Multi-Tenant-Ausbau
- Daten-getriebene Trennung von Identität, Einladung und Stimme

---

## 🗃️ Datenmodell

| Tabelle                             | Beschreibung                                         |
|-------------------------------------|------------------------------------------------------|
| `polls`                             | Abstimmungen                                         |
| `choices`                           | Antwortoptionen pro Abstimmung                       |
| `eligible_voters`                   | Poll-lokale Wahlberechtigungen                       |
| `tokens`                            | Persönliche Einladungsartefakte                      |
| `vote_sessions`                     | Kurzlebige Sessions für die Stimmabgabe              |
| `votes`                             | Abgegebene Stimmen ohne direkten Personenbezug       |
| `audit_events`                      | Admin- und Systemereignisse ohne Stimmzuordnung      |
| `users` *(SaaS)*                    | Wahlleiter- und Plattformkonten                      |
| `organizations` *(SaaS)*            | KV-/Tenant-Kontext                                   |
| `organization_memberships` *(SaaS)* | Zuordnung Nutzer ↔ Organisationen                    |
| `user_sessions` *(SaaS)*            | Session-Basis für künftige Wahlleiter-Logins         |

---

## 🛠 Tech Stack

| Bereich      | Technologie                                  |
|--------------|----------------------------------------------|
| Frontend     | React 19, React Router 7, Vite 6             |
| Backend      | Node.js ≥ 22, Express 5                      |
| Datenbank    | PostgreSQL 15                                |
| Auth         | TOTP (qrcode), bcrypt, JWT, HttpOnly-Cookie  |
| Mail         | Nodemailer (SMTP)                            |
| Security     | Helmet, express-rate-limit, CORS             |
| Deployment   | Docker, docker-compose, nginx                |
| Tests        | Node.js built-in test runner (`node --test`) |

---

## 🚀 Schnellstart

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

### Backend einrichten

```bash
cd backend
cp .env.example .env   # .env anpassen: DB, SMTP, APP_BASE_URL
npm install
npm run migrate
npm run seed
```

**Demo-Login nach dem Seed:**

- Benutzername: `admin`
- Passwort: `admin123`

> Beim ersten Admin-Login wird TOTP für den Adminzugang eingerichtet.

### Backend starten

```bash
npm start
# → http://localhost:3000
```

### Frontend starten

```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

### Lokale Mailtests

Für echte Einladungstests einen SMTP-Relay in `backend/.env` eintragen. Ohne SMTP bleibt der Mailversand deaktiviert.

Details: [`docs/local-realmail-test.md`](docs/local-realmail-test.md) · CSV-Vorlage: [`docs/voter-import-live-test-template.csv`](docs/voter-import-live-test-template.csv)

---

## 💡 Nutzung

### Wahlleitung (Admin)

1. Im Adminbereich anmelden — Login + TOTP-Verify
2. Poll anlegen und Antwortoptionen definieren
3. Wahlberechtigte per CSV importieren
4. Einladungen generieren und per E-Mail versenden
5. Versand- und Abstimmungsstatus verfolgen
6. Bei Bedarf Re-Invite für noch nicht abstimmende Personen auslösen
7. Ergebnisse nach Wahlende einsehen oder als CSV exportieren

### Wahlberechtigte

1. Einladungs-E-Mail erhalten
2. Persönlichen Link öffnen
3. Vote-Session wird serverseitig erzeugt
4. Option auswählen und Stimme verbindlich abgeben
5. Bestätigungsseite

---

## 🔌 API-Überblick

### Öffentliche Endpunkte

| Methode | Route                         | Beschreibung         |
|---------|-------------------------------|----------------------|
| GET     | `/api/health`                 | Health-Check         |
| POST    | `/api/vote-auth/access-state` | Vote-Session-Status  |
| POST    | `/api/vote`                   | Stimme abgeben       |

### Admin-Endpunkte (Session-gesichert)

| Methode | Route                                              | Beschreibung               |
|---------|----------------------------------------------------|----------------------------|
| POST    | `/api/auth/login`                                  | Login (Schritt 1)          |
| POST    | `/api/auth/login/verify`                           | TOTP-Verify (Schritt 2)    |
| POST    | `/api/auth/setup/confirm`                          | TOTP-Setup bestätigen      |
| GET     | `/api/auth/session`                                | Session prüfen             |
| POST    | `/api/auth/logout`                                 | Logout                     |
| GET     | `/api/polls`                                       | Alle Polls abrufen         |
| POST    | `/api/polls`                                       | Poll erstellen             |
| GET     | `/api/polls/:id`                                   | Poll-Details               |
| DELETE  | `/api/polls/:id`                                   | Poll löschen               |
| GET     | `/api/polls/:id/voters`                            | Wählerliste                |
| POST    | `/api/polls/:id/voters/import`                     | CSV-Import                 |
| PATCH   | `/api/polls/:id/voters/:voterId/disable`           | Wähler deaktivieren        |
| POST    | `/api/polls/:id/voters/:voterId/regenerate-invite` | Re-Invite                  |
| POST    | `/api/polls/:id/invitations/generate`              | Einladungen generieren     |
| GET     | `/api/polls/:id/results`                           | Ergebnisse (nach Wahlende) |
| GET     | `/api/polls/:id/results/csv`                       | Ergebnisse als CSV         |

---

## 🛣 Roadmap

### 🏗 Phase 1–3: Tenant- & User-Grundlage

- Self-Service-Registrierung für Wahlleiter
- E-Mail-Verifikation, Passwort-Setup, TOTP-Enrollment
- KV-/Organisationsanlage im Self-Service
- Tenant-Isolation: Polls, Wähler und Ergebnisse pro Organisation getrennt

### 🧩 Phase 4–5: Plattformkonsole & Lizenzmodell

- Plattformkonsole für Kundenverwaltung, Lizenzen und Sperrungen
- Kontrollierter Supportzugriff mit Auditlog (kein Klartextzugriff auf Credentials)
- Jahrespläne pro KV, Trial-Modus, Upgrade-Pfad

### 🎯 Phase 6: Self-Service-Produktreife

- Testwahl und Wahl kopieren
- Onboarding-Checkliste für neue Wahlleiter
- Reminder-Flow und verbesserte Zustelldarstellung
- In-App-Hilfe und Supporthinweise

### 🔒 Phase 7: Betriebs- & Vertrauensschicht

- Backup/Restore mit Restore-Test
- Datenschutzinfo, AVV/DPA, AGB
- Monitoring, Alarmierung und Incident-Runbook

### 🚀 Phase 8: Pilot & Marktstart

- Pilot mit 2–5 echten Kreisverbaänden
- Auswertung: Registrierung, Zustellung, Supportlast, Zeit bis zur ersten Wahl
- Gezielter Nachbau nur an echten Blockern

---

## 🎯 Was dieses Projekt zeigt

- Invite-basierte Authentifizierungsarchitektur ohne Wähler-Registrierung
- Zweiphasiger Admin-Login (Passwort + TOTP) mit sicherer Session-Verwaltung
- Datenschutzkonforme Systemtrennung: Wahlberechtigung, Einladung, Session und Stimme sind entkoppelt
- Automatisiertes Retention-Cleanup ohne Ergebnisverlust
- Erweiterbare Multi-Tenant-Grundlage für SaaS-Ausbau
- Produktreifes PoC mit klarer Pilot- und Vermarktungsperspektive

---

## 👨‍💻 Autor

**Patrick Neumann**

GitHub: [github.com/Codenix-1349](https://github.com/Codenix-1349)

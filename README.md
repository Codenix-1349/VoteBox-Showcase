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
- [Screenshots](#screenshots)
- [Wahlfluss im Detail](#wahlfluss-im-detail)
- [Kernfunktionen](#kernfunktionen)
- [Technische Architektur](#technische-architektur)
- [Tech Stack](#tech-stack)
- [Schnellstart](#schnellstart)
- [Roadmap](#roadmap)
- [Was dieses Projekt zeigt](#was-dieses-projekt-zeigt)
- [Autor](#autor)

---

## <a id="screenshots"></a>📸 Screenshots

<table>
  <tr>
    <td align="center" width="50%">
      <img src="https://github.com/user-attachments/assets/d278127e-15a9-4322-8bd5-f4641f79a324"
           width="300" alt="Admin-Login" /><br/>
      <sub><b>Admin-Login mit TOTP-Absicherung</b></sub>
    </td>
    <td align="center" width="50%">
      <img src="https://github.com/user-attachments/assets/f4e9f4ee-01ba-4c0b-bd23-96a08ef2f9f9"
           width="300" alt="Abstimmung erstellen" /><br/>
      <sub><b>Neue Abstimmung erstellen</b></sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="https://github.com/user-attachments/assets/f30f728d-9c18-43a2-a05f-f63d34ac98ec"
           width="300" alt="Stimmabgabe" /><br/>
      <sub><b>Stimmabgabe durch Wahlberechtigte</b></sub>
    </td>
    <td align="center" width="50%">
      <img src="https://github.com/user-attachments/assets/e93c04b7-99b2-4281-8533-24781b43f69e"
           width="300" alt="Abstimmungsübersicht" /><br/>
      <sub><b>Abstimmungsübersicht (Admin)</b></sub>
    </td>
  </tr>
  <tr>
    <td align="center" colspan="2">
      <img src="https://github.com/user-attachments/assets/817ef4ce-f73b-4865-88a7-c72527e33d5a"
           width="620" alt="CSV-Import" /><br/>
      <sub><b>CSV-Import der Wahlberechtigten</b></sub>
    </td>
  </tr>
</table>

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

**2️⃣ Einladungsversand**

**3️⃣ Monitoring**

**4️⃣ Ergebnisse**



---

### 🗳 Seite der Wahlberechtigten

**1️⃣ Einladung erhalten**

**2️⃣ Link öffnen**

**3️⃣ Stimme abgeben**

**4️⃣ Bestätigung**


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

# 🌍 gdi2go - Geodateninfrastruktur im Container

Ein portables, vollständig containerisiertes GIS-Labor basierend auf Docker. Dieses Repository startet einen kompletten Tech-Stack bestehend aus Datenbank, GIS-Servern und Management-Tools mit einem einzigen Befehl.

## 🚀 Quick Start

Voraussetzung: [Docker Desktop](https://www.docker.com/products/docker-desktop/) und Git sind installiert.

1. **Repository klonen**
```bash
git clone https://github.com/GeowazM/gdi2go.git
cd gdi2go
```
   
   ## Stack starten

```bash
docker compose up -d
```

(Bitte warte beim ersten Start ca. 30-60 Sekunden, bis die Datenbank vollständig initialisiert ist.)

#### Status prüfen
```bash
docker ps
```

### 📦 Enthaltene Komponenten & Zugangsdaten
Hier sind alle Dienste aufgelistet, die gestartet werden.

| Dienst | URL / Zugang | Port (Extern) | Benutzer | Passwort | Beschreibung |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PostGIS - Datenbank** | `localhost` | `5433` ⚠️ | `admin` | `sicherheitspasswort123` | [PostgreSQL + PostGIS 15 3.4](https://postgis.net/) |
| **pgAdmin 4 - DBMS** | [http://localhost:5050](http://localhost:5050) | `5050` | `admin@admin.com` | `admin` | [Datenbankmanagementsystem](https://www.pgadmin.org/) |
| **GeoServer - WMS/WFS** | [http://localhost:8080/geoserver](http://localhost:8080/geoserver) | `8080` | `admin` | `geoserver` | [OGC-konforme Dienste bereitsellen](https://geoserver.org/) |
| **MapProxy - Caches** | [http://localhost:8085](http://localhost:8085) | - | - | - | [Geospatial caches](https://mapproxy.org/) |
| **Geonetwork MIS** | [http://localhost:8086/](http://localhost:8086/) | `8086` | `admin` | `admin` | [Geonetwork (4.4) - Metadateninformationssystem](https://www.geonetwork-opensource.org/) |

Für mehr Details siehe unten .yaml Datei im gdi2go repo.
⚠️ Wichtiger Hinweis zum Datenbank-Port:Um Konflikte mit lokalen PostgreSQL-Installationen zu vermeiden, ist die Datenbank extern auf Port 5433 gemappt (intern läuft sie auf 5432).

## 🔌Verbindung mit QGIS Desktop

Um dich von deinem lokalen QGIS (außerhalb von Docker) mit der Datenbank zu verbinden, nutze folgende Einstellungen:
- Host: localhost
- Port: 5433 (wichtig)
- Datenbank: gis_data
- Benutzername: admin
- Passwort: sicherheitspasswort123
- SSL Mode: disable (oder allow)

## 🛠️ Einrichtung & Tipps: pgAdmin Server hinzufügen
Nach dem Login in pgAdmin (Port 5050, admin) muss der Server einmalig registriert werden, da pgAdmin im Container läuft:
- Rechtsklick auf "Servers" -> Register -> Server
- Tab General: Name frei wählbar (z.B. "Docker DB")
- Tab Connection:Host name: gis_db (⚠️ Hier NICHT localhost verwenden, da wir im Docker-Netzwerk sind!)
- Port: 5432Username: admin
- Password: sicherheitspasswort123
- Troubleshooting: Datenbank-Login fehlgeschlagen?

Falls Authentifizierungsfehler auftreten ("password authentication failed"), wurde das Datenbank-Volume eventuell mit alten Daten oder einem alten Passwort erstellt.Lösung (Hard Reset):
Dies löscht die Datenbank und setzt sie mit dem Passwort aus der YAML neu auf.
⚠️ Achtung: Löscht alle gespeicherten Geodaten in der DB! 📂 
```bash
Bashdocker compose down -v
```

```bash
docker compose up -d
```
Ordnerstruktur
- docker-compose.yml: Die Definition aller Dienste.
- .gitignore: Verhindert, dass lokale Datenbank-Dateien hochgeladen werden
- .volumes/: (Wird automatisch erstellt) Hier liegen die persistenten Daten lokal bei dir
- .init.sql: (Optional) SQL-Skripte, die beim ersten Start ausgeführt werden (Platz für eine Demo).



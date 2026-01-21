# 🌍 gdi2go - Geodateninfrastruktur im Container

Ein portables, containerisierte **open source Geodateninfrastruktur (GDI)** basierend auf Docker. Dieses Repository bietet über einen `Docker stack` eine Software Architektur für Geodaten bestehend aus einer räumlicher Datenbank, einem räumlichen Servern und Management-Tools mit einem Befehl.

# 1. Quick Start 🚀

Voraussetzung: [Docker Desktop](https://www.docker.com/products/docker-desktop/) und Git sind installiert.

## 1.1 Repository **klonen**
```bash
git clone https://github.com/GeowazM/gdi2go.git
cd gdi2go
```
   
## 1.2 Stack starten
Im Ordner ```database_init``` gibt es eine gezippte SQL-Datenbank. Hier sind Beispieldaten enthalten, die zu Trainingszwecken in *pgAdmin4* und dem *Geoserver* genutzt werden können. Entpacke diese und führe anschließend im Terminal folgenden Befehl aus:

```bash
docker compose up -d
```

(Beim ersten Start ca. 2-3 Minuten warten. Bis die Datenbank vollständig initialisiert ist.)

## 1.3  Status prüfen
```bash
docker ps
```
Hier lassen sich die laufenden Docker-Container inklusive Ports einsehen. Diese lassen sich aber auch in Docker Desktop einsehen und können hier. In Docker Desktop können die Ports auch über eine Verlinkung aufgerufen werden. Als nächstes findet sich eine Liste dieser Container und deren Ports, die ein Aufruf zu den Anwendungen ermöglichen. 

## 1.4  Enthaltene Komponenten & Zugangsdaten 📦
Hier sind alle Dienste aufgelistet, die gestartet werden.

| Dienst | URL / Zugang | Port (Extern) | Benutzer | Passwort | Beschreibung |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PostGIS** *Datenbank* | `localhost` | `5433` ⚠️ | `postgres` | `sicherheitspasswort123` | [PostgreSQL + PostGIS 15 3.4](https://postgis.net/) |
| **pgAdmin** *DBMS* | [http://localhost:5050](http://localhost:5050) | `5050` | `admin@admin.com` | `admin` | [Datenbankmanagementsystem](https://www.pgadmin.org/) |
| **GeoServer** *WMS/WFS* | [http://localhost:8080/geoserver](http://localhost:8080/geoserver) | `8080` | `admin` | `geoserver` | [OGC-konforme Dienste bereitsellen](https://geoserver.org/) |
| **MapProxy** - *Caches* | [http://localhost:8085](http://localhost:8085) | - | - | - | [Geospatial caches](https://mapproxy.org/) |
| **Geonetwork** *MIS* | [http://localhost:8086/](http://localhost:8086/) | `8086` | `admin` | `admin` | [Geonetwork (4.4) - Metadateninformationssystem](https://www.geonetwork-opensource.org/) |

Für mehr Details siehe docker-compose.yaml Datei in diesem Repo.
⚠️ Wichtiger Hinweis zum Datenbank-Port: Um Konflikte mit lokalen PostgreSQL-Installationen zu vermeiden, ist die Datenbank extern auf Port 5433 geleitet (intern läuft sie auf 5432).

---

# 2. Initialisierung 🔌
Wir werden zuerst einen Blick in die *PostGIS*-Datenbank werfen. Dort befindet sich eine kleine Testdatenbank mit derer wir die GDI kennenlernen werden. Die Einsicht in die Geodatenbank erhalten wir mit Hilfe von *pgAdmin4*, dass ebenfalls Teil dieser GDI. Im Anschluss werden wir den *GeoServer* mit der *PostGIS*-Datenbank verbinden und einen WMS-Layer erstellen. Als drittes Verbinden wir unser lokales *QGIS Desktop* mit unserer GDI (PostGIS-Datenbank & WMS-Layer).

## 2.1 pgAdmin: Einen Server hinzufügen 🐘
Nach dem Login in pgAdmin (Port 5050) muss der Server einmalig registriert werden, da pgAdmin im Container läuft:
- Rechtsklick auf "Servers" -> Register -> Server
- General: Name frei wählbar (z.B. "Docker DB")

Connection: 
- Host name: gis_db (⚠️ nicht localhost verwenden, da innerhalb vom Docker-Netzwerk)
- Port: 5432
- Username: postgres
- Password: sicherheitspasswort123

## 2.2 GeoServer: Eine Verbindung zur Geodatenbank herstellen ⚙
Nach dem Login (name: admin, pw: geoserver) in GeoServer (Port 8080/geoserver) erstelle einen Arbeitsbereich mit einem Namen (bspw. TestBereich).
Unter Datenspeicher (Stores) -> Datenspeicher hinzufügen (Add new Store) ->  PostGIS - PostGIS Database | lässt sich eeine neue Datenbankverbindung herstellen.

| Feld | Wert | Erklärung |
| :--- | :--- | :--- |
| **Host** | `gis_db` | Service-Name aus der docker-compose.yaml |
| **Port** | `5432` | Der interne Container-Port |
| **Database** | `gis_data` | Name der Datenbank (POSTGRES_DB) |
| **Schema** | `public` | Standard-Schema |
| **User** | `postgres` | Datenbank-Benutzer |
| **Password** | `sicherheitspasswort123` | Datenbank-Passwort |

### GeoServer gibt SCRAM-Fehler zurück
Sie bedeutet, dass Host (gis_db) und Port (5432) korrekt sind, da der Server (die Datenbank) antwortet.
Der Fehler ```The server requested SCRAM-based authentication, but no password was provided besagt```, dass Postgres (Version 15 nutzt standardmäßig die moderne SCRAM-Verschlüsselung) auf das Passwort wartet, aber der GeoServer keines oder ein leeres Passwort sendet. Der GeoServer hat eine Eigenheit: Wenn man die Verbindungsdaten bearbeitet und speichert, wird das Passwortfeld aus Sicherheitsgründen oft wieder geleert (es stehen zwar Punkte drin, aber es wird nicht immer gesendet).

Lösung:

Gehe im GeoServer zurück in den Dialog für die Datastore-Verbindung > Lösche den Inhalt des Feldes Password > Tippe das Passwort ```sicherheitspasswort123``` erneut explizit ein. Scrolle ganz nach unten und klicke auf Speichern.

## 2.3 Verbindung mit QGIS Desktop 🛠️ 

Um dein lokalen QGIS Desktop (außerhalb von Docker) mit der *PostGIS*-Datenbank zu verbinden, nutze folgende Einstellungen:
- Host: localhost
- Port: 5433 (wichtig)
- Datenbank: gis_data
- Benutzername: postgres
- Passwort: sicherheitspasswort123
- SSL Mode: disable (oder allow)


---

# 3. Verbindungsprobleme

⚠️Falls Authentifizierungsfehler auftreten ("password authentication failed"), wurde das Datenbank-Volume ggf. mit alten Daten oder einem alten Passwort erstellt. Potenzielle Lösung (Hard Reset):
Dies löscht die Datenbank und setzt sie mit dem Passwort aus der YAML neu auf.

```bash
docker compose down -v
```
⚠️ Achtung: Löscht alle gespeicherten Geodaten in der Datenbank!
Danach erneut starten mit:
```bash
docker compose up -d
```
Ordnerstruktur
- docker-compose.yml: Die Definition aller Dienste.
- .gitignore: Verhindert, dass lokale Datenbank-Dateien hochgeladen werden
- .volumes/: (Wird automatisch erstellt) Hier liegen die persistenten Daten lokal bei dir
- .init.sql: (Optional) SQL-Skripte, die beim ersten Start ausgeführt werden (Platz für eine Demo).



# Das Verrueckte Labyrinth - API Dokumentation

Projekt im Rahmen der Lehrveranstaltung "Advanced Integrative Project", Masterstudiengang Digital Business & Software Engineering (MCI Management Center Innsbruck).

Dieses Repository enthaelt die verbindliche Schnittstellenspezifikation fuer die verteilte Umsetzung des Spiels "Das Verrueckte Labyrinth".

## Dokumentationsdateien

- `labyrint_api_doc.html`: Vollstaendige, interaktive API-Dokumentation im Browser (inklusive Parameter-Tabellen, JSON-Beispielen, ErrorCodes und Mobilansicht).
- `labyrint_api_doc.pdf`: Dokumentation fuer den Ausdruck / als PDF.

## Architekturueberblick

Das System gliedert sich in zwei Kommunikationsbereiche:

1. **Verzeichnisserver (REST-API):**
   - Basis-URL: `http://<host>:<port>/api/servers`
   - Ermoeglicht die Registrierung aktiver Spieleserver (`POST`), Heartbeats (`PUT`), Abmeldung (`DELETE`) sowie das Abrufen der Serverliste fuer Clients (`GET`).

2. **Spieleserver (WebSocket mit JSON-Envelope):**
   - Endpunkt: `ws://<host>:<port>/game`
   - Nachrichtenaufbau: Einheitliches Envelope-Format:
     ```json
     {
       "type": "MESSAGE_TYPE",
       "data": { ... }
     }
     ```

## Kernregeln der Schnittstelle

- **Authentifizierung & Reconnect:** Nach dem Verbindungsaufbau muss innerhalb von 10 Sekunden ein `CONNECT`-Befehl gesendet werden. Der Server liefert in `CONNECT_ACK` ein `identifierToken`, mit dem bei Verbindungsabbruechen innerhalb von 30 Sekunden reconnected werden kann.
- **Kachelrepraesentation:** Gitterfelder definieren Zugaenge ueber offene Kanten (`entrances: ["UP", "DOWN", "LEFT", "RIGHT"]`).
- **Zugablauf:** Streng rundenbasiert mit zwei Phasen:
  1. `ROTATE_SPARE_TILE` (optional) und `PUSH_TILE` (Schieben der Ersatzkachel in ungerade Reihen/Spalten).
  2. `MOVE_FIGURE` (Ziehen der Spielfigur entlang offener Wege) oder `PASS_MOVE` (Stehenbleiben).
- **Client-KI:** Die Logik zur Spielersteuerung ueber eine KI laeuft vollstaendig auf dem Client. Der Server wird ueber `SET_AI_MODE` informiert und loggt den Status.
- **Siegbedingung:** Alle geheimen Schatzkarten muessen gesammelt werden und die Figur muss auf ihr Startfeld (`homePosition`) zurueckkehren.


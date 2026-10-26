# Prüfprotokoll – Fokus 1.1

Stand: 26.10.2026

## Durchgeführt

### Browser (Headless Chromium)

- Lokaler Testbereich und Wiederherstellung nach Neuladen.
- Abschluss einer Folgeaufgabe gesperrt, bis ihre Voraussetzungen erledigt sind.
- Mehrere Voraussetzungen: A allein reicht für C nicht, wenn C auch B voraussetzt.
- Offene Checklistenpunkte sperren den Abschluss.
- Wiederöffnen einer Voraussetzung erhält abgeschlossene Folgeaufgaben und zeigt einen Hinweis.
- Zyklische Abhängigkeiten werden im Editor zurückgewiesen.
- Löschen einer noch benötigten Voraussetzung wird verhindert.
- HTML in Aufgabentiteln wird als Text dargestellt, nicht ausgeführt.
- Ungültige Kalenderdaten werden abgewiesen.
- Bearbeitbare KI-Vorschau und Übernahme der Beziehungen mit neuen IDs.
- Backup-Import samt ID-Neuvergabe und wieder geöffneten historischen Abschlüssen; fehlerhaftes JSON wird abgefangen.
- Darstellung bei 1440 px Desktopbreite und 390 px Mobilbreite; kein horizontaler Seitenüberlauf.
- Mobilansicht des Editors sowie Dark Mode geprüft.
- Keine JavaScript-Laufzeitfehler in diesen Browserabläufen.

### Synchronisationsablauf im Browser mit simuliertem Supabase

- Laden eines angemeldeten Kontos und Speichern per RPC.
- Versionskonflikt überschreibt keine Daten; Editoreingabe bleibt erhalten.
- Erneutes Synchronisieren lädt Änderungen eines simulierten zweiten Geräts.

### PostgreSQL (PGlite / tatsächliche Postgres-Engine)

- SQL-Skript zweimal erfolgreich ausgeführt (idempotent).
- RLS: Ein Nutzer kann den Stand des anderen nicht lesen.
- Direkte Schreibzugriffe als authenticated verweigert; Schreiben nur über geprüfte RPC.
- Anonyme RPC-Aufrufe verweigert.
- Versionsprüfung verhindert veraltete Schreibvorgänge.
- Kreise, fehlende Voraussetzungen und ungültige Daten werden serverseitig abgewiesen.
- Abschlussprüfung und Wiederöffnen mit erhaltener Historie.
- KI-Kontingent: Aufruf 1–40 zugelassen, 41 abgewiesen.

### Edge Function (simulierte Auth-/Gemini-Antworten)

- TypeScript-Syntax geprüft.
- Ohne Auth-Header bzw. bei ungültiger Anmeldung kein KI-Aufruf.
- Quotenfehler und Gemini-429 sauber abgefangen.
- Kreise in KI-Antworten zurückgewiesen.
- Gültige strukturierte Aufgabenliste zurückgegeben.
- OPTIONS/CORS und HTTP-Methodenprüfung.

## Noch nicht live geprüft

Es wurden keine Änderungen an deinem echten Supabase-Projekt vorgenommen und keine echten Gemini-Anfragen versendet. Kein Zugriff auf dein GitHub-Konto und kein Deployment. Die angegebenen Tests ersetzen nicht den abschließenden Test mit deinem Projekt.

Nach dem Einrichten bitte testen:

1. In Windows anmelden, zwei Aufgaben mit Abhängigkeit anlegen.
2. Mit demselben Konto auf dem iPhone anmelden; beide Aufgaben müssen erscheinen.
3. Voraussetzung erledigen; Folgeaufgabe auf dem anderen Gerät prüfen.
4. Einen kurzen deutschen Text über die KI erkennen lassen, Vorschau prüfen und übernehmen.
5. Export herunterladen und bei Bedarf im lokalen Testbereich importieren.
6. Passwort-Rücksetzlink und Home-Bildschirm-Verknüpfung prüfen.

Physisches iPhone/Safari, echte Auth-E-Mail-Zustellung, echte Realtime-Verbindung und die Modellverfügbarkeit in deinem Google-Konto wurden hier nicht getestet. Die CDN-Bibliotheken wurden in den isolierten Browser-Tests ersetzt; die Anwendung verwendet im Betrieb Supabase JS 2.49.1 und Font Awesome 6.7.2 von ihren CDNs.

## Bewusste Grenzen

- Persönliche Aufgabenverwaltung; keine geteilten Listen und keine Zuweisung an andere Nutzer.
- Keine Push-Erinnerungen, wiederkehrenden Aufgaben oder Board-Ansicht in diesem bestätigten Umfang.
- Online-Schreiben im Kontomodus; lokaler Lesecache, keine Offline-Warteschlange und kein garantierter Offline-Neustart.
- Keine automatische Zusammenführung gleichzeitiger Änderungen. Konflikte werden sichtbar, statt Änderungen zu verlieren.
- Backup-Import ergänzt Kopien; er ersetzt den bestehenden Stand nicht.
- KI-Ergebnisse bleiben Vorschläge und werden vor Speicherung bestätigt.

## Aktualisierung 1.1

Neue Grund- und Abschnittsfarben übernommen. Registrierungsbutton und JavaScript-signUp-Handler entfernt; administrativer Account-Workflow in der Anleitung beschrieben. Lokaler Testbereich bleibt erhalten. Browser-Prüfungen für Aufgabenlogik, Import, Persistenz, Dark Mode und mobile Breite erneut erfolgreich. Alle neun vorgegebenen Farben im CSS vorhanden. Serverseitige Registrierung muss der Projektinhaber in Supabase deaktivieren; diese Einstellung wurde nicht remote geändert.

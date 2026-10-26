# Fokus – deine To-do-App

Die erste Version für iPhone und Windows. Die gesamte Oberfläche steckt in **index.html**: HTML, CSS und Vanilla JavaScript. Deine Supabase-URL und dein öffentlicher Publishable Key sind bereits eingetragen. Keine geheimen Schlüssel sind enthalten.

## Enthaltene Dateien

- `index.html` – die App; diese Datei kommt auf GitHub Pages.
- `supabase-setup.sql` – Tabellen, Zugriffsschutz, Speichern mit Versionsprüfung, serverseitige Prüfung von Abhängigkeiten und KI-Tageslimit.
- `extract-todos.ts` – Supabase Edge Function für Gemini. Nicht als Browser-JavaScript einbinden.
- `config.toml` – optionale Konfiguration für die Supabase CLI; im Dashboard stattdessen die unten beschriebene Einstellung verwenden.
- `PRUEFUNG.md` – ausgeführte Tests und Grenzen.

Die Oberfläche ist eine einzige Datei. Datenbank und KI-Serverfunktion laufen separat bei Supabase. Es gibt keinen Build-Schritt und keine Node-Installation für die Einrichtung im Browser.

## 1. Direkt ausprobieren

`index.html` am Windows-Rechner öffnen und **„Erst einmal lokal ausprobieren“** wählen. Zwei Beispielaufgaben demonstrieren die Kaskade. Änderungen im Testbereich bleiben lokal, werden nicht mit dem Konto vermischt und lösen keine KI-Anfragen aus.

Für Anmeldung und Synchronisation die App über GitHub Pages öffnen. Lokale `file://`-Adressen eignen sich nicht für E-Mail-Rückleitungen. Testaufgaben lassen sich bei Bedarf per Backup exportieren und später im Konto importieren.

## 2. Datenbank vorbereiten

1. Dein Supabase-Projekt `lggtrgiokpuehlvozfdq` öffnen.
2. **SQL Editor → New query**.
3. Den kompletten Inhalt von `supabase-setup.sql` hineinkopieren.
4. **Run** drücken.

Das Skript verwendet eigene `fokus_…`-Tabellen und Funktionen. Es lässt andere Anwendungstabellen unverändert und kann erneut ausgeführt werden. Nicht einfach nur Tabellen manuell nachbauen: Zugriffsschutz und RPC-Funktionen gehören dazu.

Das Datenmodell speichert pro Konto ein zusammenhängendes JSON-Dokument in Postgres. Dadurch werden eine ganze Aufgabenfolge und ihre Beziehungen gemeinsam gespeichert. Eine Versionsnummer verhindert, dass zwei Geräte unbemerkt unterschiedliche Stände überschreiben. Ausgelegt für die persönliche Nutzung, bis 2.000 Aufgaben und 100 Projekte pro Konto. Keine gemeinsam bearbeiteten Listen in dieser Version.

## 3. App auf GitHub Pages veröffentlichen

1. In deinem Repository **Add file → Upload files** wählen.
2. `index.html` direkt in das Hauptverzeichnis hochladen und committen.
3. **Settings → Pages** öffnen.
4. Unter **Build and deployment** die Quelle **Deploy from a branch** wählen.
5. Branch `main`, Ordner `/(root)` auswählen und speichern.
6. Die dort angezeigte HTTPS-Adresse öffnen. Das kann nach dem Commit einige Minuten dauern.

Beispiel: `https://DEIN-NAME.github.io/todo/`. Die Platzhalter in diesem Beispiel durch deine tatsächliche Adresse ersetzen. Nur `index.html` muss öffentlich gehostet werden; SQL und Function-Code werden in Supabase eingerichtet.

## 4. Anmeldung einrichten

1. Supabase → **Authentication → URL Configuration**.
2. **Site URL** auf deine tatsächliche GitHub-Pages-Adresse mit abschließendem `/` setzen.
3. Unter **Redirect URLs** dieselbe Adresse ergänzen. Wenn du die App auch explizit über `/index.html` aufrufst, diese Variante ebenfalls ergänzen.
4. E-Mail/Passwort als Anmeldeverfahren verwenden.
5. Das Konto ausschließlich im Supabase-Dashboard unter **Authentication → Users → Add user** mit E-Mail und Passwort anlegen und bestätigen. Danach in der App anmelden. Die HTML-App enthält keine Registrierung und keinen `signUp`-Aufruf.

Weitere Konten ebenfalls direkt im Supabase-Dashboard anlegen und bestätigen. So bist du für den ersten Start nicht vom E-Mail-Versand abhängig. Für Passwort-zurücksetzen-E-Mails muss der Versand funktionieren; der Supabase-Standardversand ist eingeschränkt. Bei Zustellproblemen die Auth-E-Mail-/SMTP-Einstellungen prüfen.

**Allow new users to sign up** in Supabase deaktivieren. Das ist zusätzlich zur entfernten HTML-Registrierung notwendig, damit auch direkte öffentliche API-Aufrufe keine Konten anlegen können. Administrative Kontoanlage im Dashboard bleibt möglich. Bestehende Nutzer können sich weiterhin anmelden.

## 5. Gemini-Serverfunktion einrichten

1. Supabase → **Edge Functions → Secrets**:
   - `GEMINI_API_KEY` = dein geheimer Gemini-Schlüssel.
   - Optional `GEMINI_MODEL` = ein in deinem Google-AI-Studio-Projekt verfügbares kostenloses Textmodell. Vorgabe im Code: `gemini-3.1-flash-lite`. Verfügbarkeit und Kontingent im eigenen Konto prüfen; Modellangebote können sich ändern.
   - Für die persönliche App empfohlen: `ALLOWED_USER_IDS` = deine Nutzer-UUID aus **Authentication → Users**. Damit darf nur dein Konto die KI aufrufen. Mehrere UUIDs können durch Komma getrennt werden. Ohne diese Angabe sind alle angemeldeten Nutzer dieses Supabase-Projekts zugelassen.
2. **Edge Functions → Deploy a new function → Via Editor**.
3. Funktionsname exakt **`extract-todos`** setzen.
4. Den Beispielcode in der dortigen `index.ts` vollständig durch den Inhalt von `extract-todos.ts` ersetzen.
5. Funktion deployen.
6. In den Details der Funktion die vorgelagerte/Legacy-JWT-Prüfung (**Verify JWT**) deaktivieren. **Die Authentifizierung bleibt im gelieferten Handler aktiv:** Jeder Aufruf muss einen gültigen Nutzer-Token mitbringen, den der Handler beim Supabase-Auth-Server prüft. Ohne gültige Anmeldung wird Gemini nicht aufgerufen. Diese Einstellung ist auf genau diesen Handler abgestimmt.
7. Die KI in der angemeldeten App über **Aus Text erstellen** testen.

Falls du die CLI verwendest, gehört der Funktionscode nach `supabase/functions/extract-todos/index.ts`; den Abschnitt aus `config.toml` in `supabase/config.toml` übernehmen. Alternativ beim Deploy `--no-verify-jwt` verwenden. Die Einrichtung im Dashboard reicht aber vollständig aus.

**Der Gemini-Schlüssel gehört ausschließlich in Supabase Secrets**, nicht in `index.html`, GitHub oder ein Import-Backup. Die Funktion benötigt keinen `service_role`-Schlüssel. Sie greift mit deinem Nutzer-Token auf die Kontingentfunktion zu.

Im Google-Projekt den Free-Tarif verwenden und keine kostenpflichtige Abrechnung aktivieren, wenn du ausschließlich kostenlos starten möchtest. Die App selbst kann Googles Abrechnungsstatus nicht prüfen. Sie wechselt niemals automatisch zu einem anderen Anbieter oder Modell. Ein bezahltes Google-Projekt kann trotz unserer Begrenzung Kosten verursachen.

Die Serverfunktion erlaubt 40 Anfragen pro Nutzer und UTC-Tag. Auch ein fehlgeschlagener Gemini-Aufruf nach der Kontingentprüfung zählt dazu. Zusätzlich gelten die eigenen Gemini-Limits, die niedriger sein können. Ein Limitfehler blockiert nur die KI, nicht die normale Aufgabenverwaltung.

## 6. iPhone und Windows

**iPhone:** GitHub-Pages-Adresse in Safari öffnen → Teilen → Zum Home-Bildschirm. Über diese Verknüpfung startet die App im vorgesehenen Vollbildmodus. Im Home-Bildschirm-Modus gegebenenfalls erneut anmelden.

**Windows:** Dieselbe Adresse im Browser öffnen und mit demselben Konto anmelden. Ein Lesezeichen genügt. Strg+K fokussiert die Aufgabensuche.

Es gibt bewusst nur die eine Frontend-Datei. Deshalb ist kein Service Worker enthalten: Ein garantierter Offline-Kaltstart und eine vollständige installierbare Desktop-PWA mit Manifest gehören nicht zu dieser Version. Eine bereits geöffnete App kann ihren zuletzt geladenen Stand anzeigen. Änderungen im Kontomodus benötigen eine Verbindung; es gibt keine Offline-Warteschlange. Die App benötigt beim Laden außerdem die Supabase-Bibliothek vom CDN; Icons kommen von Font Awesome.

## Bedienung

- **Schnellerfassung:** Text eingeben, Plus drücken. Unter „Mein Tag“ werden neue Aufgaben automatisch für heute eingeplant. Die gerade gewählte Projektfilterung wird übernommen.
- **Details:** Auf den Aufgabentitel oder bei der Schnellerfassung auf `⋯` klicken.
- **Mein Tag:** Das Sonnensymbol wählt eine Aufgabe für das heutige Datum aus. Die Auswahl ist eine Tagesplanung, kein Fälligkeitsdatum; sie überträgt sich nicht automatisch auf morgen.
- **Jetzt machbar:** Offene Aufgaben mit erfüllten Abhängigkeiten, die nicht auf eine externe Rückmeldung warten. Eine noch offene Checkliste ist dabei gerade die verbleibende Arbeit.
- **Checklisten:** In den Details Schritte ergänzen und abhaken. Vor dem Abschluss müssen alle Checklistenpunkte erledigt sein.
- **Abhängigkeiten:** In den Details andere Aufgaben als Voraussetzungen auswählen. Alle müssen erledigt sein. Kreise werden verhindert.
- **Wieder öffnen:** Im Erledigt-Verlauf auf den Haken klicken. Bereits erledigte Folgeaufgaben bleiben erledigt; offene Voraussetzungen werden dort als Hinweis gezeigt.
- **Wartet auf:** Enthält sowohl explizit wartende als auch durch Voraussetzungen blockierte Aufgaben. Nachfassdaten sind sichtbar, lösen aber keine Push-Benachrichtigungen aus.
- **Projekte:** Unten links, am Handy neben der Aufgabenanzahl. Umbenennen mit Haken speichern. Beim Löschen eines Projekts bleiben seine Aufgaben erhalten.
- **Löschen:** Bestätigung erforderlich. Eine noch als Voraussetzung verwendete Aufgabe kann erst gelöscht werden, wenn ihre Abhängigkeiten in den Folgeaufgaben entfernt wurden.
- **KI-Vorschau:** Titel, Fälligkeit, Priorität, Projekt, Notizen und Voraussetzungen vor dem Übernehmen bearbeiten. Abhängigkeiten beziehen sich auf die sichtbaren Nummern, etwa `1, 2`. Entfernte Vorschläge werden aus den Beziehungen genommen; anschließend die Reihenfolge nochmals prüfen.
- **Synchronisation:** Echtzeit-Updates, zusätzlich beim Fokuswechsel und manuell in den Einstellungen. Bei einem Versionskonflikt bleiben die Eingaben im Editor erhalten, werden aber nicht gespeichert. Änderungen notieren/kopieren, synchronisieren, Editor neu öffnen und erneut anwenden. Kein automatisches Überschreiben.
- **Backup:** Einstellungen → Exportieren. Import ergänzt Kopien mit neuen IDs und erhält ihre Beziehungen. Gleichnamige Projekte werden zusammengeführt. Bestehende Aufgaben werden nicht überschrieben. Erneuter Import erstellt erneut Kopien. Historisch erledigte Aufgaben mit inzwischen offenen Voraussetzungen oder offenen Checklisten werden beim Import wieder geöffnet; die Bestätigung weist darauf hin, das ursprüngliche Abschlussdatum wird in den Notizen aufbewahrt.
- **Abmelden:** Entfernt den lokalen Aufgaben-Cache dieses Kontos. Der getrennte lokale Testbereich bleibt erhalten. Browserdaten nicht auf gemeinsam genutzten Geräten angemeldet lassen.

## Wenn etwas nicht klappt

- **„Nicht verbunden“ / Tabelle oder Funktion fehlt:** SQL-Skript vollständig ausführen und in der App „Jetzt synchronisieren“ drücken.
- **Bestätigungslink führt auf localhost:** Supabase Site URL und Redirect URLs korrigieren, neuen Link anfordern.
- **E-Mail kommt nicht an:** Spam, Auth-E-Mail-Limits und SMTP-Konfiguration prüfen. Für das persönliche Erstkonto ist die Anlage im Dashboard möglich.
- **Gemini HTTP 404:** `GEMINI_MODEL` auf ein im eigenen AI-Studio-Konto verfügbares Modell setzen.
- **Gemini HTTP 400/403:** Schlüssel, Modellzugang und Free-Tier-Berechtigung prüfen. Nicht automatisch Billing aktivieren.
- **401 bei der Function:** Erneut anmelden. Funktionsname und die oben beschriebene Legacy-JWT-Einstellung prüfen. Die explizite Auth-Prüfung im Code unbedingt beibehalten.
- **429:** Google-Kontingent oder das eigene Tageslimit erreicht; später erneut versuchen.
- **iPhone/Windows unterscheiden sich:** Prüfen, ob auf beiden Geräten dasselbe Konto und dieselbe Pages-Adresse geöffnet sind. „Jetzt synchronisieren“ ausführen. Lokale Testbereiche werden nicht synchronisiert.

## Quellen zur Einrichtung

- Gemini-Schlüssel: https://ai.google.dev/gemini-api/docs/api-key
- Gemini-Modelle/Preise: https://ai.google.dev/gemini-api/docs/pricing
- Strukturierte Gemini-Ausgabe: https://ai.google.dev/gemini-api/docs/generate-content/structured-output
- Supabase Edge Functions im Dashboard: https://supabase.com/docs/guides/functions/quickstart-dashboard
- Secrets: https://supabase.com/docs/guides/functions/secrets
- Auth-Rückleitungen: https://supabase.com/docs/guides/auth/redirect-urls
- GitHub Pages: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site

## Gestaltung in Version 1.1

Grundfarben: Weiß `#FFFFFF`, Blaugrau `#5D767B`, Schwarz `#000000` und Blau `#0069B0`. Abgeleitete transparente Tönungen dienen als Trennlinien und Flächen. Im Dunkelmodus werden die neutralen Flächen invertiert und Hilfstexte aufgehellt.

Abschnittsfarben: Mein Tag `#8D649D`, Jetzt machbar `#D7C741`, Alle Aufgaben `#A03B3B`, Wartet auf `#CE703E`, Erledigt `#639F5A`. Die Farben markieren Navigation und Überschriften, während normaler Text kontrastreich bleibt. Der lokale Testbereich ist weiterhin verfügbar.

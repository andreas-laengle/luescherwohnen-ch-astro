# SEO-Vorschlag für Lüscher Wohnen AG

## Kurzfazit

Die Astro-Version hat eine gute inhaltliche Basis: Sie nennt Beratung, Einrichtungsplanung, Möbel und Lieferung/Montage. Ihre technischen SEO-Grundlagen sind aber noch nicht fertig. Besonders wichtig: Das Astro-Layout enthält noch englische Standard-Metadaten, während die öffentlich erreichbare `www`-Website bereits eigene deutsche Metadaten und strukturierte Firmendaten ausliefert. Die beiden Versionen scheinen außerdem unterschiedliche Produkt-URLs und Inhalte zu verwenden.

**Empfehlung:** Zuerst die Umstellung von der aktuellen öffentlichen Website auf Astro sauber planen; danach Seiteninhalte und lokale Sichtbarkeit gezielt ausbauen.

## Befunde

- In der Astro-Version stehen `lang="en"`, der Titel „Bootstrap with Astro“ und eine generische englische Description im [Layout](./src/layouts/Layout.astro). Es fehlen dort außerdem seitenbezogene Canonical- und Social-Metadaten.
- Die Startseite hat derzeit eine allgemeine H1 („Willkommen bei Lüscher Wohnen AG“). Angebot und Standort sind zwar im Text vorhanden, aber noch nicht als präzise Suchergebnisseinstiege formuliert: [Startseite](./src/pages/index.astro).
- Astro veröffentlicht derzeit eine Produktseite unter `/products`; die öffentliche `www`-Website liefert dagegen `/products.html`, und `/products` wurde mit 404 beantwortet. Als einheitliche neue Zielroute wird `/shop` empfohlen. Auch `robots.txt` und `sitemap.xml` waren öffentlich nicht erreichbar.
- Die Astro-Produktseite enthält teils generische Beschreibungen und Angaben, die vor Veröffentlichung geprüft werden sollten, darunter Preise und Zahlungslinks: [Produktseite](./src/pages/products.astro).
- **Positiv:** Die öffentliche `www`-Startseite hat bereits einen passenden deutschen Seitentitel, Description, Canonical-URL, Open-Graph-Daten und `FurnitureStore`-Daten. Diese Grundlagen sollten beim Wechsel erhalten und für Astro konsistent umgesetzt werden. Das Astro-Projekt hat derzeit keine entsprechende SEO-Konfiguration in der [Astro-Konfiguration](./astro.config.mjs).

## Maßnahmen nach Priorität

### 1. Website-Wechsel und Indexierung absichern — höchste Priorität

Diese Massnahmen sollten vor oder unmittelbar beim Wechsel der öffentlichen Website auf Astro umgesetzt werden.

#### 1. Kanonische Domain festlegen

Die bisherige öffentliche Website verwendet `https://www.luescherwohnen.ch/` bereits als Canonical-Adresse. Deshalb empfiehlt sich, `www.luescherwohnen.ch` als Hauptdomain beizubehalten. Alle anderen Varianten sollten dauerhaft und direkt dorthin weiterleiten:

- `http://luescherwohnen.ch/*` → `https://www.luescherwohnen.ch/*`
- `https://luescherwohnen.ch/*` → `https://www.luescherwohnen.ch/*`
- `http://www.luescherwohnen.ch/*` → `https://www.luescherwohnen.ch/*`

Die Weiterleitung sollte den Pfad und allfällige URL-Parameter erhalten. So wird beispielsweise auch `https://luescherwohnen.ch/products.html` direkt auf die neue Shop-Seite unter der Hauptdomain geleitet.

#### Technische Umsetzung der Hauptdomain

Die nachfolgenden Schritte sind bewusst anbieterneutral. DNS-Zielwerte und Redirect-Einstellungen unterscheiden sich je nach Hosting-Anbieter; konkrete Werte dürfen erst nach Identifikation des Hostings eingesetzt werden.

Eine Weiterleitung wird nicht durch einen DNS-Eintrag allein eingerichtet. DNS sorgt dafür, dass ein Hostname beim richtigen Hosting oder CDN ankommt; die eigentliche HTTP-Weiterleitung wird dort konfiguriert.

1. **Zuständigkeiten und DNS-Ziele feststellen.** Prüfen, bei welchem Anbieter die DNS-Zone für `luescherwohnen.ch` verwaltet wird und wo die Astro-Website gehostet wird. Das kann derselbe Anbieter sein, muss es aber nicht. Im Hosting-Dashboard die Astro-Website mit beiden Hostnamen verbinden: `luescherwohnen.ch` (Apex-Domain) und `www.luescherwohnen.ch`. Der Anbieter gibt vor, welche DNS-Einträge zu verwenden sind. Häufig zeigt `www` per CNAME auf den vom Anbieter genannten Hostnamen; für die Apex-Domain werden je nach Anbieter A-, AAAA-, ALIAS- oder ANAME-Einträge verwendet. Keine Beispiel-IP-Adresse übernehmen: Nur die Werte des tatsächlichen Hosting-Anbieters eintragen. Vorhandene Mail-Einträge wie MX, SPF, DKIM und DMARC dabei nicht verändern.

2. **Beide Hostnamen dem Hosting zuordnen.** Auch wenn die Apex-Domain nur weiterleitet, muss sie in der Regel dem Hosting/CDN als Domain hinzugefügt sein, damit dieses die Anfrage annimmt und weiterleitet. Bei einem Anbieter mit „Primary domain“-, „Domain redirect“- oder ähnlicher Einstellung `www.luescherwohnen.ch` als primäre Domain auswählen und die Weiterleitung der Apex-Domain auf die primäre Domain aktivieren. Zusätzlich HTTP auf HTTPS umleiten lassen.

3. **TLS/HTTPS für beide Namen aktivieren.** Ein gültiges TLS-Zertifikat muss `www.luescherwohnen.ch` und `luescherwohnen.ch` abdecken. Das gilt auch für die Apex-Domain, die unmittelbar auf www weiterleitet: Der Browser baut zuerst eine HTTPS-Verbindung zur aufgerufenen Domain auf und akzeptiert kein Zertifikatsproblem nur deshalb, weil danach eine Weiterleitung folgt. Viele verwaltete Hostings stellen Zertifikate automatisch aus, sobald beide Domains verbunden und korrekt aufgelöst sind. Erst HTTPS aktivieren bzw. erzwingen, wenn beide Hostnamen ein gültiges Zertifikat erhalten haben.

4. **Weiterleitung am Hosting oder Webserver konfigurieren.** Die Weiterleitung für die Nicht-Hauptdomain soll den Pfad und die Query-Parameter unverändert übernehmen. Zum Beispiel muss `/produkte?farbe=blau` auf `https://www.luescherwohnen.ch/produkte?farbe=blau` zeigen. Die Weiterleitung sollte in einem einzigen Schritt direkt auf HTTPS und www führen.

   Bei einem eigenen Nginx-Server kann das Prinzip beispielsweise so aussehen; Zertifikatspfade und der eigentliche `www`-Site-Block müssen an die Serverumgebung angepasst werden:

   ```nginx
   server {
       listen 80;
       listen [::]:80;
       server_name luescherwohnen.ch www.luescherwohnen.ch;
       return 301 https://www.luescherwohnen.ch$request_uri;
   }

   server {
       listen 443 ssl;
       listen [::]:443 ssl;
       server_name luescherwohnen.ch;

       ssl_certificate     /path/to/certificate-covering-apex-and-www/fullchain.pem;
       ssl_certificate_key /path/to/certificate-covering-apex-and-www/privkey.pem;

       return 301 https://www.luescherwohnen.ch$request_uri;
   }
   ```

   Der produktive HTTPS-Site-Block für `www.luescherwohnen.ch` muss die Website ausliefern und darf nicht ebenfalls auf die Apex-Domain zurückleiten. Bei verwaltetem Hosting diese Regeln nicht zusätzlich in Nginx-Dateien eintragen, sondern die Domain- und Redirect-Einstellungen des Anbieters verwenden. Astro selbst kann die Umleitung nicht für alle Hostnamen zuverlässig übernehmen, weil für eine HTTP-Weiterleitung zuerst die Anfrage beim Hosting ankommen muss.

5. **Kanonische URLs in der Website angleichen.** Jede indexierbare Seite sollte einen absoluten Canonical-Link auf ihre endgültige www-HTTPS-Adresse enthalten, beispielsweise:

   ```html
   <link rel="canonical" href="https://www.luescherwohnen.ch/shop" />
   ```

   Ebenso sollten interne Navigation, Open-Graph-URLs und Sitemap-Einträge ausschliesslich die endgültigen `https://www.luescherwohnen.ch/...`-Adressen verwenden. Canonical-Tags sind kein Ersatz für Weiterleitungen; beides muss konsistent sein.

6. **Änderung kontrolliert veröffentlichen.** Bestehende DNS-Werte und Hosting-Einstellungen vor der Änderung dokumentieren. Dann die Domain am Hosting verbinden, Zertifikate abwarten und die Weiterleitungsregeln aktivieren. DNS-Änderungen können je nach TTL und Resolver unterschiedlich schnell sichtbar werden. Die Website sollte während der Umstellung erreichbar bleiben; deshalb keine alten DNS-Einträge löschen, bevor die neuen Ziele funktionieren.

7. **Alle vier Varianten testen.** Nach Veröffentlichung für Startseite und mindestens eine Unterseite jeweils HTTP/HTTPS und www/apex aufrufen. Unter Windows zeigen diese Befehle die erste HTTP-Antwort samt `Location`-Header:

   ```powershell
   curl.exe -sS -D - -o NUL http://luescherwohnen.ch/testpfad?x=1
   curl.exe -sS -D - -o NUL https://luescherwohnen.ch/testpfad?x=1
   curl.exe -sS -D - -o NUL http://www.luescherwohnen.ch/testpfad?x=1
   curl.exe -sS -D - -o NUL https://www.luescherwohnen.ch/testpfad?x=1
   ```

   Die ersten drei Varianten sollten mit `301` und `Location: https://www.luescherwohnen.ch/testpfad?x=1` antworten. Die letzte Variante sollte die Seite direkt ausliefern (normalerweise `200 OK`). `testpfad` muss für den Test eine vorhandene Route sein. Zusätzlich die Zertifikate beider Hostnamen im Browser prüfen und mit `curl.exe -IL` den gesamten Weiterleitungsverlauf kontrollieren; es darf keine Schleife oder unnötige Weiterleitungskette geben.

**Für eine exakte, anbieterbezogene Anleitung** müssen vor der Änderung der DNS-Zone bzw. des Hosting-Dashboards folgende Angaben vorliegen: DNS-Verwalter, Hosting-Anbieter der Astro-Website und ob dort ein verwalteter Domain-Redirect oder eigener Nginx-/Apache-Server verwendet wird. Danach die aktuellen DNS-Einträge sichern, die vom Hosting vorgegebenen Zielwerte eintragen und die Weiterleitung samt TLS-Zertifikat gemäss dessen Oberfläche oder Webserver-Konfiguration einrichten. Ohne diese Angaben bleiben Beispielwerte absichtlich generisch.

#### 2. Alte URLs auf passende Astro-Routen weiterleiten

Vor dem Wechsel eine Zuordnung aller aktuell veröffentlichten URLs zu ihren neuen Zielen erstellen. Für die bekannten Seiten ist folgende Zuordnung ein sinnvoller Ausgangspunkt:

| Bisherige URL | Neue URL |
|---|---|
| `/index.html` | `/` |
| `/products.html` | `/shop` |
| `/impressum.html` | `/impressum` |
| `/agb.html` | `/agb` |

Zusätzlich sollte die derzeitige Astro-Route `/products` direkt auf `/shop` weiterleiten, damit bestehende Links auf die Astro-Version gültig bleiben. Jede Weiterleitung sollte als permanenter **HTTP-Status 301** auf Server- oder Hosting-Ebene eingerichtet werden. Weiterleitungsziele müssen passende, tatsächlich vorhandene Seiten sein; mehrere Weiterleitungsschritte (Ketten) sind zu vermeiden. Alle internen Links, Canonical-URLs und Sitemap-Einträge sollten gleich auf die neuen URLs zeigen.

Die konkrete Konfiguration hängt vom Hosting-Anbieter und dessen Webserver ab. Da Astro hier statische Dateien in `dist/` erzeugt, reicht eine Astro-Seite allein nicht für zuverlässige Weiterleitungen alter URLs. Vor dem Go-live muss daher geklärt werden, wo die Domain und Weiterleitungen konfiguriert werden.

#### 3. Sitemap und robots.txt bereitstellen

Für die Astro-Website sollten nach dem Build diese Dateien unter der Hauptdomain öffentlich erreichbar sein:

- `https://www.luescherwohnen.ch/sitemap.xml` mit den indexierbaren, kanonischen Zielseiten (nicht mit alten URLs oder Weiterleitungen).
- `https://www.luescherwohnen.ch/robots.txt` mit einer passenden Crawl-Regel und einem Verweis auf `https://www.luescherwohnen.ch/sitemap.xml`.

Die Sitemap sollte nach jedem Produktions-Build aktualisiert werden. Die `robots.txt` darf wichtige Seiten oder Ressourcen nicht versehentlich vom Crawling ausschliessen.

#### 4. Vor und nach dem Go-live prüfen

Für jede alte URL und Domainvariante prüfen:

1. Die alte Adresse liefert genau eine permanente Weiterleitung (`301`) auf die passende neue URL.
2. Die Zielseite liefert `200 OK` und zeigt den erwarteten Inhalt.
3. Es gibt keine Weiterleitungsschleifen, Ketten oder unerwarteten 404-Seiten.
4. Canonical-Links zeigen auf `https://www.luescherwohnen.ch/...`.
5. Interne Navigation und Sitemap verlinken direkt auf die neuen Ziel-URLs.
6. `robots.txt` und `sitemap.xml` sind öffentlich erreichbar und enthalten keine falschen oder nicht-kanonischen URLs.

Nach dem Go-live die Sitemap in der Google Search Console einreichen und dort Indexierungsfehler sowie die Erfassung der wichtigsten Seiten beobachten. Die tatsächliche Einrichtung der HTTP-Weiterleitungen muss anhand des eingesetzten Hostings erfolgen; dessen Anbieter ist vor der Umsetzung noch zu bestätigen.

### 2. Deutsche, individuelle Metadaten je Seite

Das Layout sollte Titel, Description und Canonical-URL pro Route entgegennehmen, statt für alle Seiten dieselben Standardwerte auszugeben. Zusätzlich `lang="de-CH"` setzen sowie passende Open-Graph-Daten für geteilte Links pflegen.

**Beispiel für die Startseite:**

- **Title:** `Möbel & Wohnberatung in Baden | Lüscher Wohnen AG`
- **Description:** `Persönliche Wohnberatung, Möbel und Einrichtungsplanung in Baden. Lüscher Wohnen AG liefert und montiert Möbel, Betten, Vorhänge und Bodenbeläge.`

Als Ausgangspunkt für die Keyword-Recherche bieten sich Suchanfragen wie „Möbelhaus Baden“, „Wohnberatung Baden“, „Einrichtungshaus Baden“ und „Möbellieferung Baden“ an. Das sind zunächst **Themenhypothesen**; Suchvolumen und tatsächliche Nachfrage sollten mit Search Console und Keyword-Tools geprüft werden.

### 3. Angebotsseiten statt nur einer langen Startseite

Die Startseite eignet sich als lokale Übersichtsseite. Für wichtige Leistungen sollte es eigene, inhaltlich eigenständige Seiten geben, etwa:

- Wohnberatung und Einrichtungsplanung
- Möbel und Polstermöbel
- Betten und Bettwaren
- Vorhänge und Bodenbeläge
- Lieferung und Montage

Die vorhandenen Texte zu Beratung, Planung und Montage auf der [Startseite](./src/pages/index.astro) liefern dafür erste Inhalte. Jede neue Seite sollte konkrete Leistungen, Ablauf, passende Fotos, Standortbezug und eine klare Kontaktmöglichkeit enthalten. Ortsseiten nur dort ergänzen, wo Lüscher Wohnen tatsächlich tätig ist und dafür jeweils hilfreiche, einzigartige Inhalte möglich sind.

### 4. Lokale Auffindbarkeit stärken

Die bereits auf der öffentlichen Website vorhandenen `FurnitureStore`-Daten in die Astro-Version übernehmen und auf Öffnungszeiten, Telefonnummer, Adresse und Domain abstimmen. Zusätzlich das Google-Unternehmensprofil aktuell halten: Kategorien, Öffnungszeiten, Leistungen, Fotos und Antworten auf Bewertungen. Name, Adresse und Telefonnummer sollten auf Website und relevanten Branchenverzeichnissen übereinstimmen.

### 5. Produktdaten und Vertrauenswürdigkeit korrigieren

Vor dem Launch Produktpreise, Beschreibungstexte und Zahlungslinks einzeln mit dem tatsächlichen Angebot abgleichen. Unpassende Standardformulierungen ersetzen und Produktdetails klar nennen. Strukturierte Produktdaten nur dann verwenden, wenn Preis und Kaufmöglichkeit tatsächlich stimmen; andernfalls sollte die Seite als Produktübersicht mit Kontakt-CTA gestaltet werden.

Auch zeitgebundene Hinweise wie Betriebsferien zeitnah entfernen oder aktualisieren. Das betrifft aktuell den Hinweis im Bereich „Aktuelles“ der [Startseite](./src/pages/index.astro).

### 6. Bilder, Nutzererlebnis und Erfolgsmessung

Aussagekräftige Alt-Texte verwenden, die das jeweilige Möbelstück oder die Raumsituation beschreiben, statt nur Namen oder Marken zu nennen. Astro optimiert Bilder bereits; das Hero-Bild sollte zusätzlich auf schnelles Laden im sichtbaren Seitenbereich geprüft werden. Für den eingebetteten Kartenbereich ist Lazy Loading bereits vorhanden.

Vor dem Launch Google Search Console einrichten und danach Indexierung, Suchanfragen, Klicks und Kontaktaktionen über etwa 90 Tage beobachten. Ergänzend mobile Bedienbarkeit und Core Web Vitals messen. Ohne Zugriff auf Search Console oder Performance-Daten lässt sich der aktuelle organische Traffic nicht belastbar beurteilen.

## Sinnvolle Reihenfolge

1. **Vor dem Go-live:** Domain festlegen, Redirects und URL-Migration planen, Sitemap und `robots.txt` bereitstellen.
2. **Beim Astro-Launch:** deutsche Seitensprache, individuelle Metadaten und lokale Firmendaten vervollständigen.
3. **Danach:** wichtigste Leistungsseiten ausbauen, Produktinformationen verifizieren und Google-Unternehmensprofil pflegen.
4. **Nach 1–3 Monaten:** Search-Console-Daten auswerten und Inhalte nach realen Suchanfragen priorisieren.

**Wichtigster Punkt:** Die derzeitige öffentliche Website und die vorhandene Astro-Version sollten vor SEO-Arbeiten technisch und inhaltlich auf einen einheitlichen Stand gebracht werden. Sonst drohen falsche oder veraltete Suchergebnisse, 404-Fehler und verlorene Sichtbarkeit bei der Umstellung.

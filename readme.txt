=== Card Forms Webhook & AI ===
Contributors:      wpmfs
Tags:              forms, webhook, ai, n8n, gutenberg
Requires at least: 6.5
Tested up to:      7.0
Requires PHP:      7.4
Stable tag:        1.1.1
License:           GPLv2 or later
License URI:       https://www.gnu.org/licenses/gpl-2.0.html

Schickt Card-Forms-Einsendungen signiert an einen Webhook, etwa an einen n8n-Workflow. Benötigt Card Forms 1.3.0 oder neuer, für den Webhook-Schalter im Formular-Editor 1.3.1.

== Description ==

Erweitert **Card Forms** um den Versand jeder Einsendung an eine Webhook-URL.

* **Je Formular einstellbar** — im Formular unter „Webhook“ einschalten und die Ziel-URL eintragen.
* **JSON per POST** — Formular-ID und -Titel, Einsendungs-ID, Zeitpunkt, alle Antworten mit Feld-Label (bei Auswahlfeldern auch mit dem Label der gewählten Option).
* **Signiert** — jede Anfrage trägt eine HMAC-SHA256-Signatur, damit der Empfänger prüfen kann, dass sie von dieser Website stammt.
* **Im Hintergrund, mit Wiederholung** — der Besucher sieht sofort die Dankesmeldung. Der Versand läuft über WP-Cron; schlägt er fehl, folgen bis zu drei weitere Versuche nach 1 Minute, 10 Minuten und 1 Stunde.
* **Zustellprotokoll** — unter Card Forms → Webhook stehen die letzten Zustellungen mit Status und HTTP-Antwort; fehlgeschlagene lassen sich neu anstoßen.

Nicht enthalten, auch wenn der Plugin-Name es nahelegt: Ergebnisseiten aus der Webhook-Antwort, KI-Auswertung in WordPress und E-Mail-Versand. Die Antwort des Empfängers wird nicht ausgewertet, nur ihr HTTP-Status.

Das Basis-Plugin `mm-wpmfs-card-forms` muss installiert und aktiv sein. Ab Card Forms 1.3.1 blendet der Formular-Editor den Webhook-Schalter nur ein, wenn dieses Addon aktiv ist; mit Card Forms 1.3.0 sendet das Addon ebenfalls, der Schalter erscheint dort aber unabhängig vom Addon.

= Anfrage =

Kopfzeilen:

* `Content-Type: application/json; charset=utf-8`
* `X-WPMFS-Event: submission.created`
* `X-WPMFS-Delivery` — ID der Zustellung. Bleibt bei Wiederholungen gleich; der Empfänger kann damit Doppelte erkennen.
* `X-WPMFS-Timestamp` — Unix-Zeit des Versuchs
* `X-WPMFS-Signature` — `sha256=` gefolgt von HMAC-SHA256 über `<X-WPMFS-Timestamp>.<Rumpf>` mit dem Secret

Rumpf (gekürzt):

    {
      "event": "submission.created",
      "deliveryId": 37,
      "site": "https://example.org/",
      "formId": 23,
      "formTitle": "Mietgesuch",
      "submissionId": 9499,
      "submittedAt": "2026-10-30T21:07:05+01:00",
      "fields": [
        { "key": "q-zimmer", "label": "Wie viele Zimmer?", "type": "single_choice", "value": "drei", "valueLabel": "3 Zimmer" },
        { "key": "email", "label": "E-Mail", "type": "email", "value": "anna@example.org" }
      ],
      "answers": { "q-zimmer": "drei", "email": "anna@example.org" }
    }

Als zugestellt gilt jede 2xx-Antwort. Weiterleitungen werden nicht verfolgt. 4xx-Antworten (außer 408, 425, 429) werden nicht wiederholt.

= Sicherheit =

Gesendet wird nur an `https`-Adressen, die auf eine öffentliche IP zeigen, und nur auf die Ports, die WordPress für sichere Anfragen zulässt (80, 443, 8080). Adressen im internen Netz des Servers, Loopback und Cloud-Metadaten-Dienste werden abgelehnt; die Zustellung steht dann als „Failed“ im Protokoll. Wer bewusst an einen internen Host senden muss, gibt ihn über den WordPress-Filter `http_request_host_is_external` frei, einen weiteren Port über `http_allowed_safe_ports`.

Das Secret wird bei der Aktivierung erzeugt und steht unter Card Forms → Webhook. Alternativ in der `wp-config.php` setzen, dann hat es Vorrang:

    define( 'MM_WPMFS_CARD_FORMS_WEBHOOK_SECRET', '...' );

= Datenschutz =

Einsendungen verlassen die Website, sobald in einem Formular ein Webhook eingeschaltet ist. Welche Daten das sind, bestimmt das jeweilige Formular. Der Zielendpunkt ist frei wählbar und wird vom Plugin nicht vorgegeben — die datenschutzrechtliche Einordnung (etwa Auftragsverarbeitung) liegt beim Betreiber.

Das Zustellprotokoll enthält keine Antworten, nur IDs, Ziel-URL, Status und HTTP-Code; es wird nach 30 Tagen gelöscht. Die Antworten liest der Versand erst im Moment des Sendens aus der Einsendung. Wurde sie bis dahin gelöscht (Datenschutz-Löschung, Aufbewahrungsfrist), wird nichts mehr gesendet.

= Lizenz und Nutzungsbedingungen =

Der Code steht unter GPLv2 or later. Kommerziell sind **Updates und Support**, nicht der Code: Die Nutzung setzt eine Vereinbarung mit Motion Media voraus. Anfragen über https://wpmfs.de.

== Installation ==

1. Zuerst **Card Forms** (ab 1.3.0, für den Schalter im Formular-Editor ab 1.3.1) installieren und aktivieren.
2. Dieses ZIP über Plugins → Installieren → Plugin hochladen einspielen und aktivieren.
3. Im Formular (Card Forms → Formular bearbeiten) unter „Webhook“ den Versand einschalten und die Ziel-URL eintragen.
4. Das Secret unter Card Forms → Webhook beim Empfänger hinterlegen.

Updates kommen automatisch über den Plugins-Bildschirm.

== Frequently Asked Questions ==

= Warum lässt sich das Plugin nicht aktivieren? =

Es setzt das kostenlose **Card Forms** voraus. WordPress blockiert die Aktivierung, solange das Basis-Plugin fehlt oder inaktiv ist.

= Die Zustellung bleibt auf „Queued“ stehen. =

Der Versand läuft über WP-Cron. Das stößt das Plugin direkt nach der Einsendung über eine Anfrage an die eigene Website an. Blockiert der Server solche Anfragen an sich selbst, oder ist `DISABLE_WP_CRON` gesetzt, braucht die Website einen echten Cronjob, der `wp-cron.php` regelmäßig aufruft. Zustellungen, die länger als zwei Stunden offen sind, markiert das Plugin als „Failed“; sie lassen sich dann im Protokoll neu anstoßen.

= Mein n8n läuft auf Port 5678. =

Diesen Port lässt WordPress für sichere Anfragen nicht zu. n8n hinter einen Reverse-Proxy auf 443 stellen (empfohlen) oder den Port über den Filter `http_allowed_safe_ports` freigeben.

= Wie prüfe ich die Signatur in n8n? =

Im Code-Node HMAC-SHA256 über `Timestamp + "." + Rohrumpf` mit dem Secret berechnen, als Hex kodieren, `sha256=` voranstellen und mit `X-WPMFS-Signature` vergleichen. Wichtig ist der unveränderte Rohrumpf — im Webhook-Node „Raw Body“ einschalten. Anfragen mit einem Zeitstempel, der mehr als einige Minuten alt ist, sollte der Empfänger verwerfen.

== Changelog ==

= 1.1.1 =
* Deutsche Übersetzung: Die Seite Card Forms → Webhook und alle Hinweise des Plugins erscheinen auf deutschen Websites jetzt auf Deutsch. Bisher fehlten die Sprachdateien im Paket.

= 1.1.0 =
* Der Webhook-Versand funktioniert jetzt. Bis 1.0.3 war er nur angelegt: Einsendungen wurden nie gesendet, auch wenn im Formular ein Webhook eingetragen war.
* Versand als signiertes JSON (HMAC-SHA256) mit Feld-Labels, im Hintergrund über WP-Cron, mit bis zu drei Wiederholungen.
* Meldet Card Forms ab 1.3.1, dass Webhooks zugestellt werden; erst dann zeigt der Formular-Editor den Webhook-Schalter.
* Neue Seite Card Forms → Webhook: Secret anzeigen und neu erzeugen, letzte Zustellungen mit Status, fehlgeschlagene neu anstoßen.
* Gesendet wird nur an öffentliche https-Adressen; interne Adressen des Servers werden abgelehnt.
* Die Tabellen werden nur noch bei Aktivierung und nach einem Update angelegt, nicht mehr bei jedem Seitenaufruf.
* Die bisher beschriebene Ergebnisseite war nie umgesetzt und ist aus der Beschreibung entfernt.

= 1.0.3 =
* Das Plugin startet jetzt auch dann, wenn alle Plugins aktiv sind. Vorher lud
  es vor dem Basis-Plugin, meldete fälschlich „benötigt das Basis-Plugin“ und
  blieb wirkungslos.

= 1.0.2 =
* Plugin-Symbol und -Banner werden jetzt an WordPress gemeldet und erscheinen in der Plugin-Liste und im „Details ansehen"-Dialog. Vorläufige Grafiken, bis endgültiges Material vorliegt.
* Der geteilte Update-Code trägt eine neue Versionsnummer. Ohne sie hätte die älteste installierte Kopie das Verhalten aller Plugins der Familie bestimmt.

= 1.0.1 =
* Beim Löschen des Plugins wird die Ergebnistabelle entfernt. Formulare und Einsendungen gehören dem Basis-Plugin und bleiben unangetastet.

= 1.0.0 =
* Erste Fassung: Webhook je Einsendung, gespeicherte Ergebnisse, Ergebnisseite.
* Umbenannt auf das Präfix der WPMFS-Familie (vormals `card-forms-webhook-ai`).
* Lizenz-Header von Proprietary auf GPLv2 or later; kommerziell sind Updates und Support.

== Upgrade Notice ==

= 1.1.0 =
Webhooks werden ab jetzt tatsächlich gesendet. Vor dem Update prüfen, welche Formulare einen Webhook eingetragen haben — deren Einsendungen gehen ab dann an die eingetragene Adresse. Benötigt Card Forms 1.3.0, für den Webhook-Schalter im Formular-Editor 1.3.1.

= 1.0.2 =
Bringt Symbole und Banner mit. Damit sie überall greifen, sollten alle Plugins der Familie aktualisiert werden.

= 1.0.1 =
Beim **Löschen** des Addons werden gespeicherte Webhook-Ergebnisse mit entfernt.

= 1.0.0 =
Erste Fassung unter dem neuen Plugin-Slug `mm-wpmfs-card-forms-webhook-ai`. Die REST-Route heißt jetzt `mm-wpmfs-card-forms/v1`; bestehende n8n-Workflows müssen angepasst werden.

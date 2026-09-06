=== Card Forms Webhook & AI ===
Contributors:      wpmfs
Tags:              forms, webhook, ai, n8n, gutenberg
Requires at least: 6.5
Tested up to:      7.0
Requires PHP:      7.4
Stable tag:        1.0.1
License:           GPLv2 or later
License URI:       https://www.gnu.org/licenses/gpl-2.0.html

Schickt Card-Forms-Einsendungen an einen Webhook und zeigt die Antwort als Ergebnisseite. Benötigt Card Forms.

== Description ==

Erweitert **Card Forms** um zwei Dinge:

* **Webhook** — jede Einsendung wird an eine konfigurierbare URL geschickt, etwa an einen n8n-Workflow.
* **Ergebnisseite** — die Antwort des Webhooks wird gespeichert und dem Absender angezeigt. Damit lassen sich ausgewertete Ergebnisse zurückspielen, ohne dass die Auswertung in WordPress stattfindet.

Das Basis-Plugin `mm-wpmfs-card-forms` muss installiert und aktiv sein.

= Datenschutz =

Einsendungen verlassen die Website, sobald ein Webhook eingetragen ist. Welche Daten das sind, bestimmt das jeweilige Formular. Der Zielendpunkt ist frei wählbar und wird vom Plugin nicht vorgegeben — die datenschutzrechtliche Einordnung liegt beim Betreiber.

= Lizenz und Nutzungsbedingungen =

Der Code steht unter GPLv2 or later. Kommerziell sind **Updates und Support**, nicht der Code: Die Nutzung setzt eine Vereinbarung mit Motion Media voraus. Anfragen über https://wpmfs.de.

== Installation ==

1. Zuerst **Card Forms** installieren und aktivieren.
2. Dieses ZIP über Plugins → Installieren → Plugin hochladen einspielen und aktivieren.
3. Im Card-Form-Block den Webhook aktivieren und die Ziel-URL eintragen.

Updates kommen automatisch über den Plugins-Bildschirm.

== Frequently Asked Questions ==

= Warum lässt sich das Plugin nicht aktivieren? =

Es setzt das kostenlose **Card Forms** voraus. WordPress blockiert die Aktivierung, solange das Basis-Plugin fehlt oder inaktiv ist.

= Wie kommt die Antwort zum Absender zurück? =

Das Plugin legt die Webhook-Antwort in einer eigenen Tabelle ab und liefert sie über die REST-Route `mm-wpmfs-card-forms/v1/result/<session_id>` aus.

== Changelog ==

= 1.0.1 =
* Beim Löschen des Plugins wird die Ergebnistabelle entfernt. Formulare und Einsendungen gehören dem Basis-Plugin und bleiben unangetastet.

= 1.0.0 =
* Erste Fassung: Webhook je Einsendung, gespeicherte Ergebnisse, Ergebnisseite.
* Umbenannt auf das Präfix der WPMFS-Familie (vormals `card-forms-webhook-ai`).
* Lizenz-Header von Proprietary auf GPLv2 or later; kommerziell sind Updates und Support.

== Upgrade Notice ==

= 1.0.1 =
Beim **Löschen** des Addons werden gespeicherte Webhook-Ergebnisse mit entfernt.

= 1.0.0 =
Erste Fassung unter dem neuen Plugin-Slug `mm-wpmfs-card-forms-webhook-ai`. Die REST-Route heißt jetzt `mm-wpmfs-card-forms/v1`; bestehende n8n-Workflows müssen angepasst werden.

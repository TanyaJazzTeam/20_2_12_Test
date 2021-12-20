---
"$title": Verwenden Sie AMP als Datenquelle für PWA
"$order": eins
description: Wenn Sie in AMP investiert, aber noch keine Progressive Web App erstellt haben, können Ihre AMP-Seiten die Entwicklung Ihrer Progressive Web App erheblich vereinfachen.
formats:
  - Websites
author: pbackaus
---

Wenn Sie in AMP investiert, aber noch keine Progressive Web App erstellt haben, können Ihre AMP-Seiten die Entwicklung Ihrer Progressive Web App erheblich vereinfachen. In diesem Tutorial erfahren Sie, wie Sie AMP in Ihrer Progressive Web App verwenden und vorhandene AMP-Seiten als Datenquelle verwenden.

## JSON zu AMP

Im gängigsten Szenario ist eine progressive Webanwendung eine Einzelseitenanwendung, die über Ajax eine Verbindung zu einer JSON-API herstellt. Diese JSON-API gibt dann Datensätze zur Navigationssteuerung und tatsächlichen Inhalt zum Anzeigen von Artikeln zurück.

Teststrecke 1

Anschließend müssen Sie den Rohinhalt in verwendbares HTML umwandeln und auf dem Client anzeigen. Dieses Verfahren ist teuer und oft schwer zu warten. Stattdessen können Sie vorhandene AMP-Seiten als Inhaltsquelle wiederverwenden. Das Beste ist, dass Sie dies mit AMP mit nur wenigen Codezeilen tun können.

Teststrecke 2

## Integrieren Sie Shadow AMP in Ihre Progressive Web App

Der erste Schritt besteht darin, eine spezielle Version von AMP, die wir "Shadow AMP" nennen, in Ihre Progressive Web App einzubinden. Ja, das ist richtig. Sie laden die AMP-Bibliothek in die Seite der obersten Ebene, aber sie steuert nicht den Inhalt der obersten Ebene. Er wird nur die Teile unserer Seite "erweitern", von denen Sie ihm erzählen.

Fügen Sie Shadow AMP in den Titel Ihrer Seite ein, zum Beispiel:

[Quellcode: html]

<!-- Asynchronously load the AMP-with-Shadow-DOM runtime library. -->

&lt;script async = "" src = "https://cdn.ampproject.org/shadow-v0.js"&gt; &lt;/script&gt;

[/Quelle]

### Woher weiß ich, dass die Shadow AMP API einsatzbereit ist?

Wir empfehlen Ihnen, die Shadow AMP-Bibliothek mit dem `async` Attributsatz herunterzuladen. Dies bedeutet jedoch, dass Sie einen bestimmten Ansatz verfolgen müssen, um zu verstehen, wann die Bibliothek vollständig geladen und einsatzbereit ist.

Das richtige Signal, das Sie beobachten sollten, ist die Verfügbarkeit der globalen `AMP` Variablen, und Shadow AMP verwendet einen " [asynchronen Funktionsladeansatz](http://mrcoles.com/blog/google-analytics-asynchronous-tracking-how-it-work/) ", um dabei zu helfen. Betrachten Sie diesen Code:

[Quelle: Javascript] (window.AMP = window.AMP || []). drücken (Funktion (AMP) {// AMP ist jetzt verfügbar.}); [/Quelle]

Dieser Code funktioniert und eine beliebige Anzahl von Callbacks, die auf diese Weise hinzugefügt werden, wird tatsächlich ausgelöst, wenn AMP verfügbar ist, aber warum?

Dieser Code bedeutet übersetzt:

1. "Wenn window.AMP nicht existiert, erstellen Sie ein leeres Array, um seine Position einzunehmen."
2. "Fügen Sie dann eine Callback-Funktion in das Array ein, die ausgeführt werden soll, wenn der AMP bereit ist."

Es funktioniert, weil die Shadow AMP-Bibliothek beim tatsächlichen Laden erkennt, dass unter `window.AMP` , und dann die gesamte Warteschlange verarbeitet. Wenn Sie die gleiche Funktion später erneut ausführen, funktioniert sie immer noch, da Shadow AMP window.AMP durch sich selbst und eine benutzerdefinierte `push` Methode ersetzt, die den Callback einfach `window.AMP`

[tip type = "tip"] TIPP **.** Damit das obige Codebeispiel praktikabel ist, empfehlen wir Ihnen, es in ein Promise zu packen und dieses Promise dann immer zu verwenden, bevor Sie mit der AMP-API arbeiten. Sehen Sie sich unseren [React-Democode](https://github.com/ampproject/amp-publisher-sample/blob/master/amp-pwa/src/components/amp-document/amp-document.js#L20) für ein Beispiel an. [/tip]

## Steuern Sie die Navigation in Ihrer Progressive Web App

Sie müssen diesen Schritt weiterhin manuell ausführen. Letztendlich liegt es an Ihnen, wie Sie Links zu Inhalten in Ihrem Navigationskonzept präsentieren. Anzahl Listen? Viele Karten?

In einem typischen Szenario erhalten Sie JSON, das geordnete URLs mit einigen Metadaten zurückgibt. Letztendlich sollten Sie einen Funktionsrückruf erhalten, der ausgelöst wird, wenn der Benutzer auf einen der Links klickt, und der angegebene Rückruf sollte die URL der angeforderten AMP-Seite enthalten. Wenn Sie einen haben, sind Sie bereit für den letzten Schritt.

## Verwenden Sie die Shadow AMP API, um die Seite in einer Linie zu rendern

Wenn Sie schließlich Inhalte nach einer Benutzeraktion anzeigen möchten, ist es an der Zeit, das entsprechende AMP-Dokument zu erhalten und Shadow AMP übernehmen zu lassen. Implementieren Sie zunächst eine Funktion, um die Seite wie folgt abzurufen:

[Quelle: Javascript] Funktion fetchDocument (URL) {

// leider unterstützt fetch() das Abrufen von Dokumenten nicht, // also müssen wir auf das gute alte XMLHttpRequest zurückgreifen. var xhr = neuer XMLHttpRequest ();

neues Versprechen zurückgeben (Funktion (erlauben, ablehnen) {xhr.open ('GET', url, true); xhr.responseType = 'document'; xhr.setRequestHeader ('Accept', 'text / html'); xhr.onload = function () {// .responseXML enthält eine gebrauchsfertige Dokumentobjektauflösung (xhr.responseXML);}; xhr.send ();}); } [/Quelle]

[tip type = "important"] **WICHTIG -** Um das obige Codebeispiel zu vereinfachen, haben wir die Fehlerbehandlung weggelassen. Sie sollten immer darauf achten, Fehler richtig abzufangen und zu behandeln. [/tip]

Da wir nun unser `Document` Objekt einsatzbereit haben, ist es an der Zeit, AMP das Rendering übernehmen zu lassen. Rufen Sie einen Verweis auf das DOM-Element ab, das als Container für das AMP-Dokument dient, und rufen `AMP.attachShadowDoc()` , zum Beispiel:

[source: javascript] // Dies kann jedes DOM-Element sein var container = document.getElementById ('container');

// AMP-Seite, die Sie anzeigen möchten var url = "https: //my-domain/amp/an-article.html";

// Verwenden Sie unsere fetchDocument-Methode, um das Dokument zu erhalten fetchDocument (url) .then (function (doc) {// Lassen Sie AMP die Seite übernehmen und rendern var ampedDoc = AMP.attachShadowDoc (container, doc, url);}); [/Quelle]

[tip type = "tip"] TIPP **.** Bevor Sie das AMP-Dokument tatsächlich einreichen, müssen Sie die Seitenelemente entfernen, die beim Offline-, aber nicht Inline-Rendering der AMP-Seite sinnvoll sind: beispielsweise Fußzeilen und Kopfzeilen. ... [/tip]

Und es ist alles! Ihre AMP-Seite wird als untergeordnete Seite Ihrer generischen progressiven Web-App angezeigt.

## Raus nach dir

Die Chancen stehen gut, dass Ihr Benutzer in Ihrer Progressive Web App von AMP zu AMP wechselt. Wenn Sie eine zuvor gerenderte AMP-Seite verlassen, informieren Sie AMP immer darüber, zum Beispiel:

[Quelle: Javascript] // ampedDoc ist der Link, der von AMP.attachShadowDoc zurückgegeben wird ampedDoc.close (); [/Quelle]

Dadurch wird AMP darüber informiert, dass Sie dieses Dokument nicht mehr verwenden und Speicher- und CPU-Auslastung freigeben.

## Sehen Sie es in Aktion

[video src = "/static / img / docs / pwamp_react_demo.mp4" width = "620" height = "1100" loop = "true", control = "true"]

Sie können das AMP-in-PWA-Muster im von uns erstellten [React-Beispiel in Aktion sehen.](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa) Es demonstriert reibungslose Übergänge während der Navigation und wird mit einer einfachen React-Komponente geliefert, die die obigen Schritte ausführt. Es ist das Beste aus beiden Welten – flexibles benutzerdefiniertes JavaScript in einer progressiven Web-App und AMP für die Inhaltsverwaltung.

- Laden Sie den Quellcode hier herunter: [https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa)
- Verwenden Sie eine eigenständige React-Komponente über npm: [https://www.npmjs.com/package/react-amp-document](https://www.npmjs.com/package/react-amp-document)
- Sehen Sie hier, wie es funktioniert: [https://choumx.github.io/amp-pwa/](https://choumx.github.io/amp-pwa/) (am besten auf Telefon- oder Handy-Emulation)

Sie können auch eine Beispiel-PWA und -AMP mit dem Polymer-Framework anzeigen. Im Beispiel wird [amp-viewer](https://github.com/PolymerLabs/amp-viewer/) zum Einbetten von AMP-Seiten verwendet.

- Nimm den Code hier: [https://github.com/Polymer/news/tree/amp](https://github.com/Polymer/news/tree/amp)
- Sehen Sie es hier in Aktion: [https://polymer-news-amp.appspot.com/](https://polymer-news-amp.appspot.com/)

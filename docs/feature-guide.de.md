# Funktionsübersicht von Launchpad

Dieser Leitfaden bietet eine knappe Übersicht des Launchpad-Starterkits für potenzielle Käufer. Er fasst zusammen, was enthalten ist, was externe Dienste oder Konfiguration erfordert und was das Kit nicht automatisch bereitstellt. Der vollständige Funktionsleitfaden und der Quellcode des gekauften Projekts bleiben die maßgeblichen Referenzen für die Implementierung.

## Was Launchpad enthält

Launchpad ist eine modulare SaaS-Anwendung, die auf einer gemeinsamen Codebasis aufgebaut ist. Die optionalen Module umfassen Abrechnung, KI-Chat, Organisationen, Veröffentlichung, Support, Community, Benachrichtigungen, E-Mail-Marketing, Analysen, Empfehlungen, Werbung, SEO, Integrationen und privaten Dateispeicher. Der Einrichtungswizard ermöglicht es Ihnen, auszuwählen, welche optionalen Module beibehalten und welches Bereitstellungsziel verwendet werden soll. Die Kernkomponenten für Konten, Sicherheit, Einstellungen und die gemeinsame Datenbank bleiben bestehen.

Bei einem Git-Checkout verwendet die Einrichtung sparse-checkout, sodass ausgeschlossene, aber nachverfolgte Dateien in der Git-Verlaufshistorie verbleiben. In einem gekauften ZIP-Archiv werden die Dateien der ausgeschlossenen Module aus dem entpackten Projekt entfernt; sie können nicht aus der Git-Verlaufshistorie wiederhergestellt werden, da das ZIP keine Repository-Metadaten enthält. Dies ist eine einmalige Einrichtung und kein Werkzeug zum Umschalten von Modulen an Ort und Stelle. Sichern Sie das entpackte ZIP vor der Konfiguration, falls Sie möglicherweise später ausgeschlossenen Quellcode wiederherstellen möchten.

Der Wizard schreibt die ausgewählten Module und Bereitstellungseinstellungen in `launchpad.config.json` und die Anbieter-Geheimnisse in die von Git ignorierte Datei `.env`. Er kann die ausgewählten Anbieter-Zugangsdaten überprüfen, wenn Sie explizit **Verbindung testen** wählen. Er erstellt keine Anbieterkonten, stellt keine Infrastruktur bereit und konfiguriert keine externen Dashboards. Sie müssen diese Dienste weiterhin selbst abschließen und warten.

Der Plugin-Bereich stellt einen erlaubten Satz gebündelter Komponenten bereit; es handelt sich nicht um einen beliebigen Paketmarkt und es werden keine hochgeladenen Plugin-Codes heruntergeladen oder ausgeführt. Der optionale Event-Relay-Dienst kann separat für das derzeit unterstützte Ereignis zur Aktualisierung des Kontoprofils bereitgestellt werden. Es ist keine allgemeine Dienstepregistrierung, kein Nachrichtenbus und kein System für Bestätigungen pro Verbraucher.

## Konten und Sicherheit

Die Authentifizierung unterstützt die Anmeldung per E-Mail sowie die optionale Anmeldung über Google und GitHub. In der Entwicklungsumgebung kann Mailpit die Anmelde-E-Mails empfangen, wenn kein E-Mail-Anbieter konfiguriert ist. Bereitgestellte Umgebungen benötigen einen E-Mail-Anbieter für die E-Mail-Anmeldung und transaktionale E-Mails. Administrationsseiten und APIs erzwingen Rollen auf dem Server.

Die optionale TOTP-Mehrfachauthentifizierung funktioniert mit kompatiblen Authenticator-Apps und enthält Einmal-Wiederherstellungscodes. Die Sicherheitseinstellungen umfassen die Sitzungsverwaltung und die Überwachung von Aktivitäten. Das Kit enthält keine passwortbasierte Anmeldung und keinen Passwort-Wiederherstellungsprozess.

Die Checkliste für die Konto-Onboarding-Phase umfasst Profileinstellungen, Kontosicherheit und das Dashboard; sie ist optional und kann erneut aufgerufen werden.

## Abrechnung und kostenpflichtiger Zugriff

Das Abrechnungsmodul unterstützt Stripe Checkout für konfigurierte Abonnements und einmalige Lifetime-Käufe sowie das Stripe Customer Portal. Die optionale Kaufverifizierung über den Apple App Store und Google Play kann eine weitere Abrechnungsquelle bereitstellen, wenn die erforderlichen Produkte, Zugangsdaten, Store-Konten und die native Client-Integration konfiguriert sind. Die Store-Routen sind nicht verfügbar, bis ihre Einstellungen bereitgestellt werden; Stripe kann unabhängig weiterarbeiten.

Das Kit verarbeitet verifizierte Zahlungs- und Abonnementereignisse, protokolliert die Zahlungshistorie und behandelt Erstattungen und Streitfälle. Es schützt **nicht** automatisch jede Funktion in Ihrem Produkt nur deshalb, weil die Abrechnung aktiviert ist. `paymentBlocked` ist ein Abrechnungssignal, kein universeller Paywall-Mechanismus. Jede kostenpflichtige Funktion muss den Zugriff auf dem Server erzwingen; das Ausblenden einer Schaltfläche im Browser reicht nicht aus. Entscheiden Sie, welche Funktionen kostenpflichtig sind und was eingeschränkte Kunden weiterhin tun können, und implementieren und testen Sie dann diese Regeln. Das Kit enthält auch keinen fertigen Administrator-Workflow zur Verifizierung von Zahlungen, die außerhalb von Stripe getätigt wurden.

Die Abrechnungse-Mails verwenden den konfigurierten E-Mail-Anbieter, aber die Abrechnung benötigt keine E-Mail-Funktionalität, um zu arbeiten. Die Zustellung von E-Mails kann verzögert, fehlschlagen oder nach einer unklaren Netzwerkantwort dupliziert werden; es handelt sich nicht um ein genau-einmal-Zustellungssystem.

## E-Mail, Warteliste und Marketing

Der gemeinsame E-Mail-Transport unterstützt Resend, SendGrid und SMTP. Er wird für Authentifizierung, Abrechnungshinweise, Benachrichtigungen, Newsletter und Marketing-Nachrichten verwendet. Absenderidentitäten und Anbieter-Zugangsdaten müssen konfiguriert werden. Die SMTP-Prüfung im Einrichtungswizard authentifiziert sich, ohne eine Nachricht zu senden.

Die lokalisierte Frühzugangswarteliste erfordert eine explizite Zustimmung. Ihre Status werden manuell verwaltet: Das Einladen oder Markieren einer Person als beigetreten erstellt kein Konto, sendet keine Einladung und gewährt keinen Zugriff. Die Zustimmung zur Warteliste ist von der Newsletter-Anmeldung getrennt.

Der Newsletter unterstützt zustimmungsbasierte Anmeldungen, Abmeldelinks und von Administratoren erstellte Klartext-Kampagnen. Die Marketing-Automatisierung unterstützt kurze E-Mail-Sequenzen für neue Newsletter-Abonnenten. Diese Werkzeuge sind kein visueller E-Mail-Editor, kein System zur Segmentierung von Zielgruppen und kein Compliance- oder Rechtsdienst. Sie registrieren Wartelisteneinträge nicht automatisch und senden keine Nachrichten über Push-Benachrichtigungen und Webhooks. Geplante Kampagnen und Automatisierungen benötigen einen externen Scheduler sowie einen konfigurierten E-Mail-Anbieter.

## KI-Assistent und Organisationen

Der optionale Dashboard-Assistent verbindet sich mit einer konfigurierten OpenAI-kompatiblen Text-API. Er speichert Konversationen in der Anwendungsdatenbank, erzwingt Kontolimits für Anfragen und unterstützt persönliche und organisationsbezogene Konversationen. Er enthält kein Modell, führt keine Tools aus und unternimmt keine autonomen Aktionen. Prompts und Antworten werden an den konfigurierten Anbieter gesendet und von der Anwendung gespeichert; konfigurieren Sie die Datenschutz- und Aufbewahrungseinstellungen des Anbieters und offenbaren Sie diese Verarbeitung den Nutzern.

Organisationen bieten gemeinsame Arbeitsbereiche, Mitgliederrollen und Einladungen. Die Autorisierung wird anhand der Organisationsmitgliedschaft geprüft, aber dies ist eine Tenancy in einer gemeinsamen Datenbank, keine separate Datenbank oder Schema für jeden Kunden. Abrechnung, Kontoeinstellungen, Support, Benachrichtigungen, Analysen, Newsletter und globale Veröffentlichungsinhalte sind nicht automatisch organisationsbezogen.

## Veröffentlichung, Support und Community

Separate Module bieten einen lokalisierten Markdown-Blog, einen Feed für Release-Updates und CMS-Seiten. Administratoren verwalten Entwürfe und veröffentlichte Inhalte. Öffentliche Seiten umfassen lokalisierte Metadaten und Sitemap-Unterstützung. Die Veröffentlichungswerkzeuge enthalten keine geplante Veröffentlichung, keine automatische Übersetzung, keine Bilduploads und kein Kommentar-Moderationssystem.

Das Supportmodul bietet Kundentickets und Administrator-Antworten. Das Communitymodul bietet ein Diskussionsforum. Das kuratierte Verzeichnis ermöglicht es Administratoren, öffentliche Einträge zu verwalten. Dies sind Starter-Funktionen und keine Integrationen mit einem gehosteten Helpdesk oder einem vollständigen sozialen Netzwerk: Anhänge, Zuweisungs-/SLA-Automatisierung, Forum-Reaktionen, akzeptierte Antworten und öffentliche Nutzerprofile sind nicht enthalten.

## Benachrichtigungen und Kundendaten

Das Dashboard verfügt über ein persistente Benachrichtigungs-Eingang und optionale Zustellkanäle für E-Mail, Web Push und signierte Webhooks. Push-Benachrichtigungen erfordern VAPID-Schlüssel und Browser-Abonnements; Webhook-Verbraucher müssen Signaturen überprüfen und Ereignisse deduplizieren. Automatische Wiederholungen für Benachrichtigungen, Marketing und Speicherbereinigung benötigen extern geplante Jobs.

Die Kontoeinstellungen bieten einen JSON-Export der unterstützten Kontodaten und einen separaten Link zur Abrechnungsverwaltung. Der Export schließt Zugangsdaten, Sitzungen, Signierungsgeheimnisse und andere sensible Tokens aus. Es handelt sich um einen gestreamten Download, nicht um eine vollständige Datenbank-Sicherung oder einen Hintergrund-Archivierungsdienst.

## Analysen, Lokalisierung und Anpassung

Das Dashboard enthält Ansichten für Konten und Abrechnung. Optionale Produktanalysen sammeln zustimmungsgeschützte Ereignisse und bieten administrative Berichte; sie sind kein Ersatz für eine Drittanbieter-Analyseplattform. Attribution, Marketing-Tracking und Empfehlungscookies können in Ihrer Bereitstellung Zustimmungspflichten haben. Launchpad enthält keine Consent-Management-Plattform.

Die Oberfläche unterstützt 18 Lokalisierungen: Englisch, Arabisch, Bengalisch, Deutsch, Spanisch, Französisch, Hindi, Indonesisch, Italienisch, Japanisch, Koreanisch, Niederländisch, Polnisch, Portugiesisch, Russisch, Türkisch, Vietnamesisch und Chinesisch. Englisch ist das Fallback-Katalog und die Fallback-Sprache des Einrichtungswizards. Prüfen Sie Übersetzungen und rechtliche Texte für Ihre Zielgruppe vor dem Start.

Der Produkt-Builder bearbeitet einen erlaubten Satz von Anwendungsbereichen, Navigation, lokalisierte Texte und Markeneinstellungen. Er generiert keinen beliebigen Code, erstellt keine Anwendungsrouten und ändert keine Geschäftslogik. Einstellungen für Theme, Bewegung und andere persönliche Oberflächeneinstellungen werden im Browser gespeichert und nicht über Geräte hinweg synchronisiert.

## Speicher, Empfehlungen, Werbung und SEO

Der optionale private Dateispeicher verwendet S3-kompatible Objektspeicher. Die Anwendung validiert Dateitypen und -größen, verwendet kurzlebige Upload-/Download-URLs und prüft die Dateieigentümerschaft. Konfigurieren Sie einen privaten Bucket und die erforderlichen Zugangsdaten. Das Modul scannt Dateien nicht auf Malware, und die Kontolöschung entfernt nicht automatisch entfernte Objekte.

Das Empfehlungsmodul verfolgt Partnerlinks und Stripe-abgeleitete Provisionen. Auszahlungen werden außerhalb von Launchpad durchgeführt. Es berechnet keine Provisionen für Käufe im App Store oder Google Play. Die Empfehlungszuordnung verwendet ein 30-Tage-Cookie; Launchpad hat keinen integrierten Consent-Manager, daher müssen Sie anwendbare Zustimmungserfordernisse bewerten und implementieren.

Werkzeuge für Werbung organisieren Kampagnen- und Kontoinformationen und importieren Berichte aus CSV. Sie verbinden sich nicht mit Werbenetzwerken, um aktive Kampagnen zu erstellen oder zu verwalten. SEO-Funktionen bieten lokalisierte Metadaten, Sitemaps, robots-Regeln, strukturierte Daten und eine Auditierung der Anwendungsdatenbank; sie sind kein Webcrawler, kein Ranking-Tracker, kein Keyword-Forschungstool und kein Redirect-Manager.

Das Integrationskatalog meldet, ob die erwartete Konfiguration vorhanden ist. Es beweist nicht, dass Zugangsdaten funktionieren, erstellt keine Anbieterkonten und garantiert keine Zustellung.

## Bereitstellung und Betriebsverantwortlichkeiten

Bereitstellungen erfordern PostgreSQL und geeignete Umgebungsvariablen, einschließlich Datenbankverbindungen und `AUTH_SECRET`. Konfigurieren Sie E-Mail, Stripe, Store-Abrechnung, Objektspeicher, KI, Push und geplante Jobs nur für die Funktionen, die Sie aktivieren. Produktionsmigrationen müssen mit dem dokumentierten Bereitstellungsprozess angewendet werden; halten Sie Vorschauumgebungen auf einer isolierten Datenbank.

Die Anwendung enthält serverseitige Validierung, Rollenprüfungen, Sicherheitskontrollen und Audit-Protokollierung, aber sie ist kein vollständiges Sicherheitsbetriebsprogramm. Sie bietet keinen WAF, keine Malware-Scans, keine externe Sicherheitsüberwachung und keine strenge Content Security Policy. Konfigurieren Sie einen vertrauenswürdigen Ingress und Hosting-Schutzmaßnahmen, prüfen Sie das Bedrohungsmodell und testen Sie die bereitgestellte Anwendung vor dem Start.

Für detaillierte Konfiguration, Datenbesitz, Grenzen, Migrationen und Betriebsverfahren siehe den vollständigen Funktionsleitfaden, der mit dem Projekt ausgeliefert wird.

# Panoramica delle funzionalità di Launchpad

Questa guida è una panoramica concisa del kit di avvio Launchpad destinata ai potenziali acquirenti. Riassume cosa è incluso, cosa richiede servizi esterni o configurazione e cosa il kit non fornisce automaticamente. La guida completa delle funzionalità e il codice sorgente del progetto acquistato rimangono i riferimenti autoritativi per l'implementazione.

## Cosa include Launchpad

Launchpad è un'applicazione SaaS modulare costruita su un'unica base di codice condivisa. I suoi moduli opzionali coprono fatturazione, chat AI, organizzazioni, pubblicazione, supporto, community, notifiche, marketing via email, analisi, referral, pubblicità, SEO, integrazioni e archiviazione privata dei file. La procedura guidata di configurazione consente di scegliere quali moduli opzionali mantenere e un target di deployment. I componenti principali per account, sicurezza, impostazioni e database condiviso rimangono invariati.

In un checkout Git, la configurazione utilizza sparse-checkout, quindi i file tracciati esclusi rimangono nella cronologia di Git. In un ZIP acquistato, i file dei moduli esclusi vengono rimossi dal progetto estratto; non possono essere ripristinati dalla cronologia di Git perché il ZIP non contiene i metadati del repository. Questa è una configurazione una tantum, non uno strumento di commutazione dei moduli in loco. Effettuare un backup del ZIP estratto prima di configurarlo se si potrebbe voler recuperare successivamente il codice sorgente escluso.

La procedura guidata scrive i moduli selezionati e le preferenze di deployment in `launchpad.config.json`, e i segreti dei provider nel file `.env` ignorato da Git. Può verificare le credenziali dei provider selezionati quando si sceglie esplicitamente **Testa connessione**. Non crea account dei provider, non distribuisce infrastrutture e non configura dashboard esterni. È comunque necessario completare e mantenere quei servizi in autonomia.

L'area plugin espone un insieme di componenti inclusi in un elenco di autorizzazione; non è un marketplace di pacchetti arbitrario e non scarica o esegue codice di plugin caricato. Il servizio opzionale di relay eventi può essere distribuito separatamente per l'evento attualmente supportato di aggiornamento del profilo account. Non è un registro di servizi generale, un message bus o un sistema di conferma per singolo consumatore.

## Account e sicurezza

L'autenticazione supporta l'accesso via email e l'accesso opzionale tramite Google e GitHub. In ambiente di sviluppo, Mailpit può ricevere le email di accesso quando non è configurato un provider email. Gli ambienti distribuiti necessitano di un provider email per l'accesso via email e per le email transazionali. Le pagine e le API di amministrazione applicano i ruoli lato server.

L'autenticazione multifattore TOTP opzionale funziona con app di autenticazione compatibili e include codici di recupero usa e getta. Le impostazioni di sicurezza includono la gestione delle sessioni e l'attività di audit. Il kit non include l'accesso basato su password o un flusso di recupero password.

L'elenco di controllo per l'onboarding dell'account copre le impostazioni del profilo, la sicurezza dell'account e la dashboard; è opzionale e può essere rivisitato.

## Fatturazione e accesso a pagamento

Il modulo di fatturazione supporta Stripe Checkout per abbonamenti configurati e acquisti one-time Lifetime, oltre allo Stripe Customer Portal. La verifica opzionale degli acquisti tramite Apple App Store e Google Play può fornire un'altra fonte di fatturazione quando sono configurati i prodotti richiesti, le credenziali, gli account store e l'integrazione client nativa. I percorsi store non sono disponibili finché non vengono fornite le relative impostazioni; Stripe può continuare a operare in modo indipendente.

Il kit elabora eventi di pagamento e abbonamento verificati, registra la cronologia dei pagamenti e gestisce rimborsi e controversie. Non **protegge** automaticamente ogni funzionalità del prodotto solo perché la fatturazione è abilitata. `paymentBlocked` è un segnale di fatturazione, non un paywall universale. Ogni funzionalità a pagamento deve applicare l'accesso lato server; nascondere un pulsante nel browser non è sufficiente. Decidere quali funzionalità sono a pagamento e cosa possono ancora fare i clienti con accesso limitato, quindi implementare e testare quelle regole. Il kit non include inoltre un flusso di lavoro pronto per amministratori per la verifica dei pagamenti effettuati al di fuori di Stripe.

Le email di fatturazione utilizzano il provider email configurato, ma la fatturazione non richiede le email per funzionare. La consegna delle email può essere ritardata, fallire o essere duplicata dopo una risposta di rete incerta; non è un sistema di consegna esattamente una volta.

## Email, lista d'attesa e marketing

Il trasporto email condiviso supporta Resend, SendGrid e SMTP. Viene utilizzato per autenticazione, avvisi di fatturazione, notifiche, newsletter e messaggi di marketing. Le identità del mittente e le credenziali del provider devono essere configurate. Il controllo SMTP della procedura guidata di configurazione autentica senza inviare un messaggio.

La lista d'attesa per l'accesso anticipato localizzata richiede un consenso esplicito. I suoi stati sono gestiti manualmente: invitare o marcare qualcuno come iscritto non crea un account, non invia un invito e non concede l'accesso. Il consenso alla lista d'attesa è separato dall'iscrizione alla newsletter.

La newsletter supporta iscrizioni basate sul consenso, link di disiscrizione e campagne in testo semplice create dall'amministratore. L'automazione di marketing supporta brevi sequenze di email per nuovi iscritti alla newsletter. Questi strumenti non sono un editor email visuale, un sistema di segmentazione del pubblico o un servizio di conformità/giuridico. Non iscrivono automaticamente le voci della lista d'attesa né inviano messaggi tramite notifiche push e webhook. Le campagne programmate e l'automazione richiedono anche uno scheduler esterno oltre a un provider email configurato.

## Assistente AI e organizzazioni

L'assistente opzionale della dashboard si connette a un'API di testo compatibile con OpenAI configurata. Archivia le conversazioni nel database dell'applicazione, applica limiti di richiesta per account e supporta conversazioni personali e di organizzazione. Non include un modello, non esegue strumenti e non compie azioni autonome. Prompt e risposte vengono inviati al provider configurato e archiviati dall'applicazione; configurare le impostazioni di privacy e conservazione del provider e dichiarare questo trattamento agli utenti.

Le organizzazioni forniscono workspace condivisi, ruoli dei membri e inviti. L'autorizzazione viene verificata rispetto all'appartenenza all'organizzazione, ma si tratta di una locazione a database condiviso, non di un database o schema separato per ogni cliente. Fatturazione, impostazioni account, supporto, notifiche, analisi, newsletter e contenuti di pubblicazione globale non sono automaticamente limitati all'ambito dell'organizzazione.

## Pubblicazione, supporto e community

Moduli separati forniscono un blog Markdown localizzato, un feed di aggiornamenti di rilascio e pagine CMS. Gli amministratori gestiscono bozze e contenuti pubblicati. Le pagine pubbliche includono metadati localizzati e supporto per sitemap. Gli strumenti di pubblicazione non includono pubblicazione programmata, traduzione automatica, caricamento di immagini o un sistema di moderazione dei commenti.

Il modulo di supporto fornisce ticket per i clienti e risposte degli amministratori. Il modulo community fornisce un forum di discussione. L'indice curato consente agli amministratori di gestire voci pubbliche. Queste sono funzionalità di avvio piuttosto che integrazioni con un help desk ospitato o una rete sociale completa: allegati, automazione di assegnazione/SLA, reazioni al forum, risposte accettate e profili utente pubblici non sono inclusi.

## Notifiche e dati dei clienti

La dashboard ha una casella di notifica persistente e canali di consegna opzionali via email, Web Push e webhook firmati. Le notifiche push richiedono chiavi VAPID e sottoscrizione del browser; i consumatori dei webhook devono verificare le firme e deduplicare gli eventi. I tentativi automatici per notifiche, marketing e pulizia dello storage necessitano di job schedulati esternamente.

Le impostazioni dell'account forniscono un'esportazione JSON dei dati dell'account supportati e un link separato alla gestione della fatturazione. L'esportazione esclude credenziali, sessioni, segreti di firma e altri token sensibili. È un download in streaming, non un backup completo del database o un servizio di archiviazione in background.

## Analisi, localizzazione e personalizzazione

La dashboard include viste per account e fatturazione. L'analisi prodotto opzionale raccoglie eventi basati sul consenso e fornisce report amministrativi; non sono una sostituzione per una piattaforma di analisi di terze parti. L'attribuzione, il tracciamento di marketing e i cookie di referral possono avere obblighi di consenso nel vostro deployment. Launchpad non include una piattaforma di gestione del consenso.

L'interfaccia supporta 18 lingue: inglese, arabo, bengalese, tedesco, spagnolo, francese, hindi, indonesiano, italiano, giapponese, coreano, olandese, polacco, portoghese, russo, turco, vietnamita e cinese. L'inglese è il catalogo di fallback e la lingua di fallback della procedura guidata di configurazione. Rivedere traduzioni e testi legali per il vostro pubblico prima del lancio.

Il builder di prodotto modifica un insieme di sezioni dell'applicazione in elenco di autorizzazione, navigazione, testi localizzati e impostazioni del brand. Non genera codice arbitrario, non crea percorsi dell'applicazione e non modifica la logica di business. Tema, movimento e altre preferenze personali dell'interfaccia sono archiviati nel browser, non sincronizzati tra dispositivi.

## Archiviazione, referral, pubblicità e SEO

L'archiviazione privata dei file opzionale utilizza lo storage di oggetti compatibile con S3. L'applicazione valida tipi e dimensioni dei file, utilizza URL di caricamento/scaricamento a breve durata e verifica la proprietà del file. Configurare un bucket privato e le credenziali richieste. Il modulo non scansiona i file per malware e l'eliminazione dell'account non rimuove automaticamente gli oggetti remoti.

Il modulo referral traccia link dei partner e commissioni derivate da Stripe. I pagamenti vengono effettuati al di fuori di Launchpad. Non calcola commissioni per acquisti tramite App Store o Google Play. L'attribuzione referral utilizza un cookie di 30 giorni; Launchpad non ha un gestore del consenso integrato, quindi valutare e implementare i requisiti di consenso applicabili.

Gli strumenti di pubblicità organizzano informazioni su campagne e account e importano report da CSV. Non si connettono a reti pubblicitarie per creare o gestire campagne live. Le funzionalità SEO forniscono metadati localizzati, sitemap, regole robots, dati strutturati e un audit del database dell'applicazione; non sono un crawler web, un tracker di posizionamento, uno strumento di ricerca keyword o un gestore di redirect.

Il catalogo di integrazioni riporta se la configurazione attesa è presente. Non dimostra che le credenziali funzionino, non crea account dei provider e non garantisce la consegna.

## Deployment e responsabilità operative

I deployment richiedono PostgreSQL e variabili di ambiente appropriate, inclusi collegamenti al database e `AUTH_SECRET`. Configurare email, Stripe, fatturazione store, object storage, AI, push e job schedulati solo per le funzionalità che si abilitano. Le migrazioni di produzione devono essere applicate utilizzando il processo di deployment documentato; mantenere gli ambienti di anteprima su un database isolato.

L'applicazione include validazione lato server, controlli dei ruoli, controlli di sicurezza e logging di audit, ma non è un programma completo di operazioni di sicurezza. Non fornisce un WAF, scansione malware, monitoraggio di sicurezza esterno o una Content Security Policy rigorosa. Configurare un ingress di fiducia e protezioni di hosting, rivedere il modello di minaccia e testare l'applicazione distribuita prima del lancio.

Per la configurazione dettagliata, la proprietà dei dati, i limiti, le migrazioni e le procedure operative, consultare la guida completa delle funzionalità fornita con il progetto.

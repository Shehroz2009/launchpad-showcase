# Przegląd funkcji Launchpad

Ten przewodnik stanowi zwięzły przegląd zestawu startowego Launchpad dla potencjalnych nabywców. Podsumowuje, co jest w nim zawarte, co wymaga zewnętrznych usług lub konfiguracji, oraz czego zestaw nie zapewnia automatycznie. Pełny przewodnik po funkcjach i kod źródłowy zakupionego projektu pozostają autorytatywnymi referencjami implementacyjnymi.

## Zakup i aktualizacje

Launchpad jest regularnie aktualizowany. Aktualne opcje dostępu i warunki zakupu znajdziesz na stronie produktu w Gumroad. Launchpad to oprogramowanie, które samodzielnie hostujesz i obsługujesz dla swojego produktu, a nie hostowana usługa SaaS; hosting i usługi zewnętrznych dostawców są płatne osobno.

## Co zawiera Launchpad

Launchpad to modułowa aplikacja SaaS zbudowana na jednej wspólnej bazie kodu. Jej opcjonalne moduły obejmują rozliczenia, czat AI, organizacje, publikację, wsparcie, społeczność, powiadomienia, marketing e-mailowy, analitykę, polecenia, reklamę, SEO, integracje oraz prywatne przechowywanie plików. Kreator konfiguracji pozwala wybrać, które opcjonalne moduły zachować, oraz cel wdrożenia. Komponenty podstawowe dotyczące kont, bezpieczeństwa, ustawień i wspólnej bazy danych pozostają w miejscu.

 Wdrożenie aplikacji wymaga osobnej konfiguracji aplikacji, bazy danych i dostawców.

W przypadku pobrania przez Git, konfiguracja używa sparse-checkout, więc wykluczone śledzone pliki pozostają w historii Git. Jest to jednorazowa konfiguracja, a nie narzędzie do przełączania modułów w miejscu.

Kreator zapisuje wybrane moduły i preferencje wdrożeniowe w
`launchpad.config.json`, a sekrety dostawców w pliku `.env` ignorowanym przez Git.
Może sprawdzić wybrane dane uwierzytelniające dostawców, gdy wprost wybierzesz **Test
połączenia**. Nie tworzy kont dostawców, nie wdraża infrastruktury
ani nie konfiguruje zewnętrznych paneli administracyjnych. Nadal musisz samodzielnie
ukończyć i utrzymywać te usługi.

Obszar wtyczek udostępnia dozwolony zestaw wbudowanych komponentów; nie jest
to dowolny rynek pakietów i nie pobiera ani nie wykonuje kodu wgranych
wtyczek. Opcjonalna usługa przekaźnika zdarzeń może być wdrożona osobno dla
aktualnie wspieranego zdarzenia aktualizacji profilu konta. Nie jest to ogólny
rejestr usług, szyna komunikacyjna ani system potwierdzania odbioru dla każdego
konsumenta.

## Konta i bezpieczeństwo

Uwierzytelnianie obsługuje logowanie e-mailem oraz opcjonalne logowanie przez Google i GitHub.
W środowisku deweloperskim Mailpit może odbierać maile logowania, gdy nie jest skonfigurowany
dostawca e-mail. Środowiska wdrożone wymagają dostawcy e-mail dla logowania e-mailem
i maili transakcyjnych. Strony i interfejsy API administratora egzekwują role po stronie serwera.

Opcjonalne uwierzytelnianie dwuskładnikowe TOTP działa z kompatybilnymi
aplikacjami uwierzytelniającymi i zawiera jednorazowe kody odzyskiwania. Ustawienia
bezpieczeństwa obejmują zarządzanie sesjami i audyt aktywności. Zestaw nie zawiera
logowania opartego na hasłach ani przepływu odzyskiwania hasła.

Checklist onboardingowa konta obejmuje ustawienia profilu, bezpieczeństwo konta
i panel sterowania; jest opcjonalna i można do niej wrócić.

## Rozliczenia i płatny dostęp

Launchpad obsługuje subskrypcje i jednorazowe zakupy Lifetime przez Stripe, Paddle, Lemon Squeezy lub PayPro Global; dla każdego produktu wybiera się jednego dostawcę płatności. Dla produktów z natywnymi aplikacjami mobilnymi dostępna jest również weryfikacja zakupów Apple App Store i Google Play.

Zestaw przetwarza zweryfikowane zdarzenia płatności i subskrypcji, zapisuje
historię płatności oraz obsługuje zwroty i spory. Nie **chroni** automatycznie
każdej funkcji w Twoim produkcie tylko dlatego, że rozliczenia są włączone.
`paymentBlocked` to sygnał rozliczeniowy, a nie uniwersalny paywall. Każda płatna
funkcja musi egzekwować dostęp po stronie serwera; ukrycie przycisku w przeglądarce
jest niewystarczające. Określ, które funkcje są płatne i co ograniczeni klienci
mogą nadal robić, a następnie zaimplementuj i przetestuj te reguły. Zestaw również
nie zawiera gotowego przepływu pracy administratora do weryfikacji płatności
dokonanych poza Stripe.

E-maile rozliczeniowe używają skonfigurowanego dostawcy e-mail, ale rozliczenia
nie wymagają e-maila do działania. Dostawa e-maili może być opóźniona, nieudana
lub zduplikowana po niepewnej odpowiedzi sieciowej; nie jest to system dostawy
dokładnie raz.

## E-mail, lista oczekujących i marketing

Wspólny transport e-mail obsługuje Resend, SendGrid i SMTP. Jest używany
do uwierzytelniania, powiadomień rozliczeniowych, powiadomień, newsletterów
i wiadomości marketingowych. Tożsamości nadawców i dane uwierzytelniające
dostawców muszą być skonfigurowane. Sprawdzenie SMTP w kreatorze konfiguracji
uwierzytelnia się bez wysyłania wiadomości.

Lokalizowana lista oczekujących wczesnego dostępu wymaga wyraźnej zgody. Jej
statusy są zarządzane ręcznie: zapraszanie lub oznaczanie kogoś jako dołączonego
nie tworzy konta, nie wysyła zaproszenia ani nie udziela dostępu. Zgoda na listę
oczekujących jest odrębna od subskrypcji newslettera.

Newsletter obsługuje rejestracje oparte na zgodzie, linki do wypisania się
i kampanie w formacie plain-text tworzone przez administratora. Automatyzacja
marketingowa obsługuje krótkie sekwencje e-maili dla nowych subskrybentów
newslettera. Te narzędzia nie są wizualnym edytorem e-maili, systemem segmentacji
odbiorców ani usługą zgodności/prawną. Nie rejestrują automatycznie wpisów z listy
oczekujących ani nie wysyłają wiadomości przez powiadomienia push i webhooki.
Zaplanowane kampanie i automatyzacja wymagają również zewnętrznego harmonogramu
oraz skonfigurowanego dostawcy e-mail.

## Asystent AI i organizacje

Opcjonalny asystent panelu sterowania łączy się ze skonfigurowanym
kompatybilnym z OpenAI interfejsem API tekstowym. Przechowuje rozmowy w bazie
danych aplikacji, egzekwuje limity żądań konta i obsługuje rozmowy osobiste
i organizacyjne. Nie zawiera modelu, nie wykonuje narzędzi ani nie podejmuje
autonomicznych działań. Podpowiedzi i odpowiedzi są wysyłane do skonfigurowanego
dostawcy i przechowywane przez aplikację; skonfiguruj ustawienia prywatności
i retencji dostawcy oraz ujawnij to przetwarzanie użytkownikom.

Organizacje zapewniają wspólne przestrzenie robocze, role członków i zaproszenia.
Autoryzacja jest sprawdzana w oparciu o przynależność do organizacji, ale jest to
współdzielona dzierżawa bazy danych, a nie osobna baza danych lub schemat dla
każdego klienta. Rozliczenia, ustawienia konta, wsparcie, powiadomienia, analityka,
newslettery i globalna treść publikacji nie są automatycznie zakresowane do
organizacji.

## Publikacja, wsparcie i społeczność

Osobne moduły zapewniają lokalizowany blog Markdown, strumień aktualizacji
wydań i strony CMS. Administratorzy zarządzają szkicami i opublikowaną treścią.
Strony publiczne zawierają lokalizowane metadane i obsługę mapy witryny. Narzędzia
publikacyjne nie zawierają zaplanowanej publikacji, automatycznego tłumaczenia,
wgrania obrazów ani systemu moderacji komentarzy.

Moduł wsparcia zapewnia bilety klientów i odpowiedzi administratora. Moduł
społeczności zapewnia forum dyskusyjne. Kuratowany katalog pozwala administratorom
zarządzać publicznymi wpisami. Są to funkcje startowe, a nie integracje z
hostowanym help desk ani pełną siecią społeczną: załączniki, automatyzacja
przypisań/SLA, reakcje na forum, zaakceptowane odpowiedzi i publiczne profile
użytkowników nie są włączone.

## Powiadomienia i dane klienta

Panel sterowania ma trwały skrzynkę powiadomień i opcjonalne kanały dostawy
e-mail, Web Push i podpisanych webhooków. Powiadomienia push wymagają kluczy
VAPID i subskrypcji przeglądarki; konsumenci webhooków muszą weryfikować podpisy
i deduplikować zdarzenia. Automatyczne ponowne próby dla powiadomień, marketingu
i czyszczenia magazynu wymagają zewnętrznie zaplanowanych zadań.

Ustawienia konta zapewniają eksport JSON wspieranych danych konta i osobny link
do zarządzania rozliczeniami. Eksport wyklucza dane uwierzytelniające, sesje,
sekrety podpisu i inne wrażliwe tokeny. Jest to strumieniowane pobieranie,
a nie pełna kopia zapasowa bazy danych ani usługa archiwizacji w tle.

## Analityka, lokalizacja i dostosowanie

Panel sterowania zawiera widoki konta i rozliczeń. Opcjonalna analityka produktu
zbiera zdarzenia oparte na zgodzie i zapewnia raporty administracyjne; nie są
one zamiennikiem platformy analitycznej stron trzecich. Atrybucja, śledzenie
marketingowe i ciasteczka poleceń mogą mieć obowiązki zgodowe w Twoim wdrożeniu.
Launchpad nie zawiera platformy zarządzania zgodami.

Interfejs obsługuje 18 lokalizacji: angielski, arabski, bengalski, niemiecki,
hiszpański, francuski, hindi, indonezyjski, włoski, japoński, koreański, niderlandzki,
polski, portugalski, rosyjski, turecki, wietnamski i chiński. Język angielski jest
katalogiem zapasowym i językiem zapasowym kreatora konfiguracji. Przeglądaj tłumaczenia
i teksty prawne dla swojej grupy docelowej przed uruchomieniem.

Kreator produktu pozwala zmieniać obsługiwaną treść stron, nawigację i ustawienia marki. Edytor animacji umożliwia użytkownikom podgląd i dostosowanie obsługiwanych efektów.

## Magazyn, polecenia, reklama i SEO

Opcjonalne prywatne przechowywanie plików używa zgodnego z S3 magazynu obiektów.
Aplikacja waliduje typy i rozmiary plików, używa krótkotrwałych URL-i
wgrania/pobierania i sprawdza własność plików. Skonfiguruj prywatny bucket
i wymagane dane uwierzytelniające. Moduł nie skanuje plików pod kątem złośliwego
oprogramowania, a usunięcie konta nie usuwa automatycznie zdalnych obiektów.

Linki polecające pomagają partnerom przyciągać klientów do produktu. Nagrody uwzględniają kwalifikujące się płatności Stripe i są korygowane w przypadku zwrotów lub sporów; wypłaty odbywają się poza Launchpad.

Narzędzia reklamowe organizują informacje o kampaniach i kontach oraz importują
raporty z CSV. Nie łączą się z sieciami reklamowymi, aby tworzyć lub zarządzać
aktywnymi kampaniami. Funkcje SEO zapewniają lokalizowane metadane, mapy witryny,
reguły robots, dane strukturalne i audyt bazy danych aplikacji; nie są one
pajęczyną sieciową, śledzeniem pozycji, narzędziem do badania słów kluczowych
ani menedżerem przekierowań.

W sekcji integracji można skonfigurować obsługiwane usługi i sprawdzić wybrane połączenia.

## Wdrożenie i odpowiedzialności operacyjne

Wdrożenia wymagają PostgreSQL i odpowiednich zmiennych środowiskowych,
w tym połączeń z bazą danych i `AUTH_SECRET`. Skonfiguruj e-mail, Stripe,
rozliczenia sklepowe, magazyn obiektów, AI, push i zaplanowane zadania tylko
dla funkcji, które włączasz. Migracje produkcyjne muszą być stosowane za pomocą
udokumentowanego procesu wdrożeniowego; utrzymuj środowiska podglądu na
izolowanej bazie danych.

Aplikacja zawiera walidację po stronie serwera, sprawdzanie ról, kontrole
bezpieczeństwa i logowanie audytowe, ale nie jest kompletnym programem
operacji bezpieczeństwa. Nie zapewnia WAF, skanowania złośliwego oprogramowania,
zewnętrznego monitorowania bezpieczeństwa ani ścisłej polityki bezpieczeństwa
treści (CSP). Skonfiguruj zaufany ingress i ochrony hostingu, przeglądaj model
zagrożeń i testuj wdrożoną aplikację przed uruchomieniem.

W celu uzyskania szczegółowej konfiguracji, własności danych, limitów, migracji
i procedur operacyjnych, zobacz pełny przewodnik po funkcjach dostarczany
z projektem.

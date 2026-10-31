# Launchpad özellik genel bakışı

Bu kılavuz, potansiyel alıcılar için Launchpad başlangıç kitinin kısa bir genel bakışıdır. Kiti nelerin içerdiğini, nelerin harici hizmetler veya yapılandırma gerektirdiğini ve kitin otomatik olarak sağlamadığı şeyleri özetler. Satın alınan projenin tam özellik kılavuzu ve kaynak kodu, yetkili uygulama referansları olarak kalır.

## Launchpad'in içerdiği özellikler

Launchpad, tek bir ortak kod tabanı üzerinde inşa edilmiş modüler bir SaaS uygulamasıdır. İsteğe bağlı modülleri; faturalama, yapay zekâ sohbeti, organizasyonlar, yayınlama, destek, topluluk, bildirimler, e-posta pazarlama, analitik, yönlendirmeler, reklamcılık, SEO, entegrasyonlar ve özel dosya depolamayı kapsar. Kurulum sihirbazı, hangi isteğe bağlı modüllerin korunacağına ve bir dağıtım hedefine karar vermenizi sağlar. Temel hesap, güvenlik, ayarlar ve ortak veritabanı bileşenleri yerinde kalır.

Bir Git kontrolünde, kurulum sparse-checkout kullanır; bu sayede hariç tutulan izlenen dosyalar Git geçmişinde kalır. Satın alınan bir ZIP dosyasında ise hariç tutulan modül dosyaları çıkarılan projeden kaldırılır; ZIP dosyası depo meta verilerini içermediği için bu dosyalar Git geçmişinden geri alınamaz. Bu, yerinde modül değiştirme aracı değil, tek seferlik bir kurulumdur. Daha sonra hariç tutulan kaynakları kurtarmak isterseniz, yapılandırmadan önce çıkarılan ZIP dosyasını yedekleyin.

Sihirbaz, seçilen modülleri ve dağıtım tercihlerini `launchpad.config.json` dosyasına, sağlayıcı gizli anahtarlarını ise Git tarafından yok sayılan `.env` dosyasına yazar. **Bağlantıyı test et** seçeneğini açıkça seçtiğinizde seçilen sağlayıcı kimlik bilgilerini kontrol edebilir. Sağlayıcı hesapları oluşturmaz, altyapı dağıtmaz veya harici panoları yapılandırmaz. Bu hizmetleri yine kendiniz tamamlamanız ve sürdürmeniz gerekir.

Eklenti alanı, paketlenmiş bileşenlerin izin verilen bir listesini sunar; bu, keyfi bir paket pazarı değildir ve yüklenen eklenti kodunu indirmez veya çalıştırmaz. İsteğe bağlı olay iletim (event-relay) hizmeti, şu anda desteklenen hesap profili güncelleme olayı için ayrı olarak dağıtılabilir. Bu, genel bir hizmet kayıt defteri, mesaj sırası (message bus) veya tüketici başına onay sistemi değildir.

## Hesaplar ve güvenlik

Kimlik doğrulama, e-posta ile giriş ve isteğe bağlı Google ve GitHub girişini destekler. Geliştirme sırasında, e-posta sağlayıcısı yapılandırılmamışsa Mailpit, giriş e-postalarını alabilir. Dağıtılmış ortamlar, e-posta ile giriş ve işlem e-postaları için bir e-posta sağlayıcısı gerektirir. Yönetici sayfaları ve API'leri rolleri sunucu tarafında uygular.

İsteğe bağlı TOTP çok faktörlü kimlik doğrulama, uyumlu doğrulayıcı uygulamalarla çalışır ve tek kullanımlık kurtarma kodları içerir. Güvenlik ayarları, oturum yönetimi ve denetim etkinliğini içerir. Kit, şifre tabanlı giriş veya şifre kurtarma akışı içermez.

Hesap onboarding kontrol listesi, profil ayarlarını, hesap güvenliğini ve gösterge panelini kapsar; isteğe bağlıdır ve tekrar ziyaret edilebilir.

## Faturalama ve ücretli erişim

Faturalama modülü, yapılandırılmış abonelikler ve tek seferlik Ömür Boyu (Lifetime) satın alımları için Stripe Checkout'u, ayrıca Stripe Müşteri Portalı'nı destekler. İsteğe bağlı Apple App Store ve Google Play satın alma doğrulaması, gerekli ürünler, kimlik bilgileri, mağaza hesapları ve yerel istemci entegrasyonu yapılandırıldığında başka bir faturalama kaynağı sağlayabilir. Mağaza yolları, ayarları sağlanana kadar kullanılamaz; Stripe bağımsız olarak çalışmaya devam edebilir.

Kit, doğrulanmış ödeme ve abonelik olaylarını işler, ödeme geçmişini kaydeder ve iadeleri ile uyuşmazlıkları yönetir. Faturalama etkinleştirildi diye ürününüzdeki her özelliği **otomatik olarak korumaz**. `paymentBlocked` bir faturalama sinyalidir, evrensel bir ücret duvarı (paywall) değildir. Her ücretli özellik, erişimi sunucu tarafında uygulamalıdır; tarayıcıda bir düğmeyi gizlemek yeterli değildir. Hangi özelliklerin ücretli olduğunu ve kısıtlı müşterilerin hâlâ neler yapabileceğini belirleyin, ardından bu kuralları uygulayın ve test edin. Kit ayrıca, Stripe dışında yapılan ödemeleri doğrulamak için hazır bir yönetici iş akışı da içermez.

Faturalama e-postası, yapılandırılmış e-posta sağlayıcısını kullanır, ancak faturalamanın çalışması için e-posta gerekli değildir. E-posta iletimi, belirsiz bir ağ yanıtı sonrasında gecikebilir, başarısız olabilir veya yinelenir; bu, tam olarak bir kez teslim (exactly-once) sistemi değildir.

## E-posta, bekleme listesi ve pazarlama

Ortak e-posta iletimi, Resend, SendGrid ve SMTP'yi destekler. Kimlik doğrulama, faturalama bildirimleri, bildirimler, bültenler ve pazarlama mesajları için kullanılır. Gönderen kimlikleri ve sağlayıcı kimlik bilgileri yapılandırılmalıdır. Kurulum sihirbazının SMTP kontrolü, bir mesaj göndermeden kimlik doğrulaması yapar.

Yerelleştirilmiş erken erişim bekleme listesi, açık rıza gerektirir. Durumları manuel olarak yönetilir: birini davet etmek veya katıldı olarak işaretlemek bir hesap oluşturmaz, bir davet göndermez veya erişim vermez. Bekleme listesi rızası, bülten aboneliğinden ayrıdır.

Bülten, rıza tabanlı kayıtları, abonelikten çıkarma bağlantılarını ve yönetici tarafından oluşturulan düz metin kampanyalarını destekler. Pazarlama otomasyonu, yeni bülten aboneileri için kısa e-posta dizilerini destekler. Bu araçlar, görsel bir e-posta editörü, kitle segmentasyon sistemi veya uyumluluk/hukuk hizmeti değildir. Bekleme listesi kayıtlarını otomatik olarak kaydetmez veya mesajları push bildirimleri ve webhooks üzerinden göndermez. Zamanlanmış kampanyalar ve otomasyon, yapılandırılmış bir e-posta sağlayıcısının yanı sıra harici bir zamanlayıcı da gerektirir.

## Yapay zekâ asistanı ve organizasyonlar

İsteğe bağlı gösterge paneli asistanı, yapılandırılmış bir OpenAI uyumlu metin API'sine bağlanır. Sohbetleri uygulama veritabanında saklar, hesap istek sınırlamalarını uygular ve kişisel ile organizasyon sohbetlerini destekler. Bir model içermez, araçlar çalıştırmaz veya otonom eylemler almaz. İstekler ve yanıtlar, yapılandırılmış sağlayıcıya gönderilir ve uygulama tarafından saklanır; sağlayıcının gizlilik ve saklama ayarlarını yapılandırın ve bu işlemleri kullanıcılara bildirin.

Organizasyonlar, ortak çalışma alanları, üye rolleri ve davetler sağlar. Yetkilendirme, organizasyon üyeliğine karşı kontrol edilir, ancak bu, her müşteri için ayrı bir veritabanı veya şema değil, ortak veritabanı kiracılığıdır (tenancy). Faturalama, hesap ayarları, destek, bildirimler, analitik, bültenler ve küresel yayınlama içeriği otomatik olarak organizasyon kapsamına alınmaz.

## Yayınlama, destek ve topluluk

Ayrı modüller, yerelleştirilmiş bir Markdown blogu, sürüm güncelleme akışı ve CMS sayfaları sağlar. Yöneticiler taslakları ve yayımlanmış içeriği yönetir. Halka açık sayfalar, yerelleştirilmiş meta verileri ve site haritası (sitemap) desteğini içerir. Yayınlama araçları, zamanlanmış yayınlama, otomatik çeviri, görsel yükleme veya yorum moderasyon sistemi içermez.

Destek modülü, müşteri biletlerini ve yönetici yanıtlarını sağlar. Topluluk modülü, bir tartışma forumu sağlar. Düzenlenmiş dizin, yöneticilerin halka açık girdileri yönetmesine olanak tanır. Bunlar, barındırılan bir yardım masası veya tam bir sosyal ağ ile entegrasyonlar değil, başlangıç özellikleridir: ekler, atama/SLA otomasyonu, forum tepkileri, kabul edilen yanıtlar ve halka açık kullanıcı profilleri dahil değildir.

## Bildirimler ve müşteri verileri

Gösterge paneli, kalıcı bir bildirim gelen kutusuna ve isteğe bağlı e-posta, Web Push ve imzalı webhook iletim kanallarına sahiptir. Push bildirimleri VAPID anahtarları ve tarayıcı aboneliği gerektirir; webhook tüketicileri imzaları doğrulamalı ve olayları yinelenmeyi önlemelidir. Bildirimler, pazarlama ve depolama temizliği için otomatik yeniden denemeler, harici olarak zamanlanmış işler gerektirir.

Hesap ayarları, desteklenen hesap verilerinin JSON dışa aktarımını ve faturalama yönetimine ayrı bir bağlantıyı sağlar. Dışa aktarım, kimlik bilgilerini, oturumları, imzalama gizli anahtarlarını ve diğer hassas jetonları hariç tutar. Bu, tam bir veritabanı yedeği veya arka plan arşivleme hizmeti değil, akışkan bir indirmedir.

## Analitik, yerelleştirme ve özelleştirme

Gösterge paneli, hesap ve faturalama görünümlerini içerir. İsteğe bağlı ürün analitiği, rıza kapılı olayları toplar ve yönetici raporları sağlar; bunlar, üçüncü taraf bir analitik platformunun yerini tutmaz. Atıf (attribution), pazarlama izleme ve yönlendirme çerezleri, dağıtımınızda rıza yükümlülüklerine sahip olabilir. Launchpad, bir rıza yönetim platformu içermez.

Arayüz, 18 yereli destekler: İngilizce, Arapça, Bengalce, Almanca, İspanyolca, Fransızca, Hintçe, Endonezce, İtalyanca, Japonca, Korece, Felemenkçe, Lehçe, Portekizce, Rusça, Türkçe, Vietnamca ve Çince. İngilizce, yedek katalog ve kurulum sihirbazının yedek dilidir. Yayına çıkmadan önce çevirileri ve hukuki metinleri hedef kitleniz için gözden geçirin.

Ürün oluşturucu, uygulama bölümlerinin, gezinmenin, yerelleştirilmiş metinlerin ve marka ayarlarının izin verilen bir setini düzenler. Keyfi kod üretmez, uygulama yolları oluşturmaz veya iş mantığını değiştirmez. Tema, hareket ve diğer kişisel arayüz tercihleri tarayıcıda saklanır, cihazlar arasında senkronize edilmez.

## Depolama, yönlendirmeler, reklamcılık ve SEO

İsteğe bağlı özel dosya depolaması, S3 uyumlu nesne depolamasını kullanır. Uygulama, dosya türlerini ve boyutlarını doğrular, kısa ömürlü yükleme/indirme URL'leri kullanır ve dosya sahipliğini kontrol eder. Özel bir varlık kovası (bucket) ve gerekli kimlik bilgilerini yapılandırın. Modül, dosyaları kötü amaçlı yazılım taramaz ve hesap silme, uzak nesneleri otomatik olarak silmez.

Yönlendirme modülü, ortak bağlantılarını ve Stripe kaynaklı komisyonları izler. Ödemeler Launchpad dışında gerçekleştirilir. App Store veya Google Play satın alımları için komisyon hesaplamaz. Yönlendirme atfı, 30 günlük bir çerez kullanır; Launchpad yerleşik bir rıza yöneticisi içermediğinden, geçerli rıza gereksinimlerini değerlendirin ve uygulayın.

Reklamcılık araçları, kampanya ve hesap bilgilerini düzenler ve raporları CSV'den içe aktarır. Canlı kampanyaları oluşturmak veya yönetmek için reklam ağlarına bağlanmaz. SEO özellikleri, yerelleştirilmiş meta verileri, site haritalarını, robots kurallarını, yapılandırılmış verileri ve uygulama veritabanı denetimini sağlar; bunlar, bir web tarayıcısı (crawler), sıralama izleyicisi, anahtar kelime araştırma aracı veya yönlendirme yöneticisi değildir.

Entegrasyon kataloğu, beklenen yapılandırmanın var olup olmadığını raporlar. Kimlik bilgilerinin çalıştığını kanıtlamaz, sağlayıcı hesapları oluşturmaz veya teslimatı garanti etmez.

## Dağıtım ve işletme sorumlulukları

Dağıtımlar, veritabanı bağlantıları ve `AUTH_SECRET` dahil olmak üzere PostgreSQL ve uygun ortam değişkenlerini gerektirir. Yalnızca etkinleştirdiğiniz özellikler için e-posta, Stripe, mağaza faturalaması, nesne depolaması, yapay zekâ, push ve zamanlanmış işleri yapılandırın. Üretim göçleri (migrations), belgelenmiş dağıtım süreci kullanılarak uygulanmalıdır; önizleme ortamlarını yalıtılmış bir veritabanında tutun.

Uygulama, sunucu tarafı doğrulama, rol kontrolleri, güvenlik kontrolleri ve denetim günlüğünü içerir, ancak tam bir güvenlik operasyonları programı değildir. WAF, kötü amaçlı yazılım taraması, harici güvenlik izleme veya katı bir İçerik Güvenliği Politikası (Content Security Policy) sağlamaz. Güvenilir bir giriş (ingress) ve barındırma korumalarını yapılandırın, tehdit modelini gözden geçirin ve yayına çıkmadan önce dağıtılmış uygulamayı test edin.

Ayrıntılı yapılandırma, veri sahipliği, sınırlamalar, göçler ve operasyonel prosedürler için, projeyle birlikte sağlanan tam özellik kılavuzuna bakın.

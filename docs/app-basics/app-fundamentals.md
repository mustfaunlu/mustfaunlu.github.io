---
layout: default
title: App fundamentals
parent: App basics
nav_order: 2
---

# Application fundamentals

Android uygulamaları Kotlin, Java ve C++ dilleri kullanılarak yazılabilir. Android SDK araçları, kodunuzu herhangi bir
veri ve kaynak dosyasıyla birlikte bir APK veya Android App Bundle'da derler.

Android dosyalari derlendikten sonra .apk dosya uzantisina sahip olarak paketlenirler. Bu uzanti sayesinde uygulama
Android destekli bütün cihazlara yüklenebilir hale gelir.

Android App Bundle ise .aab uzantisi ile derlenirler. Apk dosyalarına göre daha küçük boyutlara sahiptirler içinde
gereksiz kullanilmamis yapilar bulunmaz. Google play'de uygulamalari yayınlamak için bu formatta yuklenir. Android
cihazlara direkt olarak yuklenemez. Google play kendisi içerde uygulamalari apk ya çevirip indirilmeye hazir hale
getirir. Örneğin, uygulamanızı Google Play üzerinden yayinlarken, Google Play'in sunucuları, yalnızca uygulamanın
yüklenmesini isteyen belirli bir cihazın gerektirdiği kaynakları ve kodu içeren optimize edilmiş APK'lar oluşturur.
Her android uygulamasi kendi guvenli sanal alaninda(sandbox) yasar, bu alanlar Androidin bazi ozellikleri sayesinde
korunur:

- Android işletim sistemi her uygulamayi(app) farkli bir kullanici olarak goren multi user Linux sistemidir.
- Sistem her uygulamaya bir Linux user ID si atar. Bu ID sadece sistemin kendisi tarafindan bilinir diğer uygulamalar
  bilmez. Sistem otomatik olarak uygulamadaki tüm dosyalara erisebilmesi icin o uygulamaya atanan ID ye özel izin
  ayarlar.
- Her işlemin kendi sanal makinesi(VM) olusturulur, bu nedenle bir uygulamanin kodu diğer uygulamalardan ayri olarak
  calisir.
- Yani her uygulama kendi linux surecinde calisir, Android sistemi uygulama açıldığında uygulamanın componentlerini
  baslatir ve artık ihtiyaç kalmadığında veya sistemin diğer uygulamalar için belleği kurtarması gerektiğinde işlemi
  kapatır.

Android sistemi, principle of least privilege (en az ayricalik ilkesini) uygular. Yani her uygulama isini yapmak icin
gerekli islemlere ulaşabilir , fazlasina ulaşamaz. Böylece uygulamaya izin verildigi olcude sistemin bölümlerine
erişebilir. Bu da guvenli bir ortam yaratmis olur.

Ancak bir uygulamanin diger uygulamalara ulasmasi gereken veya sistem servislerine ulasmasini gerektirecek durumlar
vardir. Bu durumlarda:

- Aynı Linux kullanıcı kimliğini paylaşacak iki uygulama ayarlamak mümkündür, bu durumda birbirlerinin dosyalarına
  erişebilirler. Sistem kaynaklarını korumak için, aynı kullanıcı kimliğine sahip uygulamalar aynı Linux sürecinde
  çalışacak ve aynı VM(sanal makineyi)'yi paylaşacak şekilde düzenlenebilir. Uygulamalar da aynı sertifika ile
  imzalanmalıdır.
- Bir uygulama, cihazın konumu, kamerası ve Bluetooth bağlantısı gibi cihaz verilerine erişmek için izin isteyebilir.
  Kullanıcının bu izinleri açıkça vermesi gerekir. Daha fazla bilgi için
  bkz. [Sistem İzinleriyle Çalışma](/docs/core-topics/permissions/about-permissions).

## App components

Uygulama componentleri, bir Android uygulamasının temel yapı taşlarıdır. Her component, sistemin veya kullanıcının
uygulamanıza girebileceği bir giriş noktasıdır. Bazı componentler diğerlerine bağlıdır. Dört farklı türde uygulama
componenti vardır:

- Activities
- Services
- Broadcast receivers
- Content providers
  Her component farklı bir amaca hizmet eder, her biri nasıl oluşturulacağını ve yok edileceğini tanımlayan farklı bir
  yaşam döngüsüne sahiptir. Bu 4 componenti daha yakından tanıyalım:

### Activities

Bir activity, kullanıcı ile etkileşim için giriş noktasıdır. Kullanıcı arayüzü olarak tek bir ekranı temsil eder.
Örneğin, bir e-posta uygulamasının; yeni e-postaların listesini gösteren bir activitysi, bir e-posta oluşturmak için
başka bir activitysi ve e-postaları okumak için başka bir activitysi olabilir. Activityler, e-posta uygulamasında uyumlu
bir kullanıcı deneyimi oluşturmak için birlikte çalışsa da, her biri diğerlerinden bağımsızdır. Bu nedenle, e-posta
uygulaması izin veriyorsa, farklı bir uygulama bu activitylerden herhangi birini başlatabilir. Örneğin, bir kamera
uygulaması, kullanıcının mailden bir resim paylaşmasına izin vermek için yeni posta oluşturan activityi, e-posta
uygulamasında başlatabilir. Bir activity, sistem ve uygulama arasında aşağıdaki temel etkileşimleri kolaylaştırır:

- Sistemin activiteyi barındıran(hosting) süreci çalıştırmaya devam etmesini sağlamak için kullanıcının şu anda neyle
  ilgilendiğini (ekranda ne olduğunu) takip etmek.
- Kullanıcının uygulama icinde farkli activityler arasında geçişler yapip geri dönebileceği şeyler (onStop a alinan
  activitler gibi) içerdiğini bilmek ve bu nedenle bu süreçleri takip altinda tutmaya daha fazla önem vermek.
- Kullanici uygulama içinde gezinirken activitlerin oldurulmesi tekrar restore edilmesi gibi süreçlerden gecer, sistemin
  bu durumlarla basa cikmasina yardimci olmak
- Uygulamaların birbirleri arasında kullanıcı akışları olmasi ve sistemin bu akışları koordine etmesi için bir rota
  sağlamak. (Buradaki en klasik örnek paylaşımdır.).

Bir [activity](https://developer.android.com/reference/android/app/Activity) class i olusturmak icin Activity classini
miras almanız gereklidir. Activity hakkinda daha fazla bilgi
almak icin, [Activities](/docs/app-architecture/app-entry-points/activities/introcution-to-activities)  kılavuzu'na
bakabilirsiniz.

### Services

Service, bir uygulamayı her türlü nedenden dolayı arka planda çalışır durumda tutmak için genel amaçlı bir entry
pointtir. Uzun süren işlemleri gerçekleştirmek veya remote işlemler için arka planda çalışan bir componentdir. Bir
kullanıcı arabirimi(UI) sağlamaz. Örneğin service, kullanıcı farklı bir uygulamadayken arka planda müzik çalabilir veya
kullanıcının bir activity ile etkileşimini engellemeden ağ üzerinden veri çekebilir. Activity gibi başka bir
componentde, service’i başlatabilir ve çalışmasına izin verebilir, ya da service ile etkileşime geçmek için ona
bağlanabilir.. Sisteme bir uygulamanın nasıl yönetileceğini söyleyen iki tür service vardır: ***started services*** ve ***bound
services***.

***Started services***, sisteme işleri tamamlanana kadar onları çalışır durumda tutmasını söyler. Bu, arka planda bazı verileri senkronize etmek veya kullanıcı uygulamadan ayrıldıktan sonra bile müzik çalmak olabilir. Arka planda verileri senkronize etmek veya müzik çalmak, sistemin bunları işleme şeklini değiştiren iki farklı started service türünü de temsil eder:
- Müzik çalma, kullanıcının doğrudan farkında olduğu bir şeydir, bu nedenle uygulama, bunu kullanıcıya bildirmek için bir bildirimle ön planda olmak istediğini söyleyerek sisteme bildirir; bu durumda sistem, bu servicein sürecini çalışır durumda tutmak için gerçekten çok uğraşması gerektiğini bilir, çünkü bu serivein ortadan kalkması durumunda kullanıcı mutsuz olacaktır.

- Normal bir arka plan servicei, kullanıcının çalıştığının doğrudan farkında olduğu bir şey değildir, bu nedenle sistem sürecini yönetmede daha fazla özgürlüğe sahiptir. Kullanıcıyı daha yakından ilgilendiren şeyler için RAM'e ihtiyaç duyarsa, öldürülmesine (ve bir süre sonra serviei yeniden başlatmasına) izin verebilir.

***Bound sevices***, başka bir uygulama (veya sistem) servicei kullanmak istediğini söylediği durumlarda çalışır. Bu temelde başka bir process için API sağlayan servicedir. Böylece sistem, bu processler arasında bir bağımlılık olduğunu bilir, dolayısıyla A process’i B process’ndeki bir service’e bağlı ise, B process’ini (ve service’ini) A için çalışır durumda tutması gerektiğini bilir. Ayrıca, eğer A process’i kullanıcıyla ilgili bir şeyse, o zaman B process’ini kullanıcının da bildigi bir şey olarak ele almayı da bilir.

Esneklikleri nedeniyle (iyi ya da kötü), serviceler her türlü üst düzey sistem konsepti için gerçekten yararlı bir yapı taşı haline gelir.  Live wallpapers, notification listeners, screen savers, input methods, accessibility services, ve diğer birçok core sistem özellikleri uygulamaların uyguladığı ve sistemin çalışması gerektiğinde bağlandığı serviceler olarak oluşturulmuştur.

Bir service, [Service](https://developer.android.com/reference/android/app/Service) sınıfından miras olarak olusturulur. Service sınıfı hakkında daha fazla bilgi için [Services](/docs/core-topics/services/about-services) kılavuzuna bakın.

{: .note}
Not: Uygulamanız Android 5.0 (API düzeyi 21) veya sonraki bir sürümünü hedefliyor ise, eylemleri planlamak için JobScheduler sınıfını kullanın. JobScheduler, güç tüketimini azaltmak için işleri en uygun şekilde planlayarak ve Doze API ile çalışarak pil tasarrufu avantajına sahiptir. Bu sınıfı kullanma hakkında daha fazla bilgi için [JobScheduler referans belgeleri](https://developer.android.com/reference/android/app/job/JobScheduler)ne bakın.

### Broadcast receivers
Broadcast receivers, sistemin eventlari app’e normal bir kullanıcı akışından farkli olarak iletilmesini sağlayan ve app’in sistem genelindeki yayın(broadcast) duyurularına yanıt vermesini sağlayan bir componentdir. Broadcast receiverlar ile sistem şu anda çalışmayan uygulamalara bile yayınlar sunabilir. Örneğin, bir uygulama, kullanıcıya yaklaşan bir etkinlik hakkında bilgi vermek için bir bildirim göndermek üzere bir alarm planlayabilir, ve bu alarmı uygulamanın BroadcastReceiver'ına ileterek, alarm kapanana kadar uygulamanın çalışır durumda kalmasına gerek kalmaz. Birçok yayın sistem tarafindan yapilir; örneğin, ekranın kapandığını, pilin azaldığını veya bir fotoğrafın çekildiğini belirten bir yayın.

Uygulamalar yayın başlatabilir; örneğin, diğer uygulamalara bazı verilerin cihaza indirildiğini ve kullanımları için hazır olduğunu bildirmek için. Broadcast receiverlar bir kullanıcı arabirimi görüntülemesede, bir yayın eventi meydana geldiğinde kullanıcıyı uyarmak için bir [durum çubuğu bildirimi](https://developer.android.com/develop/ui/views/notifications) oluşturabilirler. Daha yaygın olarak, bir broadcast receiver yalnızca diğer componentlere açılan bir geçittir ve çok az miktarda iş yapması amaçlanmıştır. Örneğin, JobScheduler ile evente dayalı olarak bazı işleri gerçekleştirmek için bir JobService planlayabilir.

Bir broadcast receiver, [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver)'ın bir alt sınıfı olarak uygulanir ve her yayın bir [Intent](https://www.google.com/url?q=https://developer.android.com/reference/android/content/Intent&sa=D&source=docs&ust=1681143002193675&usg=AOvVaw0hCk0JA3_p4CND0hN6J6YG) nesnesi olarak teslim edilir. Daha fazla bilgi için [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver) sınıfına bakın.

### Content providers
Bir content provider, dosya sisteminde, bir SQLite veritabanında, web'de veya uygulamanızın erişebileceği diğer herhangi bir kalıcı depolama konumunda paylaşılan depolayabileceğiniz bir app veri grubunu yönetir. Content provider izin veriyorsa, diğer uygulamalar, content provider aracılığıyla verileri sorgulayabilir veya değiştirebilir. Örneğin, Android sistemi, kullanıcının iletişim bilgilerini yöneten bir content provider sağlar. Bu nedenle, uygun izinlere sahip herhangi bir uygulama, belirli bir kişi hakkında bilgi okumak ve yazmak için [ContactsContract.Data](https://developer.android.com/reference/android/provider/ContactsContract.Data) gibi bir content provideri sorgulayabilir.

Bir content provideri herhangi bir veri tabanındaki abstract yapi olarak düşünmek cezbedicidir, çünkü bu yaygın durum için çok sayıda API ve destek halihazırda mevcuttur. Ancak, sistem tasarımı perspektifinden farklı bir temel amacı vardır. Sistem için bir content provider, bir URI şeması tarafından tanımlanan adlandırılmış veri öğelerini yayınlamak için bir uygulamaya giriş noktasıdır. Böylece bir uygulama, içerdiği verileri bir URI namespace’e nasıl map etmek istediğine karar verebilir ve bu URI'leri, verilere erişmek için bunları kullanabilecek diğer varlıklara(entity) dağıtabilir. . Bunun sistemin bir uygulamayı yönetirken yapmasına izin verdiği birkaç özel şey vardır:

- Bir URI atamak uygulamanın çalışır durumda kalmasını gerektirmez, bu nedenle URI'lar sahip oldukları uygulamalar çıktıktan sonra da kalabilir. Sistemin, yalnızca ilgili URI'den uygulamanın verilerini alması gerektiğinde uygulamanın hala çalıştığından emin olması gerekir.
- Bu URI'ler ayrıca önemli bir fine-grained güvenlik modeli sağlar. Örneğin, bir uygulama sahip olduğu bir resmin URI'sini panoya yerleştirebilir, ancak content providerı kilitli bırakarak diğer uygulamaların buna serbestçe erişememesini sağlayabilir. İkinci bir uygulama panodaki bu URI'ye erişmeye çalıştığında, sistem bu uygulamanın geçici bir URI izin verme yoluyla verilere erişmesine izin verebilir, böylece yalnızca bu URI'nin arkasındaki verilere erişmesine izin verilir, ancak ikinci uygulamada başka hiçbir şeye izin verilmez.

İçerik sağlayıcılar(Content providers), uygulamanıza özel olan ve paylaşılmayan verileri okumak ve yazmak için de kullanışlıdır.

Bir content provider, [ContentProvider](https://developer.android.com/reference/android/content/ContentProvider)'ın bir alt sınıfı olarak uygulanır ve diğer uygulamaların işlem gerçekleştirmesini sağlayan standart bir API kümesi uygulamalıdır. Daha fazla bilgi için [Content Providers](/docs/core-topics/app-data-and-files/content-providers/about-content-providers) kılavuzuna bakın.

Android sistem tasarımının benzersiz bir yönü, herhangi bir uygulamanın başka bir uygulamanın componentini başlatabilmesidir. Örneğin, kendi uygulamanizda kullanıcının cihazın kamerasıyla bir fotoğraf çekmesini istiyorsanız, telefonda bulunan kamera uygulamasini başlatabilirsiniz. Kamera uygulamasındaki kodu eklemeniz veya hatta bağlamanız gerekmez. Bunun yerine, fotoğraf çeken kamera uygulamasındaki activityi başlatabilirsiniz. Tamamlandığında, fotoğraf uygulamanıza yollanir, böylece onu kullanabilirsiniz. Kullanıcıya, kamera aslında uygulamanızın bir parçası gibi görünür.

Sistem bir componenti başlattığında, halihazırda çalışmıyorsa o uygulama Özelinde processi başlatır ve component için gereken sınıfları başlatır. Örneğin, uygulamanız fotoğraf çeken kamera uygulamasında activity başlatırsa, bu activity sizin uygulamanızın processinde değil, kamera uygulamasına ait süreçte çalışır. Bu nedenle, diğer çoğu sistemdeki uygulamaların aksine, Android uygulamalarının tek bir entry point yoktur (main() işlevi yoktur).

Sistem, diğer uygulamalara erişimi kısıtlayan dosya izinleriyle her uygulamayı ayrı bir processde çalıştırdığından(application fundamentals basligi altinda anlatmıştık), uygulamanız başka bir uygulamadan bir componenti doğrudan etkinleştiremez. Ancak, Android sistemi kendisi yapabilir. Bir componenti başka bir uygulamada etkinleştirmek için sisteme belirli componenti başlatma intent’inizi belirten bir mesaj iletin. Sistem daha sonra componenti sizin için baslatacaktir.


### Activate components
Dört component türünden üçü—activityler, services ve broadcast receivers—intent adı verilen asenkron bir mesajla etkinleştirilir. Intentler, çalışma zamanında(runtime) birbirinden bagimsiz componentleri birbirine bağlar. Intentleri kendi uygulamanıza veya başka bir uygulamaya ait olabilen, diğer componentlerden bir action talep eden haberciler olarak düşünebilirsiniz. Bir intent, belirli bir componenti (explicit intent) veya belirli bir component türünü (implicit intent) etkinleştirmek için bir mesaj tanımlayan bir Intent nesnesiyle oluşturulur.

Activityler ve serviceler için, intent; gerçekleştirilecek eylemi tanımlar(örneğin, bir şeyi görüntülemek veya göndermek için) ve başlatılmakta olan component bilmesi gerekebilecek diğer şeylerin yanı sıra üzerinde işlem yapılacak verilerin URI'sini belirtebilir. Örneğin intent, bir activitynin bir resmi göstermesi veya bir web sayfasını açması için talep iletebilir. Bazı durumlarda, bir sonuç almak için bir activity başlatabilirsiniz, bu durumda activity aynı zamanda sonucu bir Intent olarak döndürür. Örneğin, kullanıcının personal contact seçmesine ve size geri göndermesine izin vermek için bir intent yayınlayabilirsiniz. Dönen intent, seçilen kişiye işaret eden bir URI içerir.

Broadcast receiverlar için intent, basitçe yayınlanmakta olan duyuruyu tanımlar. Örnek olarak; intent, aygıtın pilinin zayıf olduğunu belirten cok bilinen bir eylem olan  “battery is low”  stringidir.

Activitylerin, servicelerin ve broadcast receiverlarin aksine, content providerlar intentler ile etkinleştirilmez. Bunun yerine, bir [ContentResolver](https://developer.android.com/reference/android/content/ContentResolver)'dan gelen bir istek tarafından hedeflendiğinde etkinleştirilirler. Content Resolver, content provider ile olan tüm doğrudan işlemleri gerçekleştirir, böylece provider ile işlem gerçekleştiren componentin buna ihtiyacı olmaz ve bunun yerine ContentResolver nesnesindeki methodlari çağırır. Bu, content provider ile bilgi talep eden component arasında bir soyutlama katmanı bırakır(güvenlik için).

Her component türünü etkinleştirmek için ayrı methodlar vardır:
- [startActivity()](https://developer.android.com/reference/android/content/Context#startActivity(android.content.Intent)) veya [startActivityForResult()](https://developer.android.com/reference/android/app/Activity#startActivityForResult(android.content.Intent, int)) methodlarina [Intent](https://developer.android.com/reference/android/content/Intent)  paslayarak,
- Android 5.0 (API düzeyi 21) ve sonraki sürümlerde, eylemleri(action) planlamak için [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler) sınıfını kullanabilirsiniz. Daha önceki Android sürümleri için, bir Intent öğesini [startService()](https://developer.android.com/reference/android/content/Context#startService(android.content.Intent)) methoduna ileterek bir service başlatabilir (veya devam eden bir servise yeni talimatlar verebilirsiniz). [BindService()](https://developer.android.com/reference/android/content/Context#bindService(android.content.Intent, android.content.ServiceConnection, int))'e bir Intent ileterek service bağlanabilirsiniz.
- [sendBroadcast()](https://developer.android.com/reference/android/content/Context#sendBroadcast(android.content.Intent)), [sendOrderedBroadcast()](https://developer.android.com/reference/android/content/Context#sendOrderedBroadcast(android.content.Intent, java.lang.String)), veya [sendStickyBroadcast()](https://developer.android.com/reference/android/content/Context#sendStickyBroadcast(android.content.Intent))  gibi yöntemlere bir Intent paslayarak bir broadcast başlatabilirsiniz.
- Bir [ContentResolver](https://developer.android.com/reference/android/content/ContentResolver) üzerinde [query()](https://developer.android.com/reference/android/content/ContentProvider#query(android.net.Uri, java.lang.String%5B%5D, android.os.Bundle, android.os.CancellationSignal)) methodunu çağırarak bir content providera sorgu gerçekleştirebilirsiniz.

Intentleri kullanma hakkında daha fazla bilgi için  [Intents and Intent Filters](/docs/core-topics/intents-and-intent-filters/about-intents-and-intent-filters) belgesine bakın.

## The manifest file
Android sistemi bir uygulama componentini başlatmadan önce, sistem, uygulamanın manifest dosyası AndroidManifest.xml'yi okuyarak bileşenin var olduğunu bilmelidir. Uygulamanız, uygulama proje dizininin rootunda olması gereken tüm componentleri bu dosya icinde bildirmelidir. Manifest dosyasi, uygulamanın bileşenlerini bildirmenin yanı sıra aşağıdakiler gibi birkaç şey daha yapar:
- İnternet erişimi veya kullanıcının kişilerine okuma erişimi gibi uygulamanın gerektirdiği tüm kullanıcı izinlerini tanımlar.
- Uygulamanın kullandığı API'lere bağlı olarak, uygulamanın gerektirdiği [minimum API Düzeyini](https://developer.android.com/guide/topics/manifest/uses-sdk-element#ApiLevels) bildirir.
- Kamera, bluetooth hizmetleri veya çoklu dokunmatik ekran gibi uygulama tarafından kullanılan veya gerekli olan donanım ve yazılım özelliklerini bildirir.
- Uygulamanın [Google Haritalar librarysi](http://code.google.com/android/add-ons/google-apis/maps-overview.html) gibi (Android framework API'leri dışında) bağlanması gereken API librarylerini bildirir.

### Declare components
Manifestin birincil görevi, sistemi uygulamanın componentleri hakkında bilgilendirmektir. Örneğin, bir manifest dosyası aşağıdaki gibi bir activity bildirebilir:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:icon="@drawable/app_icon.png" ... >
        <activity android:name="com.example.project.ExampleActivity"
                  android:label="@string/example_label" ... >
        </activity>
        ...
    </application>
</manifest>

```
[<application>](https://developer.android.com/guide/topics/manifest/application-element) öğesinde, android:icon attribute, uygulamayı tanımlayan bir icon için kaynaklara işaret eder.

[<activity>](https://developer.android.com/guide/topics/manifest/activity-element) öğesinde, android:name attiribute, [Activity](https://developer.android.com/reference/android/app/Activity) alt sınıfının tam nitelikli sınıf adını belirtir ve android:label attribute, activity için kullanıcı tarafından görülebilen label olarak kullanılacak bir string belirtir.

Aşağıdaki öğeleri kullanarak tüm uygulama bileşenlerini bildirmelisiniz:

[`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element) activityler icin.

[`<service>`](https://developer.android.com/guide/topics/manifest/service-element) serviceler icin.

[`<receiver>`](https://developer.android.com/guide/topics/manifest/receiver-element) broadcast receiver icin

[`<provider>`](https://developer.android.com/guide/topics/manifest/provider-element) content provider icin

Kodlariniza dahil ettiğiniz ancak manifestte beyan etmediğiniz activityler, seviceler ve content providerlar sistem tarafından görülmez ve sonuç olarak hiçbir zaman çalıştırılamaz. Ancak, broadcast receiverlari manifestte bildirilebilir veya bunun yerine kodda dinamik olarak [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver) nesneleri olarak oluşturulabilir ve [registerReceiver()](https://developer.android.com/reference/android/content/Context#registerReceiver(android.content.BroadcastReceiver,%20android.content.IntentFilter)) çağrılarak sisteme kaydedilebilirsiniz.

Uygulamanız için manifest dosyasının nasıl yapılandırılacağı hakkında daha fazla bilgi için [AndroidManifest.xml dosyası](about-app-manifests.md) belgelerine bakın.

### Declare component capabilities
Yukarıda tartışıldığı gibi, Activating Components bölümünde, activityleri, serviceleri ve broadcast receiverlari başlatmak için bir Intent kullanabilirsiniz. Intentde hedef bileşeni explicit adlandırarak (component sınıfı adını kullanarak) bir intent kullanabilirsiniz. Ayrıca, gerçekleştirilecek eylemin türünü ve isteğe bağlı olarak eylemi gerçekleştirmek istediğiniz verileri açıklayan implicit bir intentde kullanabilirsiniz. Implicit intent, sistemin cihazda eylemi gerçekleştirebilecek ve başlatabilecek bir component bulmasını sağlar. Intent tarafından açıklanan eylemi gerçekleştirebilecek birden fazla component varsa, kullanıcı hangisini kullanacağını seçer.

{: .warning }
Dikkat: Bir Service başlatmak için bir intent kullanırsanız, explicit bir intent kullanarak uygulamanızın güvenli olduğundan emin olun. Bir service başlatmak için implicit bir intent kullanmak bir güvenlik tehlikesidir, çünkü intent’e hangi hizmetin yanıt vereceğinden emin olamazsınız ve kullanıcı hangi service’in başladığını göremez. Android 5.0'dan (API düzeyi 21) başlayarak, bindService() öğesini implicit bir intent ile çağırırsanız sistem bir exception firlatir. Serviceleriniz için intent filterlar beyan etmeyin.

Sistem, alınan intenti cihazdaki diğer uygulamaların manifest dosyasında sağlanan intent filterlar ile karşılaştırarak bir intente yanıt verebilecek componentleri tanımlar.

Uygulamanızın manifest dosyasinda bir activity bildirdiğinizde, isteğe bağlı olarak, diğer uygulamalardan gelen intentlere yanıt verebilmesi için activitynin yeteneklerini bildiren intent filterleri ekleyebilirsiniz. Componentin manifest elementinin alt öğesi olarak bir <intent-filter> elementi ekleyerek componentiniz için bir intent filter bildirebilirsiniz.

Örneğin, yeni bir e-posta oluşturma activitysi olan bir e-posta uygulaması oluşturursanız, aşağıdaki örnekte gösterildiği gibi "gönderme(send)" intentlerine yanıt vermek için (yeni bir e-posta göndermek için) bir intent filter bildirebilirsiniz:

```xml
<manifest ... >
    ...
    <application ... >
        <activity android:name="com.example.project.ComposeEmailActivity">
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <data android:type="*/*" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
Başka bir uygulama, [ACTION_SEND](https://developer.android.com/reference/android/content/Intent#ACTION_SEND) action ile bir intent oluşturur ve bunu [startActivity()](https://developer.android.com/reference/android/app/Activity#startActivity(android.content.Intent)) methoduna iletirse, sistem, kullanıcının bir e-posta taslağı oluşturabilmesi ve gönderebilmesi için activitynizi başlatabilir.
Intent filterleri oluşturma hakkında daha fazla bilgi için [Intents ve Intent Filters](/docs/core-topics/intents-and-intent-filters/about-intents-and-intent-filters) belgesine bakın.

### Declare app requirements
Android tarafından desteklenen çeşitli cihazlar vardır ve bunların hepsi aynı özellikleri ve yetenekleri sağlamaz. Uygulamanızın, uygulamanızın ihtiyaç duyduğu özelliklere sahip olmayan cihazlara yüklenmesini önlemek için, manifest dosyanızda cihaz ve yazılım gereksinimlerini bildirerek uygulamanızın desteklediği cihaz türleri için açık bir şekilde bir profil tanımlamanız önemlidir. Bu bildirimlerin çoğu yalnızca bilgi amaçlıdır ve sistem bunları okumaz, ancak Google Play gibi harici hizmetler, kullanıcıların cihazlarından uygulama aradıklarında filtreleme sağlamak için bunları okur.

Örneğin, uygulamanız bir kamera gerektiriyorsa ve Android 8.0'da (API Düzeyi 26) sunulan API'leri kullanıyorsa, bu gereksinimleri beyan etmeniz gerekir. minSdkVersion ve targetSdkVersion değerleri, uygulama modülünüzün build.gradle dosyasında ayarlanır:
```gradle
android {
  ...
  defaultConfig {
    ...
    minSdkVersion 26
    targetSdkVersion 29
  }
}
```
{: .note }
Not: MinSdkVersion ve targetSdkVersion'ı doğrudan manifest dosyasında ayarlamayın, çünkü derleme işlemi sırasında Gradle bunların üzerine yazılacaktır. Daha fazla bilgi için bkz. [API düzeyi gereksinimlerini belirtme](https://developer.android.com/studio/publish/versioning#minsdkversion).

Kamera özelliğini doğrudan uygulamanızın bildirim dosyasında bildirin:
```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera.any"
                  android:required="true" />
    ...
</manifest>
```
Bu örneklerde gösterilen bildirimler ile kamerası olmayan veya Android sürümü 8.0'dan düşük olan cihazlar, uygulamanızı Google Play'den yükleyemez. Ancak, uygulamanızın kamerayı kullandığını, ancak bunu gerektirmediğini beyan edebilirsiniz. Bu durumda, uygulamanız  [`required`](https://developer.android.com/guide/topics/manifest/uses-feature-element#required) attribute false olarak ayarlamalı ve çalışma zamanında cihazın bir kamerası olup olmadığını kontrol etmeli ve uygun şekilde tüm kamera özelliklerini devre dışı bırakmalıdır.

Uygulamanızın farklı cihazlarla uyumluluğunu nasıl yönetebileceğiniz hakkında daha fazla bilgi [Device Compatibility](https://developer.android.com/guide/practices/compatibility) (Cihaz Uyumluluğu) dokumaninda sağlanır.

## App resources
Bir Android uygulaması koddan daha fazlasını içerir; görüntüler, ses dosyaları ve uygulamanın görsel sunumuyla ilgili her şey gibi kaynak koddan ayrı kaynaklar gerektirir. Örneğin, XML dosyalarıyla animasyonları, menüleri, stilleri, renkleri ve etkinlik kullanıcı arabirimlerinin düzenini tanımlayabilirsiniz. Uygulama kaynaklarını kullanmak, kodu değiştirmeden uygulamanızın çeşitli özelliklerini güncellemeyi kolaylaştırır. Alternatif kaynak grupları sağlamak, uygulamanızı farklı diller ve ekran boyutları gibi çeşitli cihaz yapılandırmaları için optimize etmenize olanak tanır.

Android projenize dahil ettiğiniz her kaynak için, SDK oluşturma toollari, uygulama kodunuzdan veya XML'de tanımlanan diğer kaynaklardan, kaynağa başvurmak için kullanabileceğiniz unique bir integer ID tanımlar. Örneğin, uygulamanız logo.png (res/drawable/ dizinine kaydedilmiş) adlı bir resim dosyası içeriyorsa, SDK araçları R.drawable.logo adlı bir resources ID oluşturur. Bu ID, resme referans vermek ve onu kullanıcı arayüzünüze eklemek için kullanabileceğiniz uygulamaya özel bir integer ile eşlenir.

Kaynak kodunuzdan ayrı kaynaklar sağlamanın en önemli yönlerinden biri, farklı cihaz yapılandırmaları için alternatif kaynaklar sağlama yeteneğidir. Örneğin, UI stringlerini XML'de tanımlayarak, stringleri diğer dillere çevirebilir ve bu stringleri ayrı dosyalara kaydedebilirsiniz. Ardından Android, kaynak dizinin adına (Fransızca dize değerleri için res/values-fr/ gibi) ve kullanıcının dil ayarına eklediğiniz bir dil niteleyicisine(qualifier) dayalı olarak kullanıcı arabiriminize uygun dil stringlerini bastirabilirsiniz. Android, alternatif kaynaklarınız için birçok farklı niteleyiciyi(qualifier) destekler. Niteleyici(qualifier), bu kaynakların kullanılacağı aygıt yapılandırmasını tanımlamak için kaynak dizinlerinizin adına eklediğiniz kısa bir dizedir. Örneğin, cihazın ekran yönüne ve boyutuna bağlı olarak activityleirniz için farklı layoutlar oluşturmalısınız. Aygıt ekranı dikey yönde (uzun) olduğunda, buttonlarin dikey olmasını isteyebilirsiniz, ancak ekran yatay yönde (geniş) olduğunda buttonlar yatay olarak hizalanabilir. Oryantasyona bağlı olarak layoutu değiştirmek için iki farklı layout tanımlayabilir ve her bir layoutun dizin adına uygun niteleyiciyi(qualifier) uygulayabilirsiniz. Ardından sistem, mevcut cihaz yönüne bağlı olarak uygun layoutu otomatik olarak uygular.

For more about the different kinds of resources you can include in your application and how to create alternative resources for different device configurations, read [Providing Resources](/docs/app-basics/app-resources/about-app-resources). To learn more about best practices and designing robust, production-quality apps, see [Guide to App Architecture](/docs/app-architecture/guide-to-app-architecture/about-app-architecture).


### [Additional resources](https://developer.android.com/guide/components/fundamentals#Resources)
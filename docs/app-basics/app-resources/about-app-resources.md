---
layout: default
title: About app resources
parent: App resources
grand_parent: App basics
nav_order: 1
---
# App resources
## About app resources
Resources, bitmap'ler, layout tanımları, kullanıcı arabirimi(UI) stringleri, animasyon yönergeleri ve daha fazlası gibi kodunuzun kullandığı ek dosyalar ve statik içeriktir.

Imageler ve stringler gibi uygulama resourcelerini her zaman kodunuzdan dışsallaştırmalısınız, böylece bunları bağımsız olarak koruyabilirsiniz. Ayrıca, bunları özel olarak adlandırılan kaynak dizinlerinde gruplayarak belirli aygıt yapılandırmaları için alternatif kaynaklar(resources) sağlamalısınız. Çalışma zamanında Android, geçerli yapılandırmaya göre uygun kaynağı kullanır. Örneğin, ekran boyutuna bağlı olarak farklı bir UI düzeni veya dil ayarına bağlı olarak farklı stringler sağlamak isteyebilirsiniz.

Uygulama kaynaklarınızı haricileştirdikten sonra, projenizin R sınıfında oluşturulan resource IDlerini kullanarak bunlara erişebilirsiniz. Bu dokuman, Android projenizde kaynaklarınızı nasıl gruplayacağınızı ve belirli cihaz yapılandırmaları için alternatif kaynaklar nasıl sağlayacağınızı ve ardından bunlara uygulama kodunuzdan veya diğer XML dosyalarından nasıl erişeceğinizi gösterir.

### Grouping resource types
Her tür kaynağı projenizin res/ dizininin belirli bir alt dizinine yerleştirmelisiniz. Örneğin, basit bir proje için dosya hiyerarşisi şu şekildedir:

```kotlin
MyProject/
    src/
        MyActivity.java
    res/
        drawable/
            graphic.png
        layout/
            main.xml
            info.xml
        mipmap/
            icon.png
        values/
            strings.xml
```
Bu örnekte görebileceğiniz gibi, res/ dizini tüm kaynakları (alt dizinlerde) içerir: bir görüntü kaynağı, iki layout kaynağı, launcher iconlar için mipmap/ dizinler ve bir string kaynak dosyası. Kaynak dizini adları önemlidir.

{: .note }
Not: Mipmap klasörlerini kullanma hakkında daha fazla bilgi için, bkz. [Uygulama simgelerini mipmap dizinlerine yerleştirme](https://developer.android.com/training/multiscreen/screendensities#mipmap).

Proje res/ dizini içinde desteklenen kaynak dizinleri:

| Directory | Resource Type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| animator/ | [Property animasyonları](Özellik animasyonlarını tanımlayan XML dosyaları.)nı tanımlayan XML dosyaları.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| anim/     | Gecis animasyonlarını( [tween animations](https://developer.android.com/guide/topics/graphics/view-animation#tween-animation).) tanımlayan XML dosyaları. (Özellik animasyonları da bu dizine kaydedilebilir, ancak özellik animasyonlarının iki türü ayırt etmesi için animatör/ dizini tercih edilir.)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| color/    | [Renklerin state listesini](https://developer.android.com/guide/topics/resources/color-list-resource) tanımlayan XML dosyaları                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| drawable/ | Aşağıdaki drawable resources alt türlerinde derlenen bitmap dosyaları (.png, .9.png, .jpg, .gif) veya XML dosyaları: Bitmap dosyaları, Nine-Patches (yeniden boyutlandırılabilir bitmapler), State listeleri, Şekiller, Animasyon drawables , Diğer drawabler Bkz. [Drawable Resources](https://developer.android.com/guide/topics/resources/drawable-resource).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| mipmap/   | Farklı başlatıcı simge yoğunlukları(densities) için çizilebilir dosyalar. Başlatıcı simgelerini mipmap/klasörlerle yönetme hakkında daha fazla bilgi için, bkz. [Put app icons in mipmap directories](https://developer.android.com/training/multiscreen/screendensities#mipmap).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| layout/   | Bir kullanıcı arabirimi düzenini tanımlayan XML dosyaları.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| menu/     | Seçenekler Menüsü, Bağlam Menüsü veya Alt Menü gibi uygulama menülerini tanımlayan XML dosyaları.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| raw/      | Ham formlarında kaydedilecek keyfi dosyalar. Bu kaynakları bir ham InputStream,  ile açmak için, R.raw.filename olan kaynak kimliğiyle Resources.openRawResource()'u arayın. Ancak, orijinal dosya adlarına ve dosya hiyerarşisine erişmeniz gerekiyorsa, bazı kaynakları assets/ dizinine (res/raw/ yerine) kaydetmeyi düşünebilirsiniz. assets/ dosyalara bir kaynak kimliği verilmez, bu nedenle bunları yalnızca AssetManager.kullanarak okuyabilirsiniz.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| values/   | Stirngler, integer ve renkler gibi basit değerler içeren XML dosyaları.  Diğer res/ alt dizinlerindeki XML kaynak dosyaları, XML dosya adına dayalı olarak tek bir kaynağı tanımlarken, values/ dizindeki dosyalar birden çok kaynağı tanımlar. Bu dizindeki bir dosya için <resources> elementinin her bir alt elementi tek bir kaynak tanımlar. Örneğin, bir <string> elementi bir R.string kaynağı oluşturur ve bir <color> elementi bir R.color kaynağı oluşturur.<br/>Her kaynak kendi XML öğesiyle tanımlandığından, dosyayı istediğiniz gibi adlandırabilir ve farklı kaynak türlerini tek bir dosyaya yerleştirebilirsiniz. Ancak, netlik sağlamak için benzersiz kaynak türlerini farklı dosyalara yerleştirmek isteyebilirsiniz. Örneğin, bu dizinde oluşturabileceğiniz kaynaklar için bazı dosya adı kuralları şunlardır:</br>arrays.xml for resource arrays ([typed arrays](https://developer.android.com/guide/topics/resources/more-resources#TypedArray)).</br>colors.xml for [color values](https://developer.android.com/guide/topics/resources/more-resources#Color)</br>dimens.xml for [dimension values](https://developer.android.com/guide/topics/resources/more-resources#Dimension).</br>strings.xml for [string values](https://developer.android.com/guide/topics/resources/string-resource). </br>styles.xml for [styles](https://developer.android.com/guide/topics/resources/style-resource).</br>See [String Resources](https://developer.android.com/guide/topics/resources/string-resource), [Style Resource](https://developer.android.com/guide/topics/resources/style-resource), and [More Resource Types](https://developer.android.com/guide/topics/resources/more-resources). |
| xml/      | Resources.getXML(). çağrılarak çalışma zamanında okunabilen keyfi XML dosyaları.  [searchable configuration](https://developer.android.com/guide/topics/search/searchable-config)  gibi çeşitli XML konfigürasyon dosyaları buraya kaydedilmelidir.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| font/     | Font/ .ttf, .otf veya .ttc gibi uzantılara sahip yazı tipi dosyaları veya bir <font-family> öğesi içeren XML dosyaları. Kaynak olarak yazı tipleri hakkında daha fazla bilgi için [Fonts in XML](https://developer.android.com/guide/topics/ui/look-and-feel/fonts-in-xml).'e gidin.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

{: .warning }

Dikkat: Kaynak dosyalarını asla doğrudan res/ dizininin içine kaydetmeyin; bu bir derleyici hatasına neden olur.

Belirli kaynak türleri hakkında daha fazla bilgi için bkz. [Resource Types](/docs/app-basics/app-resources/resource-types/about-resource-types)

Yukarida tanımlanan alt dizinlere kaydettiğiniz kaynaklar, "varsayılan" kaynaklarınızdır. Yani bu kaynaklar, uygulamanız için varsayılan tasarımı ve içeriği tanımlar. Bununla birlikte, farklı Android destekli cihaz türleri, farklı türde kaynaklar gerektirebilir. Örneğin, bir cihazın ekranı normalden daha büyükse, ekstra ekran alanından yararlanan farklı layout kaynakları sağlamalısınız. Veya bir aygıtın farklı bir dil ayarı varsa, metni kullanıcı arabiriminizde çeviren farklı string kaynakları sağlamalısınız. Bu farklı kaynakları farklı cihaz yapılandırmaları için sağlamak için varsayılan kaynaklarınıza ek olarak alternatif kaynaklar sağlamanız gerekir.

### Providing alternative resources
Hemen hemen her uygulama, belirli cihaz yapılandırmalarını desteklemek için alternatif kaynaklar sağlamalıdır. Örneğin, farklı ekran yoğunlukları için alternatif drawable kaynaklar ve farklı diller için alternatif string kaynakları eklemelisiniz. Çalışma zamanında Android, mevcut cihaz yapılandırmasını algılar ve uygulamanız için uygun kaynakları yükler.
Bir dizi kaynağa konfigürasyona özel alternatifler belirtmek için:
- res/ içinde <resources_name>-<qualifier> biçiminde adlandırılmış yeni bir dizin oluşturun.
  - <resources_name>, karşılık gelen varsayılan kaynakların dizin adıdır
  - <qualifier>, bu kaynakların kullanılacağı ayri  bir yapılandırmayı belirten bir addır. Birden fazla <qualifier> ekleyebilirsiniz. Her birini bir tire ile ayırın.
  {: .warning } 
  Dikkat: Birden çok qualifier eklerken, bunları listelendikleri sırayla yerleştirmelisiniz. Qualifierlar yanlış sıralanırsa kaynaklar yok sayılır.

- İlgili alternatif kaynakları bu yeni dizine kaydedin. Kaynak dosyaları, varsayılan kaynak dosyalarıyla tam olarak aynı şekilde adlandırılmalıdır. Örneğin, bazı varsayılan ve alternatif kaynaklar şunlardır:
```
res/
    drawable/
        icon.png
        background.png
    drawable-hdpi/
        icon.png
        background.png
```

hdpi qualifieri, o dizindeki kaynakların yüksek yoğunluklu ekrana sahip cihazlar için olduğunu belirtir.
Bu drawable dizinlerinin her birindeki görüntüler, belirli bir ekran yoğunluğu için boyutlandırılmıştır, ancak dosya adları tamamen aynıdır. Bu şekilde, icon.png veya background.png görüntüsüne başvurmak için kullandığınız kaynak ID her zaman aynıdır, ancak Android, cihaz yapılandırma bilgilerini aşağıdaki kaynak dizini adındaki niteleyicilerle karşılaştırarak mevcut cihazla en iyi eşleşen her kaynağın sürümünü seçer.

{: .warning }
Dikkat: Alternatif bir kaynak tanımlarken, kaynağı varsayılan bir konfigürasyonda da tanımladığınızdan emin olun. Aksi takdirde, cihaz bir yapılandırmayı değiştirdiğinde uygulamanız çalışma zamanı istisnalarıyla karşılaşabilir. Örneğin, values değil, yalnızca values-en diye bir dizin eklerseniz, kullanıcı varsayılan sistem dilini değiştirdiğinde uygulamanız bir Kaynak Bulunamadı istisnasıyla karşılaşabilir.

Android birkaç yapılandırma niteleyicisini destekler ve her qualifieri bir tire ile ayırarak bir dizin adına birden çok qualifier ekleyebilirsiniz. [Tabloda](https://developer.android.com/guide/topics/resources/providing-resources#AlternativeResources), geçerli yapılandırma niteleyicilerini öncelik sırasına göre listeler—bir kaynak dizini için birden çok qualifier kullanıyorsanız, bunları tabloda listelendikleri sırayla dizin adına eklemeniz gerekir.

#### Quealifier name rules

Yapılandırma niteleyici adlarını kullanmayla ilgili bazı kurallar şunlardır:
- Tek bir kaynak için tire ile ayrılmış birden çok qualifier belirtebilirsiniz. Örneğin, drawable-en-rUS-land, yatay yöndeki ABD-İngilizce cihazlar için geçerlidir.
- Qualifierler tablo 2'de listelenen sırada olmalıdır. Örneğin: Wrong:drawable-hdpi-port/     Correct:drawable-port-hdpi/
- Alternatif kaynak dizinleri iç içe yerleştirilemez. Örneğin, res/drawable/drawable-en/'e sahip olamazsınız.
- Değerler büyük/küçük harfe duyarsızdır. Kaynak derleyici, büyük/küçük harfe duyarlı olmayan dosya sistemlerinde sorunları önlemek için işlemden önce dizin adlarını küçük harfe dönüştürür. Adlardaki herhangi bir büyük harf kullanımı yalnızca okunabilirliğe fayda sağlamak içindir.
- Her qualifier türü için yalnızca bir değer desteklenir. Örneğin, İspanya ve Fransa için aynı drawable dosyaları kullanmak istiyorsanız, drawable-es-fr/ adlı bir dizine sahip olamazsınız. Bunun yerine, uygun dosyaları içeren drawable-es/ ve drawable-fr/ gibi iki kaynak dizinine ihtiyacınız vardır. Ancak, aynı dosyaları her iki konumda da çoğaltmanız gerekmez. Bunun yerine, bir kaynağa takma ad oluşturabilirsiniz. Aşağıdaki [Creating alias resources](#creating-alias-resources) konusuna bakın.

Alternatif kaynakları bu qualifierler ile adlandırılmış dizinlere kaydettikten sonra Android, mevcut cihaz yapılandırmasına göre uygulamanızdaki kaynakları otomatik olarak uygular. Bir kaynak her istendiğinde, Android, istenen kaynak dosyasını içeren alternatif kaynak dizinlerini kontrol eder ve ardından en uygun kaynağı bulur (aşağıda tartışılmıştır). Belirli bir cihaz yapılandırmasıyla eşleşen alternatif kaynak yoksa, Android karşılık gelen varsayılan kaynakları kullanır.


#### Creating alias resources
Birden fazla cihaz yapılandırması için kullanmak istediğiniz (ancak varsayılan kaynak olarak olarak kullanmak istemediğiniz) bir kaynağınız olduğunda, aynı kaynağı birden fazla alternatif kaynak dizinine koymanız gerekmez. Bunun yerine (bazı durumlarda) varsayılan kaynak dizininizde kayıtlı bir kaynak için takma ad görevi gören alternatif bir kaynak oluşturabilirsiniz.

{: .note }
Not: Tüm kaynaklar, başka bir kaynağa takma ad oluşturabileceğiniz bir mekanizma sunmaz. Özellikle xml/ dizinindeki anim, menu, raw ve diğer belirtilmemiş kaynaklar bu özelliği sunmaz.

Örneğin, icon.png adlı bir uygulama simgeniz olduğunu ve bunun farklı yerel bolgeler için benzersiz bir sürümüne ihtiyacınız olduğunu hayal edin. Ancak, İngilizce-Kanada ve Fransızca-Kanada olmak üzere iki yerel bolgenin aynı sürümü kullanması gerekir. Aynı görüntüyü hem İngilizce-Kanada dili hem de Fransızca-Kanada dili için kaynak dizinine kopyalamanız gerektiğini varsayabilirsiniz, ancak bu doğru değildir. Bunun yerine, her ikisi için kullanılan resmi icon_ca.png (icon.png dışında herhangi bir ad) olarak kaydedebilir ve varsayılan res/drawable/ dizinine koyabilirsiniz. Ardından, <bitmap> öğesini kullanarak res/drawable-en-rCA/ ve res/drawable-fr-rCA/ içinde icon_ca.png kaynağına atıfta bulunan bir icon.xml dosyası oluşturun. Bu, PNG dosyasının yalnızca bir sürümünü ve ona işaret eden iki küçük XML dosyasını saklamanıza olanak tanır. (Örnek bir XML dosyası aşağıda gösterilmiştir.)

1. Drawable:Mevcut bir drawable elemente bir takma ad oluşturmak için <drawable> elementini kullanın. Örneğin:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <drawable name="icon">@drawable/icon_ca</drawable>
</resources>

```
Bu dosyayı icon.xml (res/values-en-rCA/ gibi alternatif bir kaynak dizininde) olarak kaydederseniz, R.drawable.icon olarak başvurabileceğiniz bir kaynağa derlenir, ancak aslında  R.drawable.icon_ca kaynağı için bir aliastir. (res/drawable/ içinde kaydedilir).

2. Layout
```xml
<?xml version="1.0" encoding="utf-8"?>
<merge>
    <include layout="@layout/main_ltr"/>
</merge>
```
3. Strings and other simple values: Varolan bir dizeye takma ad oluşturmak için, yeni dizenin değeri olarak istenen dizenin kaynak IDsini kullanmanız yeterlidir. Örneğin:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hello">Hello</string>
    <string name="hi">@string/hello</string>
</resources>
```
R.string.hi kaynağı artık R.string.hello için bir takma addır.
Diğer basit değerler de aynı şekilde çalışır. Örneğin, bir renk:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="red">#f00</color>
    <color name="highlight">@color/red</color>
</resources>
```

### Accessing your app resources
Uygulamanızda bir kaynak sağladıktan sonra, kaynak IDsine başvurarak onu uygulayabilirsiniz. Tüm kaynak IDleri, projenizin aapt toolunun otomatik olarak oluşturduğu R sınıfında tanımlanır. Uygulamanız derlendiğinde aapt, res/ dizininizdeki tüm kaynaklar için kaynak IDlerini içeren R sınıfını oluşturur. Her kaynak türü için bir R alt sınıfı vardır (örneğin, tüm drawable kaynaklar için R.drawable) ve bu türdeki her kaynak için bir statik integer (örneğin, R.drawable.icon) vardır. Bu integer kaynağınızı almak için kullanabileceğiniz kaynak IDsidir. R sınıfı, kaynak IDlerinin belirtildiği yer olsa da, bir kaynak IDsini keşfetmek için asla oraya bakmanız gerekmez. Bir kaynak IDsi her zaman şunlardan oluşur:

  -	Kaynak tipi: Her kaynak, string, drawable ve layout gibi bir "tip" olarak gruplanır. Farklı tipler hakkında daha fazla bilgi için bkz. Kaynak Tipleri

  -	Aşağıdakilerden biri olan kaynak adı: uzantı hariç dosya adı veya kaynak basit bir değerse (örneğin bir string) XML android:name attributendeki değer.

Bir kaynağa erişmenin iki yolu vardır:

- ***Kodda:*** R sınıfınızın bir alt sınıfından statik bir integer kullanma, örneğin: ***R.string.hello***
string, kaynak tipidir ve hello, kaynak adıdır. Bu biçimde bir kaynak IDsi sağladığınızda kaynaklarınıza erişebilen birçok Android API'si vardır. Bkz. Koddaki Kaynaklara Erişme.

- ***XML'de:*** R sınıfınızda tanımlanan kaynak IDsine de karşılık gelen özel bir XML syntax kullanma, örneğin: ***@string/hello***

#### Accessing resources in code
Kaynak IDsini method parametresi olarak ileterek koddaki bir kaynağı kullanabilirsiniz. Örneğin, [setImageResource()](https://developer.android.com/reference/android/widget/ImageView#setImageResource(int)) kullanarak res/drawable/myimage.png kaynağını kullanmak için bir ImageView ayarlayabilirsiniz:
```kotlin
val imageView = findViewById(R.id.myimageview) as ImageView
imageView.setImageResource(R.drawable.myimage)
```
[getResources()](https://developer.android.com/reference/android/content/Context#getResources()) ile bir örneğini alabileceğiniz Resource'daki methodlari kullanarak tek tek kaynakları da alabilirsiniz.
#### Accessing resources from XML
#### Accessing original files
#### Accessing platform resources

### Providing the best device compatibility with resources
### How Android finds the best-matching resources
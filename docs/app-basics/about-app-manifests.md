---
layout: default
title: About app manifests
parent: App basics
nav_order: 1
---
# About app manifests

Her uygulama projesinin, proje kaynak kümesinin rootunda bir AndroidManifest.xml dosyası (tam olarak bu ada sahip) olması gerekir. Manifest dosyası, uygulamanızla ilgili temel bilgileri Android build toollarina, Android işletim sistemine ve Google Play'e açıklar.
Diğer birçok şeyin yanı sıra, manifest dosyasının aşağıdakileri bildirmesi gerekir:

  -	Uygulamanin componentleri, tum activityleri, serviceleri, broadcast receiverlari, ve content providerlari icerir. Her component temel özellikleri tanımlamalıdır; Kotlin veya Java sınıfının adı gibi. Ayrıca, hangi cihaz konfigürasyonlarını destekleyebilecegi ve componentleri nasıl başlatılabileceğini açıklayan intent filterlar gibi yetenekleri de bildirebilir.


  -	Sistemin korumalı bölümlerine veya diğer uygulamalara erişmek için uygulamanın ihtiyaç duyduğu izinler. Ayrıca, bu uygulamadan içeriğe erişmek istiyorlarsa diğer uygulamaların sahip olması gereken izinleri de bildirir.


  -	Uygulamanın gerektirdiği donanım ve yazılım özellikleri, uygulamayı Google Play'den hangi cihazların yükleyebileceğini etkiler. Manifest icerisind bu bilgileride beyan ederiz.

Uygulamanızı oluşturmak için Android Studio kullanıyorsanız, manifest dosyası sizin için oluşturulur ve temel manifest öğelerinin çoğu, uygulamanızı oluştururken eklenir (özellikle kod şablonlarını kullanırken).

## File features
Aşağıdaki bölümlerde, uygulamanızın en önemli özelliklerinden bazılarının manifest dosyasına nasıl eklendigi açıklanmaktadır.

### App components
Uygulamanızda oluşturduğunuz her uygulama componenti için manifest dosyasında karşılık gelen bir XML öğesi bildirmeniz gerekir:
    
      -	Activityler için <activity> elementi
      -	Service'ler için <service> elementi
      -	Broadcast receiver'lar için <receiver> elementi
      -	Content provider'lar için <provider> elementi

Bu componentlerden herhangi birini manifest dosyasında bildirmeden alt sınıflara ayırırsanız, sistem onu ​​başlatamaz. Alt sınıfınızın adı, tam paket adi kullanılarak name attribute ile belirtilmelidir. Örneğin, bir Activity alt sınıfı aşağıdaki gibi bildirilebilir:
```xml
<manifest ... >
    <application ... >
        <activity android:name="com.example.myapp.MainActivity" ... >
        </activity>
    </application>
</manifest>
```
Ancak, name değerindeki ilk karakter bir noktaysa, uygulamanın namespace i (modül düzeyindeki build.gradle dosyasının namespace özelliğinden) name in önüne cozumlenirken eklenir. Örneğin, namespace "com.example.myapp" ise, aşağıdaki activity adı "com.example.myapp.MainActivity"` olarak çözümlenir:

```xml
<manifest ... >
    <application ... >
        <activity android:name=".MainActivity" ... >
            ...
        </activity>
    </application>
</manifest>
```
Alt paketlerde (com.example.myapp.purchases gibi) bulunan uygulama componentleriniz varsa, name değeri eksik alt paket adlarını (".purchases.PayActivity" gibi) eklemeli veya fully qualified (nitelikli) paket adı kullanmalıdır.

#### Intent filters
Uygulama activityleri, serviceleri, and broadcast receiverlari intentlere göre etkinleştirilir. Bir intent, üzerinde işlem(action) yapılacak veriler, işlemi (acton) gerçekleştirmesi gereken component kategorisi ve diğer talimatlar dahil, gerçekleştirilecek bir eylemi tanımlayan bir Intent objesi tarafından tanımlanan bir mesajdır.
Bir uygulama sisteme bir intent tanimladiginda, sistem, her uygulamanın manifest dosyasındaki Intent filter bildirimlerine dayalı olarak Intenti handle edebilecek bir uygulama bileşeni bulur. Sistem, eşleşen componentin bir instanceni başlatır ve Intent nesnesini bu componente iletir. Birden fazla uygulama intenti handle edebiliyorsa, kullanıcı hangi uygulamayi kullanılacağını seçebilir. Mesela galerimizden bir fotograf paylasacagiz, fotografi secip paylas butonuna tikladigimizda sistem yukaridaki islemleri yapar ve fb, insta, whatsapp gibi secenekler gosterir.
Bir uygulama componenti, her biri o componentin farklı bir yeteneğini tanımlayan herhangi bir sayıda intent filter a (<intent-filter> öğesiyle tanımlanır) sahip olabilir.

#### Icons and labels
Bir dizi manifest elementi, ilgili uygulama componenti için kullanıcılara sırasıyla küçük bir icon ve bir text label görüntülemek için icon ve label attributelerine sahiptir.
Her durumda, bir parent elementde set edilen icon ve label, tüm child elementler için varsayılan icon ve label değeri olur. Örneğin, <application> elementinde ayarlanan icon ve label, uygulamanın her bir componenti için (tüm activityler gibi) varsayılan icon ve labeldir.
Bir componentin <intent-filter> içinde ayarlanan icon ve label, o component bir intenti yerine getirmek için bir seçenek olarak sunulduğunda kullanıcıya gösterilir. Default olarak, bu icon, parent component için bildirilen icondan ( <activity> veya <application> elementi) devralanir, seçici iletişim kutusunda(chooser dialog) daha iyi belirtmek istediğiniz unique bir eylem sağlıyorsa, bir intent filterin iconunu değiştirmek isteyebilirsiniz.

### Permissions
Android uygulamaları, hassas kullanıcı verilerine (kişiler ve SMS gibi) veya belirli sistem özelliklerine (kamera ve internet erişimi gibi) erişmek için izin istemelidir. Her izin unique bir label ile tanımlanır(“android.permission.SEND_SMS“ gibi). Örneğin, SMS mesajları göndermesi gereken bir uygulamanın manifest dosyasinda şu satırı olması gerekir:
```xml
<manifest ... >
    <uses-permission android:name="android.permission.SEND_SMS"/>
    ...
</manifest>
```
Android 6.0'dan (API düzeyi 23) başlayarak, kullanıcı runtimeda bazı uygulama izinlerini onaylayabilir veya reddedebilir. Ancak uygulamanızın hangi Android sürümünü desteklediği önemli değil, tüm izin isteklerini manifest dosyasinda bir <uses-permission> elementi ile bildirmeniz gerekir. Kullanici izin verirse, uygulama korunan özellikleri kullanabilir. Izin vermezse, bu özelliklere erişme girişimleri başarısız olur.
Uygulamanız ayrıca kendi componentilerini izinlerle koruyabilir. Android.Manifest.permission içinde listelendiği gibi Android tarafından tanımlanan izinlerden herhangi birini veya başka bir uygulamada bildirilen bir izni kullanabilir. Uygulamanız kendi izinlerini de tanımlayabilir. <permission> elementi ile yeni bir izin bildirilebilir.

### Device compatibility
Manifest dosyası ayrıca, uygulamanızın ne tür donanım veya yazılım özelliklerini gerektirdiğini ve dolayısıyla uygulamanızın hangi tür cihazlarla uyumlu olduğunu bildirebileceğiniz yerdir. Google Play Store, uygulamanızın gerektirdiği özellikleri veya sistem sürümünü sağlamayan cihazlara uygulamanızın yüklenmesine izin vermez. Uygulamanızın hangi cihazlarla uyumlu olduğunu tanımlayan birkaç manifest tagi vardır. Aşağıdakiler en yaygın etiketlerden sadece birkaçıdır.

***<uses-feature>:***  elementi, uygulamanızın ihtiyaç duyduğu donanım ve yazılım özelliklerini bildirmenize olanak tanır. Örneğin, uygulamanız pusula sensörü olmayan bir cihazda temel işlevleri gerçekleştiremezse pusula sensörünü aşağıdaki manifest tagiyle gerektiği gibi bildirebilirsiniz:
```xml
<manifest ... >
    <uses-feature android:name="android.hardware.sensor.compass"
                  android:required="true" />
    ...
</manifest>
```

***<uses-sdk>:*** Birbirini izleyen her platform sürümü, genellikle önceki sürümde bulunmayan yeni API'ler ekler. Uygulamanızın uyumlu olduğu minimum sürümü belirtmek için manifestiniz `<uses-sdk>` tagini ve minSdkVersion attributeni içermelidir. Ancak, <uses-sdk> elementindeki attributelerin build.gradle dosyasındaki karşılık gelen propertyler tarafından override edildigine dikkat edin. Dolayısıyla, Android Studio kullanıyorsanız, bunun yerine minSdkVersion ve targetSdkVersion değerlerini belirtmelisiniz:
```groovy
android {
    defaultConfig {
        applicationId 'com.example.myapp'

        // Defines the minimum API level required to run the app.
        minSdkVersion 21

        // Specifies the API level used to test the app.
        targetSdkVersion 33
        ...
    }
}
```
## File conventions
Bu bölüm, manifest dosyasındaki tüm elementler ve attributeslere genel olarak uygulanan sözleşmeler ve kurallari açıklar.

***Elements***
Yalnızca `<manifest>` ve `<application>` elementleri gereklidir. Her biri yalnızca bir kez yazilir. Diğer elementlerin çoğu sıfır veya daha fazla kez oluşabilir. Ancak, manifest dosyasının kullanışlı olması için bazılarının mevcut olması gerekir.

Tüm değerler, bir öğe içindeki karakter verileri olarak değil, attributeler aracılığıyla ayarlanır.
Aynı seviyedeki elementler genellikle sıralanmaz. Örneğin, `<activity>`, `<provider>` ve `<service>` öğeleri herhangi bir sırada yerleştirilebilir. Bu kuralın iki önemli istisnası vardır:

Bir `<activity-alias>` öğesi, diğer adı olan `<activity>` öğesini izlemelidir.

`<application>` öğesi,  `<manifest>` öğesi içindeki son element olmalıdır.

***Attributes***

Teknik olarak, butun attributeler opsiyoneldi. Ancak, bir elementin amacını gerçekleştirebilmesi için bircok attribute belirtilmek zorundadir. Gercekten opsiyonel attributeler, [referans dokumantasyonda](#manifest-elements-reference) default degerleri ile gosterilir.

Root `<manifest>` elementinin bazi attributeleri disinda, butun attributelerin adlari `android:`  oneki(prefix) ile baslar. Ornegin, `android:alwaysRetainTaskState`. Cunku prefixler evrenseldir, bu dokumantasyon, attributelere isimle atıfta bulunurken genellikle prefixi atlar.

***Multiple Values***

Birden fazla değer belirtilebilirse, tek bir element içinde birden çok değerin listelenmesi yerine, element hemen hemen her zaman tekrarlanır. Örneğin, bir intent filter birkaç action listeleyebilir:
```xml
<intent-filter ... >
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.INSERT" />
    <action android:name="android.intent.action.DELETE" />
    ...
</intent-filter>
```

***Resource values***

Bazı attributelerin, bir activitynin başlığı veya uygulamanızın iconu gibi kullanıcılara görüntülenen değerleri vardır. Bu attributelerin değeri, kullanıcının diline veya diğer cihaz konfigürasyonlarına (cihazın piksel yoğunluğuna göre farklı bir icon boyutu sağlamak gibi) bağlı olarak farklılık gösterebilir, bu nedenle manifest dosyasına sabit kodlanmış değerler yerine bir kaynaktan veya temadan ayarlanmalıdır. Guncel değer daha sonra farklı cihaz konfigürasyonları için sağladığınız alternatif kaynaklara göre değişebilir.

Kaynaklar, aşağıdaki formatta değerler olarak ifade edilir:
`"@[package:]type/name"`

Kaynak uygulamanız tarafından sağlanıyorsa paket adını atlayabilirsiniz (library kaynakları sizinkiyle birleştirildiğinden(merge), bir library bağımlılığı tarafından sağlanıp sağlanmadığı dahil). Android frameworkunden bir kaynak kullanmak istediğinizde, tek geçerli paket adı, `android` dir.

Type, string veya drawable gibi bir kaynak türüdür ve name, belirli kaynağı tanımlayan addır. İşte bir örnek:
```xml
<activity android:icon="@drawable/smallPic" ... >
```

***String values***

Attribute değeri bir string olduğunda, karakterlerden kaçmak için çift ters eğik çizgi (\\) kullanmalısınız, örneğin yeni satır için \\n veya Unicode karakter için \\uxxxx gibi.


[## Manifest elements reference](https://developer.android.com/guide/topics/manifest/manifest-intro#reference)


## Example manifest file
Aşağıdaki XML, uygulama için iki activity bildiren basit bir `AndroidManifest.xml` örneğidir.


```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:versionCode="1"
    android:versionName="1.0">

    <!-- Beware that these values are overridden by the build.gradle file -->
    <uses-sdk android:minSdkVersion="15" android:targetSdkVersion="26" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <!-- This name is resolved to com.example.myapp.MainActivity
             based on the namespace property in the build.gradle file -->
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".DisplayMessageActivity"
            android:parentActivityName=".MainActivity" />
    </application>
</manifest>
```
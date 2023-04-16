---
layout: default
title: About Play Feature Delivery
grand_parent: Google Play
nav_order: 1
parent: Play Feature Delivery
---

# Overview of Play Feature Delivery

Google Play'in uygulama sunma modeli, her kullanıcının cihaz yapılandırması için optimize edilmiş APK'lar oluşturmak ve
sunmak için [Android App Bundle](/docs/core-topics/android-app-bundles/android-app-bundles.md)'ları kullanır, böylece kullanıcılar yalnızca uygulamanızı çalıştırmak için ihtiyaç
duydukları kodu ve kaynakları indirir.

Play Feature Delivery, app bundle'ların gelişmiş özelliklerini kullanarak uygulamanızın belirli özelliklerinin koşullu
olarak sunulmasına veya isteğe bağlı olarak indirilmesine olanak tanır. Bunu yapmak için öncelikle bu özellikleri base
app'inizden feature modüllerine ayırmanız gerekir.

## Feature module build configuration

Android Studio kullanarak yeni bir feature modülü oluşturduğunuzda, IDE aşağıdaki Gradle eklentisini modülün `build.gradle` dosyasına uygular.

```groovy
// The following applies the dynamic-feature plugin to your feature module.
// The plugin includes the Gradle tasks and properties required to configure and build
// an app bundle that includes your feature module.

plugins {
  id 'com.android.dynamic-feature'
}

```
Standart application plugin için kullanılabilen özelliklerin çoğu feature modülünüz için de kullanılabilir. Aşağıdaki bölümlerde, feature modülünüzün build configuration'ına dahil etmeniz ve etmemeniz gereken özellikler açıklanmaktadır.

### What not to include in the feature module build configuration
Her feature modülü base modüle bağlı olduğundan, belirli yapılandırmaları da miras alır. Bu nedenle, feature modülünün `build.gradle` dosyasında aşağıdakileri eklememelisiniz:

- ***Signing configurations:*** App bundle'lar, base modülde belirttiğiniz signing configurations kullanılarak imzalanır.


- ***The `minifyEnabled` property:*** Tüm uygulama projeniz için [kod küçültme](https://developer.android.com/build/shrink-code#shrink-code)yi yalnızca base modülün build configuration'undan etkinleştirebilirsiniz. Bu nedenle, bu özelliği feature modüllerinden çıkarmanız gerekir. Bununla birlikte, her feature modülü için [ek ProGuard kuralları](#specify-additional-proguard-rules) belirleyebilirsiniz.


- ***`versionCode` ve `versionName`:*** Gradle, app bundle'ınızı oluştururken base modülün sağladığı app versiyon bilgilerini kullanır. Bu özellikleri feature modülünüzün `build.gradle` dosyasından çıkarmanız gerekir.

### Establish a relationship to the base module
Android Studio feature modülünüzü oluşturduğunda, aşağıda gösterildiği gibi base modülün `build.gradle` dosyasına `android.dynamicFeatures` property'sini ekleyerek onu base modül için görünür hale getirir:
```groovy
// In the base module’s build.gradle file.
android {
    ...
    // Specifies feature modules that have a dependency on
    // this base module.
    dynamicFeatures = [":dynamic_feature", ":dynamic_feature2"]
}
```
Ayrıca Android Studio, aşağıda gösterildiği gibi base modülü feature modülünün bir bağımlılığı olarak içerir:
```groovy
// In the feature module’s build.gradle file:
...
dependencies {
    ...
    // Declares a dependency on the base module, ':app'.
    implementation project(':app')
}
```

### Specify additional ProGuard rules
Yalnızca base modülün build configuration'ı uygulama projeniz için kod küçültmeyi etkinleştirebilse de, aşağıda gösterildiği gibi [proguardFiles](https://developer.android.com/reference/tools/gradle-api) özelliğini kullanarak her feature modülüyle birlikte özel ProGuard kuralları sağlayabilirsiniz.
```groovy
android.buildTypes {
     release {
         // You must use the following property to specify additional ProGuard
         // rules for feature modules.
         proguardFiles 'proguard-rules-dynamic-features.pro'
     }
}
```
Bu ProGuard kurallarının derleme sırasında diğer modüllerdekilerle (base modül dahil) birleştirildiğini unutmayın. Dolayısıyla, her feature modülü yeni bir kural kümesi belirleyebilse de, bu kurallar uygulama projesindeki tüm modüller için geçerlidir.

## Deploy your app
Uygulamanızı feature modülleri desteğiyle geliştirirken, menü çubuğundan Run > Run'ı seçerek (veya araç çubuğunda Run'a tıklayarak) uygulamanızı normalde yaptığınız gibi bağlı bir cihaza deploy edebilirsiniz.

Uygulama projeniz bir veya daha fazla feature modülü içeriyorsa, mevcut [run/debug configuration](https://developer.android.com/studio/run/rundebugconfig)'nızı aşağıdaki gibi değiştirerek uygulamanızı deploy ederken hangi özelliklerin dahil edileceğini seçebilirsiniz:

- Menü çubuğundan Run (Çalıştır) > Edit Configurations (Yapılandırmaları Düzenle) öğesini seçin.
- Çalıştır/Debug Yapılandırmaları iletişim kutusunun sol panelinden istediğiniz Android App configuration seçin.
- Genel sekmesindeki Dağıtılacak dinamik özellikler altında, uygulamanızı dağıtırken dahil etmek istediğiniz her özellik modülünün yanındaki kutuyu işaretleyin.
- Tamam'a tıklayın.


Varsayılan olarak, Android Studio uygulamanızı deploy etmek için app bundle'ları kullanmaz. Bunun yerine IDE, APK boyutu yerine deployment hızı için optimize edilmiş APK'lar oluşturur ve cihazınıza yükler. Android Studio'yu bunun yerine APK'ları ve anlık deneyimleri bir app bundle'dan derleyip deploy edecek şekilde yapılandırmak için [run/debug configuration'ınızı değiştirin](https://developer.android.com/studio/run/rundebugconfig#android-application).

## Use feature modules for custom delivery
Feature modüllerinin benzersiz bir avantajı, uygulamanızın farklı özelliklerinin Android 5.0 (API düzeyi 21) veya daha yüksek sürümleri çalıştıran cihazlara nasıl ve ne zaman indirileceğini özelleştirebilmenizdir. Örneğin, uygulamanızın ilk indirme boyutunu azaltmak için, belirli özellikleri isteğe bağlı olarak indirilecek şekilde ya da yalnızca fotoğraf çekme veya artırılmış gerçeklik özelliklerini destekleme gibi belirli özellikleri destekleyen cihazlar tarafından indirilecek şekilde yapılandırabilirsiniz.

Uygulamanızı bir app bundle olarak yüklediğinizde varsayılan olarak son derece optimize edilmiş indirmeler elde etseniz de, daha gelişmiş ve özelleştirilebilir feature delivery seçenekleri, feature modülleri kullanarak uygulamanızın özelliklerinin ek yapılandırılmasını ve modüler hale getirilmesini gerektirir. Yani feature modülleri, her biri gerektiğinde indirilecek şekilde yapılandırabileceğiniz modüler özellikler oluşturmak için yapı taşları sağlar.

Kullanıcılarınızın çevrimiçi bir pazarda mal alıp satmasına olanak tanıyan bir uygulama düşünün. Uygulamanın aşağıdaki işlevlerinin her birini ayrı feature modülleri halinde makul bir şekilde modüler hale getirebilirsiniz:

- Hesap girişi ve oluşturma
- Pazaryerinde gezinme
- Satış için bir ürün yerleştirme
- Ödemelerin işlenmesi


Aşağıdaki tabloda feature modüllerinin desteklediği farklı teslimat seçenekleri ve bunların örnek market uygulamasının ilk indirme boyutunu optimize etmek için nasıl kullanılabileceği açıklanmaktadır.

| Delivery option       | Behavior                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Sample use-case                                                                                                                                                                                                                                                                                                                                | Getting started                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Install-time delivery | Yukarıda açıklanan teslim seçeneklerinden herhangi birini yapılandırmayan feature modülleri, varsayılan olarak uygulama yüklendiğinde indirilir. Bu önemli bir davranıştır çünkü gelişmiş teslimat seçeneklerini kademeli olarak benimseyebileceğiniz anlamına gelir. Örneğin, uygulamanızın özelliklerini modüler hale getirmekten faydalanabilir ve isteğe bağlı teslimi(on-demand delivery) ancak Play Feature Delivery Library'yi kullanarak isteğe bağlı indirmeleri tam olarak uyguladıktan sonra etkinleştirebilirsiniz. Buna ek olarak, uygulamanız özellikleri daha sonra kaldırmayı talep edebilir. Yani, uygulama yüklenirken belirli özelliklere ihtiyaç duyuyor ancak sonrasında ihtiyaç duymuyorsanız, özelliğin cihazdan kaldırılmasını talep ederek yükleme boyutunu azaltabilirsiniz. | Uygulamanın, pazaryerinde nasıl ürün alınıp satılacağına ilişkin etkileşimli bir kılavuz gibi belirli eğitim activityleri varsa, bu özelliği varsayılan olarak uygulama yüklemesine dahil edebilirsiniz.</br>Ancak, uygulamanın yüklü boyutunu azaltmak için, kullanıcı eğitimi tamamladıktan sonra uygulama feature'ı silmeyi talep edebilir. | Gelişmiş delivery seçeneklerini yapılandırmayan feature modüllerini kullanarak [Modularize your app](#about-play-feature-delivery).</br>Kullanıcının artık ihtiyaç duymayabileceği belirli feature modüllerini kaldırarak uygulamanızın yüklü boyutunu nasıl azaltacağınızı öğrenmek için [Manage installed modules](#configure-on-demand-delivery#manage-installed-modules) bölümünü okuyun.                                                                                                                                                                                                                                                                                                                         |
| On demand delivery    | Uygulamanızın gerektiğinde feature modülleri talep etmesini ve indirmesini sağlar.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Pazaryeri uygulamasını kullananların yalnızca %20'si satış için ürün gönderiyorsa, kullanıcıların çoğunluğu için ilk indirme boyutunu azaltmak için iyi bir strateji, fotoğraf çekme, ürün açıklaması ekleme ve satış için bir ürün yerleştirme işlevlerini isteğe bağlı indirme olarak kullanılabilir hale getirmektir. Yani, uygulamanın satış işlevi için feature modülünü yalnızca bir kullanıcı pazaryerine satılık ürün yerleştirmeye ilgi gösterdiğinde indirilecek şekilde yapılandırabilirsiniz.</br>Ek olarak, kullanıcı belirli bir süre sonra artık ürün satmazsa, uygulama özelliği kaldırmayı talep ederek yüklü boyutunu azaltabilir.                                                                                                                                                                                                                                                                                                                                               | Bir feature modülü  ve [configure on demand delivery](docs/core-topics/google-play/play-feature-delivery/configure-on-demand-delivery) oluşturun. Uygulamanız daha sonra modülü on demand(talep uzerine) indirmeyi istemek için [Play Feature Delivery Library](docs/core-topics/google-play/google-play-core-libraries#integrate-the-play-feature-delivery-library) kullanabilir.                                                                                                                                                                                                                                                                                                                                    |
| Conditional delivery  | Modülerleştirilmiş bir özelliğin uygulama yüklemesinde indirilip indirilmeyeceğini belirlemek için donanım özellikleri, yerel ayar ve minimum API düzeyi gibi belirli kullanıcı cihazı gereksinimlerini belirtmenize olanak tanır.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Pazaryeri uygulaması küresel erişime sahipse, yalnızca belirli bölgelerde veya yerel halkta popüler olan ödeme yöntemlerini desteklemeniz gerekebilir. İlk uygulama indirme boyutunu azaltmak için, belirli ödeme yöntemlerini işlemek üzere ayrı feature modülleri oluşturabilir ve bunların kayıtlı yerel ayarlarına göre kullanıcının cihazına koşullu olarak yüklenmesini sağlayabilirsiniz.                                                                                                                                                                                                                                                                                                                                               | Bir feature modülü oluşturun ve [conditional delivery](docs/core-topics/google-play/play-feature-delivery/configure-conditional-delivery)'i yapılandırın.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Instant delivery      | [Google Play Instant](doc/core-topics/google-play/google-play-instant/about-google-play-instant), kullanıcıların uygulamayı cihazlarına yüklemelerine gerek kalmadan uygulamanızla etkileşime geçmelerini sağlar. Bunun yerine, Google Play Store'daki "Şimdi Dene" düğmesi veya sizin oluşturduğunuz bir URL aracılığıyla uygulamanızı deneyimleyebilirler. Bu içerik sunma biçimi, uygulamanızla etkileşimi artırmanızı kolaylaştırır. </br>Instant delivery ile, kullanıcılarınızın uygulamanızın belirli özelliklerini yükleme yapmadan anında deneyimlemelerini sağlamak için Google Play Instant'ı kullanabilirsiniz.                                                                                                                                                                            | Oyunun ilk birkaç seviyesini hafif bir feature modülüne dahil eden bir oyun düşünün. Bu modülü instant-enable edeblirsiniz, böylece kullanıcılar uygulama yüklemeden bir URL bağlantısı veya "Şimdi Dene" düğmesi aracılığıyla oyunu anında deneyimleyebilirler.                                                                                                                                                                                                                                                                                                                                              | Bir feature modülü oluşturun ve instant delivery'i yapılandırın. Uygulamanız daha sonra modülü talep üzerine(on demand) indirmeyi istemek için Play Feature Delivery Library'yi kullanabilir.</br>Uygulama özelliklerinizi feature modülleri kullanarak modüler hale getirmenin yalnızca ilk adım olduğunu unutmayın. Google Play Instant'ı desteklemek için, uygulamanızın temel modülünün ve instant-enabled özelliğinin indirme boyutunun katı boyut kısıtlamalarını karşılaması gerekir. Daha fazla bilgi edinmek için [Uygulama veya oyun boyutunu küçülterek anlık deneyimleri etkinleştirin başlıklı makaleyi](docs/core-topics/google-play/google-play-instant/about-google-play-instant#reduce-size) okuyun. |

## Buildig a URI for a resource
Bir URI kullanarak bir feature modülünde depolanan bir kaynağa erişmek istiyorsanız, [Uri.Builder()](https://developer.android.com/reference/kotlin/android/net/Uri.Builder) kullanarak bir feature modülü kaynak URI'sinin nasıl oluşturulacağı aşağıda açıklanmıştır:
```kotlin
val uri = Uri.Builder()
                .scheme(ContentResolver.SCHEME_ANDROID_RESOURCE)
                .authority(context.getPackageName()) // Look up the resources in the application with its splits loaded
                .appendPath(resources.getResourceTypeName(resId))
                .appendPath(String.format("%s:%s",
                  resources.getResourcePackageName(resId), // Look up the dynamic resource in the split namespace.
                  resources.getResourceEntryName(resId)
                  ))
                .build()
```
Kaynağa giden yolun her bir parçası çalışma zamanında oluşturulur ve split APK'lar yüklendikten sonra doğru namespace'in oluşturulmasını sağlar.

URI'nin nasıl oluşturulduğuna bir örnek olarak, aşağıdaki adlara sahip bir uygulamanız ve feature modülleriniz olduğunu varsayalım:

- Uygulama paketinin adı: `com.example.my_app_package`
- Özelliğin kaynak paket adı: `com.example.my_app_package.my_dynamic_feature`

Yukarıdaki kod parçasında resId, feature modülünüzde "my_video" adlı bir ham dosya kaynağını ifade ediyorsa, yukarıdaki Uri.Builder() kodu aşağıdaki çıktıyı verecektir:

```groovy
android.resource://com.example.my_app_package/raw/com.example.my_app_package.my_dynamic_feature:my_video
```

Bu URI daha sonra uygulamanız tarafından feature modülünün kaynağına erişmek için kullanılabilir.

URI'nizdeki yolları doğrulamak için, feature modülü APK'nızı incelemek ve paket adını belirlemek üzere [APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer) kullanabilirsiniz:

![Şekil 2. Derlenmiş bir kaynak dosyasındaki paket adını incelemek için APK Analyzer'ı kullanın.](https://developer.android.com/static/images/app-bundle/apk-analyzer-package-name.png)

Şekil 2. Derlenmiş bir kaynak dosyasındaki paket adını incelemek için APK Analyzer'ı kullanın.


## Considerations for feature modules

Feature modülleri ile derleme hızını ve mühendislik hızını artırabilir ve uygulamanızın boyutunu küçültmek için uygulamanızın özelliklerinin dağıtımını kapsamlı bir şekilde özelleştirebilirsiniz. Ancak feature modüllerini kullanırken akılda tutulması gereken bazı kısıtlamalar ve istisnai durumlar vardır:

- Koşullu(conditional) veya isteğe bağlı(on-demand) dağıtım yoluyla tek bir cihaza 50 veya daha fazla feature modülü yüklemek performans sorunlarına yol açabilir. Kaldırılabilir olarak yapılandırılmayan yükleme zamanı(install-time) modülleri otomatik olarak base modüle dahil edilir ve her cihazda yalnızca bir feature modülü olarak sayılır.
- Yükleme zamanı(install-time) delivery için kaldırılabilir olarak yapılandırdığınız modül sayısını 10 veya daha az ile sınırlandırın. Aksi takdirde, uygulamanızın indirme ve yükleme süresi artabilir.
- Yalnızca Android 5.0 (API düzeyi 21) ve üzeri sürümleri çalıştıran cihazlar talep üzerine(on-demand) özellik indirmeyi ve yüklemeyi destekler. Özelliğinizi Android'in önceki sürümlerinde kullanılabilir hale getirmek için, bir feature modülü oluşturduğunuzda [Fusing] özelliğini etkinleştirin.
- [SplitCompat](https://developer.android.com/reference/com/google/android/play/core/splitcompat/SplitCompat)'ı etkinleştirin, böylece uygulamanız talep üzerine teslim edilen (on-demand delivery)indirilmiş feature modüllerine erişebilir.
- Feature modülleri, manifestolarında android:exported true olarak ayarlanmış activityleri belirtmemelidir. Bunun nedeni, başka bir uygulama activity'yi başlatmaya çalıştığında cihazın feature modülünü indirdiğine dair bir garanti olmamasıdır. Ayrıca, uygulamanız koduna ve kaynaklarına erişmeye çalışmadan önce bir özelliğin indirildiğini onaylamalıdır. Daha fazla bilgi edinmek için [Managed Installed modules](configure-on-demand-delivery.md#manage-installed-modules) bölümünü okuyun.
- Play Feature Delivery, uygulamanızı bir app bundle kullanarak yayınlamanızı gerektirdiğinden, app bundle'ın [bilinen sorunları](/docs/core-topics/android-app-bundles/about-app-bundles)ndan haberdar olduğunuzdan emin olun.

## Feature module manifest reference
Android Studio kullanarak yeni bir feature module oluştururken, IDE modülün bir feature module gibi davranması için gereken manifesto niteliklerinin çoğunu içerir. Ayrıca, bazı özellikler derleme sırasında derleme sistemi tarafından enjekte edilir, bu nedenle bunları sizin belirtmeniz veya değiştirmeniz gerekmez. Aşağıdaki tabloda feature modüller için önemli olan manifesto attributeleri açıklanmaktadır.

| Attribute                                                       | Description |
|-----------------------------------------------------------------|-------------|
| <manifest</br> ...                                              | -           |
| xmlns:dist="http://schemas.android.com/apk/distribution"        | -           |
| split="split_name"                                              | -           |
| android:isFeatureSplit="true \| false">                         | -           |
| <dist:module                                                    | -           |
| dist:instant="true \| false"                                    | -           |
| dist:title="@string/feature_name"                               | -           |
| <dist:fusing dist:include="true \| false" />  </dist:module>    | -           |
| <dist:delivery>                                                 | -           |
| <dist:install-time>                                             | -           |
| <dist:removable dist:value="true \| false" />                   | -           |
| </dist:install-time>                                            | -           |
| <dist:on-demand/>                                               | -           |
| </dist:delivery>                                                | -           |
| <application  android:hasCode="true \| false">...</application> | -           |

{: .note}
Not: Feature modülleri, manifestolarında android:exported değeri true olarak ayarlanmış aktiviteleri belirtmemelidir. Bunun nedeni, başka bir uygulama activity'yi başlatmaya çalıştığında cihazın feature modülünü indirdiğine dair bir garanti olmamasıdır. Ayrıca, uygulamanız koduna ve kaynaklarına erişmeye çalışmadan önce bir özelliğin indirildiğini onaylamalıdır.

## Additional resources
Feature modüllerini kullanma hakkında daha fazla bilgi edinmek için aşağıdaki kaynakları deneyin.

### [Blog posts](https://developer.android.com/guide/playcore/feature-delivery#blogs)
### [Videos](https://developer.android.com/guide/playcore/feature-delivery#videos)
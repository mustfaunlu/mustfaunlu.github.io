---
layout: default
title: Using the Bill of Materials
parent: Introduction
grand_parent: Jetpack Compose
nav_order: 5
---

# Using the Bill of Materials
Compose Malzeme Listesi (BOM), yalnızca BOM'un sürümünü belirterek tüm Compose kütüphane sürümlerinizi yönetmenizi sağlar. BOM'un kendisi, birlikte iyi çalışacak şekilde farklı Compose kütüphanelerinin kararlı sürümlerine bağlantılar içerir. Uygulamanızda BOM'u kullanırken, Compose kütüphane bağımlılıklarının kendilerine herhangi bir sürüm eklemeniz gerekmez. BOM sürümünü güncellediğinizde, kullandığınız tüm kütüphaneler otomatik olarak yeni sürümlerine güncellenir.

Hangi Compose kütüphane sürümlerinin belirli bir BOM sürümüyle eşlendiğini öğrenmek için [BOM ile kütüphane sürümü eşlemesine](bom-to-library-version-mapping.md) göz atın.

## Compose Compiler kütüphanesi neden BOM'a dahil edilmemiştir?
Compose Kotlin derleyici uzantısı (androidx.compose.compiler) Compose kütüphane sürümlerine bağlı değildir. Bunun yerine, Kotlin derleyici eklentisinin sürümlerine bağlanır ve Compose'un geri kalanından ayrı bir tempoda yayınlanır, bu nedenle Kotlin sürümünüzle uyumlu bir sürüm kullandığınızdan emin olun. Eklentinin her bir sürümüyle eşleşen Kotlin sürümünü [Compose to Kotlin Compatibility Map](https://developer.android.com/jetpack/androidx/releases/compose-kotlin) adresinde bulabilirsiniz.

## BOM'da belirtilenden farklı bir kütüphane sürümünü nasıl kullanabilirim?
build.gradle dependencies bölümünde, BOM platformunun import edilmesini sağlayın. Kütüphane bağımlılığı import'unda, istediğiniz sürümü belirtin. Örneğin, BOM'da hangi sürüm belirtilmiş olursa olsun Material 3'ün alfa sürümünü kullanmak istiyorsanız bağımlılıkları şu şekilde bildirebilirsiniz:
```groovy
dependencies {
    // Import the Compose BOM
    implementation platform('androidx.compose:compose-bom:2023.04.01')

    // Override Material Design 3 library version with a pre-release version
    implementation 'androidx.compose.material3:material3:1.1.0-alpha01'

    // Import other Compose libraries without version numbers
    // ..
    implementation 'androidx.compose.foundation:foundation'
}
```

{: .note }
Not: Bir Compose kütüphanesinin alfa sürümünü kullanmak için BOM'u override etmek, derlemenizi bu alfa kütüphanesinin gerekli bağımlılıklarını kullanacak şekilde güncelleyecektir (bu da alfa olabilir).

## BOM tüm Compose kitaplıklarını otomatik olarak uygulamama ekliyor mu?
Hayır. Uygulamanıza Compose kütüphanelerini eklemek ve kullanmak için her kütüphaneyi modül (uygulama düzeyinde) Gradle dosyanızda (genellikle app/build.gradle) ayrı bir bağımlılık satırı olarak bildirmeniz gerekir.

BOM'u kullanmak, uygulamanızdaki tüm Compose kitaplıklarının sürümlerinin uyumlu olmasını sağlar, ancak BOM aslında bu Compose kitaplıklarını uygulamanıza eklemez.

## Compose kitaplık sürümlerini yönetmek için neden BOM önerilen yoldur?
İleride, Compose kütüphaneleri bağımsız olarak sürümlendirilecek, bu da sürüm numaralarının kendi hızlarında artırılmaya başlanacağı anlamına geliyor. Her kütüphanenin en son kararlı sürümleri test edilmiş ve birlikte iyi çalışacakları garanti edilmiştir. Ancak, her kütüphanenin en son kararlı sürümlerini bulmak zor olabilir ve BOM bu en son sürümleri otomatik olarak kullanmanıza yardımcı olur.

## BOM'u kullanmak zorunda mıyım?
Hayır. Her bir bağımlılık sürümünü manuel olarak eklemeyi seçebilirsiniz. Ancak, en son kararlı sürümlerin tümünü aynı anda kullanmayı kolaylaştıracağı için BOM'u kullanmanızı öneririz.

## BOM sürüm katalogları ile çalışır mı?

Evet. BOM'un kendisini sürüm kataloğuna dahil edebilir ve diğer Compose kütüphane sürümlerini atlayabilirsiniz:

```groovy
[libraries]
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "androidxComposeBom" }
androidx-compose-foundation = { group = "androidx.compose.foundation", name = "foundation" }

```
Modülünüzün build.gradle dosyasında BOM'u içe aktarmayı unutmayın:

```groovy
dependencies {
    val composeBom = platform(libs.androidx.compose.bom)
    implementation(composeBom)
    androidTestImplementation(composeBom)

    // import Compose dependencies as usual
}
```

## Bir sorunu nasıl bildirebilirim veya BOM hakkında nasıl geri bildirimde bulunabilirim?
[Sorun izleyicimize](https://issuetracker.google.com/issues/new?component=612128&template=1253476) sorunlarınızı bildirebilirsiniz.
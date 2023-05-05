---
layout: default
title: Documentation
parent: Introduction
grand_parent: Jetpack Compose
nav_order: 1
---

# Get started with Jetpack Compose
Jetpack Compose, native Android kullanıcı arayüzü oluşturmak için modern bir araç setidir. Compose kullanımı hakkında en güncel bilgileri burada bulabilirsiniz.

[Overview](https://developer.android.com/jetpack/compose) (Genel Bakış): Compose geliştiricileri için mevcut tüm kaynakları görün.

[Tutorial](https://developer.android.com/jetpack/compose/tutorial): Basit bir kullanıcı arayüzü oluşturmak için Compose'u kullanmaya başlayın.

## Foundation
[Thinking in Compose:](https://developer.android.com/jetpack/compose/mental-model) Compose'un deklaratif yaklaşımının geçmişte kullanmış olabileceğiniz view tabanlı yaklaşımdan nasıl farklı olduğunu ve Compose ile çalışmak için nasıl bir zihinsel model oluşturacağınızı öğrenin.
[Managing state:](https://developer.android.com/jetpack/compose/state) Compose uygulamanızda state ayarlama ve kullanma hakkında bilgi edinin.
[Lifecycle of composables:](https://developer.android.com/jetpack/compose/lifecycle) Bir composable'ın yaşam döngüsü ve Compose'un yeniden çizilmesi gerekip gerekmediğine nasıl karar verdiği hakkında bilgi edinin.
[Modifiers: ](https://developer.android.com/jetpack/compose/modifiers)Composable'larınızı güçlendirmek veya süslemek için modifier'ları nasıl kullanacağınızı öğrenin.
[Side-effects in Compose: ](https://developer.android.com/jetpack/compose/side-effects)Side-effectleri yönetmenin en iyi yollarını öğrenin.
[Jetpack Compose Phases: ](https://developer.android.com/jetpack/compose/phases)Compose'un kullanıcı arayüzünüzü oluşturmak için geçtiği adımları ve bu bilgileri verimli kod yazmak için nasıl kullanacağınızı öğrenin
[Architectural layering: ](https://developer.android.com/jetpack/compose/layering)Jetpack Compose'u oluşturan mimari katmanlar ve tasarımına yön veren temel ilkeler hakkında bilgi edinin.
[Performance:](https://developer.android.com/jetpack/compose/performance) Uygulamanızın performansına zarar verebilecek yaygın programlama tuzaklarından nasıl kaçınacağınızı öğrenin.
[Semantics in Compose:](https://developer.android.com/jetpack/compose/semantics) Kullanıcı arayüzünüzü erişilebilirlik hizmetleri ve test framework'ü tarafından kullanılabilecek şekilde düzenleyen Semantics ağacı hakkında bilgi edinin.
[Locally scoped data with CompositionLocal:](https://developer.android.com/jetpack/compose/compositionlocal) Composition üzerinden veri aktarmak için CompositionLocal'ı nasıl kullanacağınızı öğrenin.

## Development environment
[Android Studio with Compose:](https://developer.android.com/jetpack/compose/setup) Compose'u kullanmak için geliştirme ortamınızı ayarlayın.
[Tooling for Compose:](https://developer.android.com/jetpack/compose/tooling) Android Studio'nun Compose'u destekleyen yeni özellikleri hakkında bilgi edinin.
[Kotlin for Compose: ](https://developer.android.com/jetpack/compose/kotlin)Kotlin'e özgü bazı deyimlerin Compose ile nasıl çalıştığını öğrenin.
[Developer ergonomics: ](https://developer.android.com/jetpack/compose/ergonomics)Compose'a geçişin uygulamanızın APK boyutunu ve çalışma zamanı performansını nasıl etkileyebileceğini öğrenin.
[Bill of Materials: ](https://developer.android.com/jetpack/compose/bom)Yalnızca BOM sürümünü belirterek tüm Compose bağımlılıklarınızı yönetin.

## Design
* [Layouts](https://developer.android.com/jetpack/compose/layouts): Compose'un native layout component'leri hakkında bilgi edinin ve kendi layout'unuzu nasıl tasarlayacağınızı öğrenin.
    * [Layout basics:](https://developer.android.com/jetpack/compose/layouts/basics) Basit bir uygulama kullanıcı arayüzü için yapı taşları hakkında bilgi edinin.
    * [Material Components and layouts: ](https://developer.android.com/jetpack/compose/layouts/material)Compose'daki Materyal komponentleri ve layoutları hakkında bilgi edinin.
    * [Custom layouts:](https://developer.android.com/jetpack/compose/layouts/custom) Uygulamanızın layout'unun kontrolünü nasıl ele alacağınızı ve kendinize özel bir layout'u nasıl tasarlayacağınızı öğrenin.
    * [Build adaptive layouts:](https://developer.android.com/jetpack/compose/layouts/adaptive) Farklı ekran boyutlarına, yönlere ve form faktörlerine uyum sağlayan layoutlar oluşturmak için Compose'u nasıl kullanacağınızı öğrenin.
    * [Alignment lines:](https://developer.android.com/jetpack/compose/layouts/alignment-lines) UI öğelerinizi hassas bir şekilde hizalamak ve konumlandırmak için özel hizalama çizgilerinin nasıl oluşturulacağını öğrenin.
    * [Intrinsic measurements:](https://developer.android.com/jetpack/compose/layouts/intrinsic-measurements) Compose, UI öğelerini her geçişte yalnızca bir kez ölçmenize izin verdiğinden, bu sayfada alt öğeler hakkında ölçüm yapmadan önce nasıl bilgi sorgulanacağı açıklanmaktadır.
    * [ConstraintLayout](https://developer.android.com/jetpack/compose/layouts/constraintlayout): Compose UI'nizde ConstraintLayout'u nasıl kullanacağınızı öğrenin.

* [Design Systems:](https://developer.android.com/jetpack/compose/designsystems) Bir tasarım sistemini nasıl uygulayacağınızı ve uygulamanıza nasıl tutarlı bir görünüm ve his kazandıracağınızı öğrenin.
  * [Material Design 3](https://developer.android.com/jetpack/compose/designsystems/material3): Compose'un Material Design 3 uygulaması ile Material You'yu nasıl uygulayacağınızı öğrenin.
  * [Migrating from Material 2 to Material 3](https://developer.android.com/jetpack/compose/designsystems/material2-material3): Compose'da uygulamanızı Material Design 2'den Material Design 3'e nasıl geçireceğinizi öğrenin.
  * [Material Design 2:](https://developer.android.com/jetpack/compose/designsystems/material) Compose'un Material Design 2 uygulamasını ürününüzün markasına uyacak şekilde nasıl özelleştireceğinizi öğrenin.
  * [Custom design systems:](https://developer.android.com/jetpack/compose/designsystems/custom) Compose'da özel bir tasarım sistemini nasıl uygulayacağınızı ve mevcut Material Design composable'larını buna nasıl uyarlayacağınızı öğrenin.
  * [Anatomy of a theme:](https://developer.android.com/jetpack/compose/designsystems/anatomy) MaterialTheme ve özel tasarım sistemleri tarafından kullanılan alt düzey yapılar ve API'ler hakkında bilgi edinin.

* [Lists and grids:](https://developer.android.com/jetpack/compose/lists) Veri listelerini ve gridlerini yönetmek ve görüntülemek için Compose'un bazı seçenekleri hakkında bilgi edinin.
* [Text](https://developer.android.com/jetpack/compose/text): Metni görüntülemek ve düzenlemek için Compose'un ana seçenekleri hakkında bilgi edinin.
* [Graphics](https://developer.android.com/jetpack/compose/graphics): Compose'un özel grafikler oluşturma ve bunlarla çalışma özellikleri hakkında bilgi edinin.
* [Animation](https://developer.android.com/jetpack/compose/animation): UI öğelerinizi canlandırmak için Compose'un farklı seçenekleri hakkında bilgi edinin.
* [Gestures](https://developer.android.com/jetpack/compose/gestures): Kullanıcı hareketlerini algılayan ve bunlarla etkileşime giren bir Compose kullanıcı arayüzünün nasıl oluşturulacağını öğrenin.
* [Handling user interactions:](https://developer.android.com/jetpack/compose/handling-interaction) Bileşenlerinizin kullanıcı eylemlerine nasıl yanıt vereceğini özelleştirebilmeniz için Compose'un düşük seviyeli girdileri daha yüksek seviyeli etkileşimlere nasıl dönüştürdüğünü öğrenin.

## Adopting Compose
* [Migrate existing View-based apps:](https://developer.android.com/jetpack/compose/migrate) Mevcut View tabanlı uygulamanızı Compose'a nasıl geçireceğinizi öğrenin.
  * [Migration strategy:](https://developer.android.com/jetpack/compose/migrate/strategy) Compose'u kod tabanınıza güvenli ve aşamalı bir şekilde ekleme stratejisini öğrenin.
  * [Interoperability APIs:](https://developer.android.com/jetpack/compose/migrate/interoperability-apis) Compose'u View tabanlı kullanıcı arayüzü ile birleştirmenize yardımcı olacak Compose API'leri hakkında bilgi edinin.
  * [Other considerations:](https://developer.android.com/jetpack/compose/migrate/other-considerations) View tabanlı uygulamanızı Compose'a geçirirken tema oluşturma, mimari ve test gibi diğer hususlar hakkında bilgi edinin.
* [Compose and other libraries:](https://developer.android.com/jetpack/compose/libraries) Compose içeriğinizde view tabanlı kütüphaneleri nasıl kullanacağınızı öğrenin.
* [Compose architecture:](https://developer.android.com/jetpack/compose/architecture) Compose'da tek yönlü akış modelinin nasıl uygulanacağını, event'lerin ve state holder'ların nasıl uygulanacağını ve Compose'da ViewModel ile nasıl çalışılacağını öğrenin.
* [Navigation](https://developer.android.com/jetpack/compose/navigation): Navigation component'i Compose UI'niz ile entegre etmek için NavController'ı nasıl kullanacağınızı öğrenin.
  * [Navigation for responsive UIs:](https://developer.android.com/guide/topics/large-screens/navigation-for-responsive-uis) Uygulamanızın navigasyonunu farklı ekran boyutlarına, yönlere ve form faktörlerine uyum sağlayacak şekilde nasıl tasarlayacağınızı öğrenin.
* [Resources](https://developer.android.com/jetpack/compose/resources): Compose kodunuzda uygulamanızın kaynaklarıyla nasıl çalışacağınızı öğrenin.
* [Accessibility](https://developer.android.com/jetpack/compose/accessibility): Compose kullanıcı arayüzünüzü farklı erişilebilirlik gereksinimleri olan kullanıcılar için nasıl uygun hale getireceğinizi öğrenin.
* [Testing](https://developer.android.com/jetpack/compose/testing): Compose kodunuzu test etme hakkında bilgi edinin.
  * [Testing cheat sheet:](https://developer.android.com/jetpack/compose/testing-cheatsheet) Yararlı Compose test API'lerinin hızlı bir referansı.

## Additional resources

* [Get setup](https://developer.android.com/jetpack/compose/setup)
* [Curated learning pathway](https://developer.android.com/courses/pathways/compose)
* [Compose API guidelines](https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/docs/compose-api-guidelines.md)
* [API reference](https://developer.android.com/reference/kotlin/androidx/compose)
* [Codelabs](https://goo.gle/compose-codelabs)
* [Sample apps](https://github.com/android/compose-samples)
* [Videos](https://www.youtube.com/user/androiddevelopers/search?query=%23jetpackcompose)
---
layout: default
title: Why Compose
parent: Introduction
grand_parent: Jetpack Compose
nav_order: 2
---

# Why Compose
Jetpack Compose, Android'in yerel kullanıcı arayüzü oluşturmaya yönelik modern araç setidir. Android'de UI geliştirmeyi basitleştirir ve hızlandırır, uygulamalarınızı daha az kod, güçlü araçlar ve sezgisel Kotlin API'leri ile hayata geçirir. Android kullanıcı arayüzü oluşturmayı daha hızlı ve kolay hale getirir. Compose'u oluştururken, tüm bu avantajları ilk elden deneyimleyen ve bazı çıkarımlarını bizimle paylaşan farklı ortaklarla çalıştık.

## Less Code
Daha az kod yazmak geliştirmenin tüm aşamalarını etkiler: bir yazar olarak, daha az test ve hata ayıklama ve daha az hata olasılığı ile elinizdeki soruna odaklanabilirsiniz; bir gözden geçiren veya bakımcı olarak okumak, anlamak, gözden geçirmek ve bakımını yapmak için daha az kodunuz olur.

Compose, Android View sistemini kullanmaya kıyasla daha az kodla daha fazlasını yapmanızı sağlar: Butonlar, listeler veya animasyon - ne oluşturmanız gerekiyorsa, artık yazmanız gereken daha az kod var. İşte iş ortaklarımızdan bazılarının söyledikleri:

* "Aynı Button sınıfı için [kod] 10 kat daha küçüktü." (Twitter)
* "Ayrıca, ekranlarımızın çoğunun sahip olduğu RecyclerView ile oluşturulmuş herhangi bir ekran için de önemli bir azalma var." (Monzo)
* "Uygulamamızda listeler veya animasyonlar oluşturmak için ne kadar az satır gerektiğini görmekten çok memnun olduk. Feature başına daha az kod satırı yazıyoruz, bu da müşterilerimize değer sunmaya daha fazla odaklanmamızı sağlıyor." (Cuvva)
* Yazdığınız kodun Kotlin ve XML arasında bölünmesi yerine yalnızca Kotlin'de yazılması: "Kotlin ve XML arasında gidip gelmek yerine, hepsi aynı dilde ve genellikle aynı dosyada yazıldığında kodun izini sürmek çok daha kolay" (Monzo)
* Compose ile yazılan kod basittir ve ne inşa ederseniz edin bakımı kolaydır. "Compose'un layout sistemi kavramsal olarak daha basit, bu yüzden akıl yürütmek daha kolay. Karmaşık bileşenlerin kodunu okumak da daha kolay." (Square)
## Intuitive
Compose deklaratif bir API kullanır, yani tek yapmanız gereken kullanıcı arayüzünüzü tanımlamaktır - Compose gerisini halleder. API'ler sezgiseldir - keşfetmesi ve kullanması kolaydır: "Tema katmanımız çok daha sezgisel ve okunaklı. Birden fazla katmanlı tema kaplaması aracılığıyla nitelik tanımları ve atamalardan sorumlu olan birden fazla XML dosyasına yayılan işleri tek bir Kotlin dosyasında gerçekleştirebildik." (Twitter)

Compose ile belirli bir aktivite veya fragmenta bağlı olmayan küçük, stateless bileşenler oluşturursunuz. Bu da onların yeniden kullanımını ve test edilmesini kolaylaştırır: "Kendimize, stateless, kullanımı ve bakımı kolay ve uygulaması/genişletmesi/özelleştirmesi sezgisel olan yeni bir UI bileşenleri seti sunma hedefi koyduk. Compose bu konuda bizim için gerçekten sağlam bir yanıt sağladı." (Twitter)

Compose'da state açıktır ve composable'a aktarılır. Bu şekilde, state için tek bir doğruluk kaynağı vardır, bu da onu kapsüllenmiş ve ayrıştırılmış hale getirir. Ardından, uygulama state'i değiştikçe kullanıcı arayüzünüz otomatik olarak güncellenir. "Bir şey hakkında akıl yürütürken kafanızda tutmanız gereken daha az şey ve kontrolünüz dışında olan veya yeterince anlaşılmayan daha az davranış vardır" (Cuvva)

## Accelerate development
Compose mevcut tüm kodlarınızla uyumludur: Compose kodunu View'lardan ve View'ları Compose'dan çağırabilirsiniz. Navigation, ViewModel ve Kotlin coroutines gibi en yaygın kütüphaneler Compose ile çalışır, böylece istediğiniz zaman ve istediğiniz yerde kullanmaya başlayabilirsiniz. "Birlikte çalışabilirlik, Compose'u entegre etmeye başladığımız yerdi ve bulduğumuz şey 'sadece çalıştığı' oldu. Aydınlık ve karanlık mod gibi şeyler hakkında düşünmek zorunda olmadığımızı ve tüm deneyimin inanılmaz derecede sorunsuz olduğunu gördük." (Cuvva)

Canlı önizleme gibi özelliklerle tam Android Studio desteğini kullanarak kodu daha hızlı yineleyebilir ve gönderebilirsiniz: "Android Studio'daki önizlemeler büyük bir zaman tasarrufu sağladı. Birden fazla önizleme oluşturabilmek de bize zaman kazandırıyor. Genellikle bir kullanıcı arayüzü bileşenini farklı statelerde veya farklı ayarlarla kontrol etmemiz gerekiyor - hata state'leri veya farklı bir yazı tipi boyutu vb. gibi. Birden fazla önizleme oluşturabildiğimiz için bunu kolayca kontrol edebiliyoruz." (Square)

## Powerful
Compose, Android platform API'lerine doğrudan erişim ve Materyal Tasarım, Koyu tema, animasyonlar ve daha fazlası için yerleşik destek ile güzel uygulamalar oluşturmanıza olanak tanır: "Compose ayrıca deklaratif kullanıcı arayüzünden daha fazlasını çözdü - erişilebilirlik apileri, layout, her türlü şey geliştirildi. Yapmak istediğiniz şey ile onu gerçekten yapmak arasında daha az adım var" (Square).

Compose ile animasyonlar aracılığıyla uygulamalarınıza hareket ve hayat katmak hızlı ve kolay bir şekilde uygulanabiliyor: "Compose'a animasyon eklemek o kadar kolay ki renk/boyut/yükseklik değişiklikleri gibi şeyleri canlandırmamak için çok az neden var" (Monzo), "özel bir şey gerektirmeden animasyon yapabilirsiniz - statik bir ekran göstermekten farklı değil" (Square).

İster Material Design ile ister kendi tasarım sisteminizle oluşturuyor olun, Compose size istediğiniz tasarımı uygulama esnekliği sağlar: "Material Design'ın temelden ayrılmış olması, genellikle Material'dan farklı tasarım gereksinimleri gerektiren kendi tasarım sistemimizi oluştururken bizim için gerçekten yararlı oldu." (Square)

Twitter, Square, Monzo ve Cuvva'nın Compose'u nasıl kullandığı hakkında daha fazla bilgi edinmek için derinlemesine örnek olay incelemelerine göz atın.
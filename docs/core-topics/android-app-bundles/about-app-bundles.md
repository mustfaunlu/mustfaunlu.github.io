---
layout: default
title: About App Bundles
grand_parent: Core topics
nav_order: 1
parent: Android App Bundles
---


## About Android App Bundles

{: .note}
Önemli: Ağustos 2021'den itibaren yeni uygulamaların Google Play'de Android App Bundle ile yayınlanması gerekmektedir. Artık 150 MB'tan büyük yeni uygulamalar Play Feature Delivery veya Play Asset Delivery tarafından desteklenmektedir.


Android App Bundle, uygulamanızın tüm derlenmiş kodunu ve kaynaklarını içeren ve APK oluşturma ve imzalama işlemlerini Google Play'e bırakan bir yayınlama biçimidir.

Google Play, her cihaz yapılandırması için optimize edilmiş APK'lar oluşturmak ve sunmak için app bundle'ınızı kullanır, böylece uygulamanızı çalıştırmak için yalnızca belirli bir cihaz için gereken kod ve kaynaklar indirilir. Artık farklı cihazlara yönelik desteği optimize etmek için birden fazla APK oluşturmanız, imzalamanız ve yönetmeniz gerekmiyor ve kullanıcılar daha küçük, daha optimize edilmiş indirmeler elde ediyor.

Çoğu uygulama projesi, optimize edilmiş APK'ların sunulmasını destekleyen app bundle'ları oluşturmak için fazla çaba gerektirmez. Örneğin, uygulamanızın kodunu ve kaynaklarını zaten yerleşik kurallara göre düzenliyorsanız, Android Studio'yu veya komut satırını kullanarak imzalı Android App Bundle'ları oluşturmanız ve bunları Google Play'e yüklemeniz yeterlidir. Optimize edilmiş APK hizmeti otomatik bir avantaj haline gelir.

Uygulamanızı yayınlamak için app bundle biçimini kullandığınızda, isteğe bağlı olarak uygulama projenize feature modülleri eklemenize olanak tanıyan Play Feature Delivery özelliğinden de yararlanabilirsiniz. Bu modüller, yalnızca sizin belirlediğiniz koşullara bağlı olarak uygulamanıza dahil edilen veya daha sonra çalışma zamanında Play Core Library kullanılarak indirilebilen özellikler ve kaynaklar içerir.

Uygulamalarını app bundle'larla yayınlayan oyun geliştiricileri Play Asset Delivery'yi kullanabilir: Google Play'in geliştiricilere esnek sunum yöntemleri ve yüksek performans sunan büyük miktarda oyun varlığı sunma çözümü.

Uygulamanızı neden Android App Bundle'ları kullanarak yayınlamanız gerektiğine dair genel bir bakış için aşağıdaki videoyu izleyin.

[Top 7 takeaways for Android App Bundles](https://youtu.be/st9VZuJNIbw)

### Compressed download size restriction

Android App Bundle'lar ile yayınlamak, kullanıcılarınızın uygulamanızı mümkün olan en küçük indirme ile yüklemelerine yardımcı olur ve sıkıştırılmış indirme boyutu sınırını 150 MB'a çıkarır. Yani, bir kullanıcı uygulamanızı indirdiğinde, uygulamanızı yüklemek için gereken sıkıştırılmış APK'ların toplam boyutu (örneğin, temel APK + yapılandırma APK'ları) 150 MB'den fazla olmamalıdır. İsteğe bağlı olarak bir feature modülünün (ve yapılandırma APK'larının) indirilmesi gibi sonraki tüm indirmeler de bu sıkıştırılmış indirme boyutu kısıtlamasını karşılamalıdır. Asset paketleri bu boyut sınırına katkıda bulunmaz, ancak başka boyut kısıtlamaları vardır.

Uygulama paketinizi yüklediğinizde, Play Console uygulamanızın veya isteğe bağlı özelliklerinin olası indirmelerinden herhangi birinin 150 MB'tan fazla olduğunu tespit ederse bir hata alırsınız.

Android App Bundle'ların APK genişletme (*.obb) dosyalarını desteklemediğini unutmayın. Bu nedenle, app bundle'ınızı yayınlarken bu hatayla karşılaşırsanız, sıkıştırılmış APK indirme boyutlarını azaltmak için aşağıdaki kaynaklardan birini kullanın:

- Her yapılandırma APK'sı türü için enableSplit = true ayarını yaparak tüm yapılandırma APK'larını etkinleştirdiğinizden emin olun. Bu, kullanıcıların yalnızca uygulamanızı cihazlarında çalıştırmak için ihtiyaç duydukları kodu ve kaynakları indirmelerini sağlar.


- Kullanılmayan kod ve kaynakları kaldırarak uygulamanızı küçülttüğünüzden emin olun.


- Uygulama boyutunu daha da azaltmak için en iyi uygulamaları izleyin.


- Yalnızca bazı kullanıcılarınız tarafından kullanılan özellikleri, uygulamanızın daha sonra isteğe bağlı olarak indirebileceği feature modüllerine dönüştürmeyi düşünün. Bunun uygulamanızın yeniden düzenlenmesini gerektirebileceğini unutmayın, bu nedenle önce yukarıda açıklanan diğer önerileri denediğinizden emin olun.

### Other considerations

Aşağıda, uygulamanızı Android App Bundle'lar ile oluştururken veya sunarken şu anda bilinen sorunlar yer almaktadır. Aşağıda açıklanmayan sorunlarla karşılaşırsanız, lütfen bir hata bildirin.

- Google Play Store kullanılarak yüklenmeyen ve bir veya daha fazla required split APK'sı eksik olan sideloaded uygulamaların kısmi yüklemeleri, Google onaylı tüm cihazlarda ve Android 10 (API düzeyi 29) veya daha üstünü çalıştıran cihazlarda başarısız olur. Uygulamanızı Google Play Store üzerinden indirirken Google, uygulamanın gerekli tüm bileşenlerinin yüklenmesini sağlar.\


- Kaynak tablolarını dinamik olarak değiştiren araçlar kullanırsanız, app bundle'lardan oluşturulan APK'lar beklenmedik şekilde davranabilir. Bu nedenle, bir app bundle oluştururken bu tür araçları devre dışı bırakmanız önerilir.


- Şu anda bir feature modülünün derleme yapılandırmasında temel (veya diğer) modüllerdekilerle çakışan özellikler yapılandırmak mümkündür. Örneğin, temel modülde `buildTypes.release.debuggable = true` olarak ayarlayabilir ve bir feature modülünde bunu false olarak ayarlayabilirsiniz. Bu tür çakışmalar derleme ve çalışma zamanı sorunlarına neden olabilir. Varsayılan olarak, feature modüllerinin bazı derleme yapılandırmalarını temel modülden miras aldığını unutmayın. Bu nedenle, feature modülü derleme yapılandırmanızda hangi yapılandırmaları tutmanız ve hangilerini çıkarmanız gerektiğini anladığınızdan emin olun.


## Additional resources
Android App Bundle'lar hakkında daha fazla bilgi edinmek için aşağıdaki kaynaklara başvurun.

### [Blogs](https://developer.android.com/guide/app-bundle#blogs)

### [Videos](https://developer.android.com/guide/app-bundle#videos)
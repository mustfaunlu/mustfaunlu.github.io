---
layout: default
title: About modularization
grand_parent: App architecture
nav_order: 1
parent: Modularization
---

# Guide to Android app modularization

Çoklu Gradle modülleri içeren bir proje, multi module proje olarak bilinir. Bu kılavuz, çoklu modüllü Android
uygulamaları geliştirmek için en iyi uygulamaları ve önerilen patternleri kapsar.

{: .note }
Not: Bu sayfa, önerilen uygulama mimarisi hakkında temel bir bilgi sahibi olunduğunu varsayar.

## The growing codebase problem

Kod tabanı her geçen gün büyüdükçe, ölçeklenebilirlik, okunabilirlik ve genel kod kalitesi zamanla azalır. Bu, bakımı
kolay bir yapıyı zorlamayan bakım görevlileri olmadan kod tabanının boyutunun artmasından kaynaklanır. Modülerleştirme,
kod tabanınızı bakımını kolaylaştıran bir şekilde yapılandırmanın bir yoludur ve bu sorunları önlemenize yardımcı olur.

## What is modularization?

Modülerleştirme, bir kod tabanını loosely coupled ve kendi kendine yeten parçalara ayıran bir uygulamadır. Her parça bir
modüldür. Her modül bağımsızdır ve belirli bir amaca hizmet eder. Bir sorunu daha küçük ve daha kolay çözülebilir alt
problemlere bölmek, büyük bir sistemin tasarımı ve bakımının karmaşıklığını azaltır.

![](https://developer.android.com/static/topic/modularization/images/1_sample_dep_graph.png)
Şekil 1: Örnek bir çok modüllü kod tabanının bağımlılık grafiği

## Benefits of modularization

Modülerleştirmenin pek çok faydası vardır, ancak bunların her biri bir kod tabanının sürdürülebilirliğini ve genel
kalitesini artırmaya odaklanır. Aşağıdaki tablo temel faydaları özetlemektedir.

| Benefit                   | Summary                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Reusability               | Modülerleştirme, kod paylaşımı ve aynı temelden birden fazla uygulama oluşturma fırsatları sağlar. Modüller etkin bir şekilde yapı taşlarıdır. Uygulamalar, işlevlerin ayrı modüller olarak düzenlendiği özelliklerinin bir toplamı olmalıdır. Belirli bir modülün sağladığı işlevsellik, belirli bir uygulamada etkinleştirilebilir veya etkinleştirilmeyebilir. Örneğin, bir :feature:news tam sürümün ve wear uygulamasının bir parçası olabilir ancak demo sürümün bir parçası olmayabilir. |
| Strict visibility control | Modüller, kod tabanınızın diğer bölümlerine neleri göstereceğinizi kolayca kontrol etmenizi sağlar. Modül dışında kullanılmasını önlemek için public arayüzünüz dışındaki her şeyi internal veya private olarak işaretleyebilirsiniz.                                                                                                                                                                                                                                                           |
| Customizable delivery     | Play Feature Delivery, uygulama bundle'larının gelişmiş yeteneklerini kullanarak uygulamanızın belirli özelliklerini koşullu olarak veya talep üzerine sunmanıza olanak tanır.                                                                                                                                                                                                                                                                                                                  |

Yukarıdaki faydalar yalnızca modülerleştirilmiş bir kod tabanı ile elde edilebilir. Aşağıdaki faydalar başka tekniklerle
de elde edilebilir, ancak modülerleştirme bunları daha da fazla uygulamanıza yardımcı olabilir.

| Benefit       | Summary                                                                                                                                                                                                                                                                                                                              |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Scalability   | Tightly coupled bir kod tabanında tek bir değişiklik, kodun görünüşte ilgisiz kısımlarında bir dizi değişikliği tetikleyebilir. Düzgün bir şekilde modülerleştirilmiş bir proje, speration of concerns ilkesini benimseyecek ve bu nedenle coupling'i sınırlayacaktır. Bu da katkıda bulunanları daha fazla otonomi ile güçlendirir. |
| Ownership     | Otonomi sağlamanın yanı sıra, modüller hesap verebilirliği zorlamak için de kullanılabilir. Bir modülün, kodun bakımından, hataların düzeltilmesinden, testlerin eklenmesinden ve değişikliklerin gözden geçirilmesinden sorumlu olan özel bir sahibi olabilir.                                                                      |
| Encapsulation | Kapsülleme, kodunuzun her bir parçasının diğer parçalar hakkında mümkün olan en az miktarda bilgiye sahip olması gerektiği anlamına gelir. İzole edilmiş kodun okunması ve anlaşılması daha kolaydır.                                                                                                                                |
| Testability   | Test edilebilirlik, kodunuzu test etmenin ne kadar kolay olduğunu karakterize eder. Test edilebilir bir kod, bileşenlerin izole olarak kolayca test edilebildiği bir koddur.                                                                                                                                                         |
| Build time    | Aşamalı derleme, derleme önbelleği veya paralel derleme gibi bazı Gradle işlevleri, derleme performansını artırmak için modülerlikten yararlanabilir.                                                                                                                                                                                |

## Common pitfalls

Kod tabanınızın ayrıntı düzeyi, modüllerden ne ölçüde oluştuğudur. Daha ayrıntılı bir kod tabanında daha fazla ve daha küçük modüller bulunur. Modülerleştirilmiş bir kod tabanı tasarlarken, bir ayrıntı düzeyine karar vermelisiniz. Bunu yapmak için kod tabanınızın boyutunu ve göreceli karmaşıklığını dikkate alın. Çok ince ayrıntılara inmek ek yük getirecek, çok kaba ayrıntılara inmek ise modülerleştirmenin faydalarını azaltacaktır.

Bazı yaygın görülen tuzaklar aşağıdaki gibidir:

- **Çok ince taneli:** Her modül, artan derleme karmaşıklığı ve boilerplate kodu şeklinde belirli bir miktar ek yük getirir. Karmaşık bir derleme konfigürasyonu, konfigürasyonların modüller arasında tutarlı olmasını zorlaştırır. Çok fazla boilerplate kod, bakımı zor olan hantal bir kod tabanına neden olur. Ek yük, ölçeklenebilirlik iyileştirmelerini engelliyorsa bazı modülleri birleştirmeyi düşünmelisiniz.


- **Çok kaba taneli:** Tersine, modülleriniz çok büyüyorsa, başka bir monolit ile sonuçlanabilir ve modülerliğin sunduğu faydaları kaçırabilirsiniz. Örneğin, küçük bir projede veri katmanını tek bir modülün içine koymak sorun olmayabilir. Ancak proje büyüdükçe depoları ve veri kaynaklarını bağımsız modüllere ayırmak gerekebilir.


- **Çok karmaşık:** Projenizi modüler hale getirmek her zaman mantıklı değildir. Baskın bir faktör kod tabanının boyutudur. Projenizin belirli bir eşiğin ötesinde büyümesini beklemiyorsanız, ölçeklenebilirlik ve derleme süresi kazanımları geçerli olmayacaktır.


## Is modularization the right technique for me?
Yeniden kullanılabilirlik, sıkı görünürlük kontrolü veya Play Feature Delivery'yi kullanmanın avantajlarına ihtiyacınız varsa, modülerleştirme sizin için bir gerekliliktir. İhtiyacınız yoksa, ancak yine de gelişmiş ölçeklenebilirlik, sahiplik, kapsülleme veya derleme sürelerinden yararlanmak istiyorsanız, modülerleştirme düşünmeye değer bir şeydir.


## Samples
- [Now in Android](https://github.com/android/nowinandroid)  Modülerleştirme özelliğine sahip tamamen işlevsel Android uygulaması örneği.
- [Multi module architecture sample](https://github.com/android/architecture-samples/tree/multimodule)
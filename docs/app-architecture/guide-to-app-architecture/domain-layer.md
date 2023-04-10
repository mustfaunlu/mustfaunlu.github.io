---
layout: default
title: Domain layer
grand_parent: App architecture
nav_order: 3
parent: Guide to app architecture
---

## Domain layer
Domain katmanı, UI katmanı ile data katmanı arasında yer alan isteğe bağlı bir katmandır.
![Domain layer](/assets/images/domain-layer.png)

Domain katmanı, karmaşık business logic'in veya birden fazla ViewModel tarafından yeniden kullanılan basit business logic'in enkapsüle edilmesinden sorumludur. Bu katman isteğe bağlıdır çünkü tüm uygulamalar bu gereksinimlere sahip olmayacaktır. Yalnızca gerektiğinde kullanmalısınız - örneğin, karmaşıklığı handle etmek veya yeniden kullanılabilirliği kolaylaştırmak için.

Domain katmanı aşağıdaki faydaları sağlar:

* Kod tekrarını önler.
* Domain katmanı sınıflarını kullanan sınıflarda okunabilirliği artırır.
* Uygulamanın test edilebilirliğini artırır.
* Sorumlulukları bölmenize izin vererek büyük sınıfları önler.

Bu sınıfları basit ve hafif tutmak için, her kullanım senaryosu yalnızca tek bir fonksiyonellik üzerinde sorumluluk sahibi olmalı ve mutable veri içermemelidir. Bunun yerine mutable verileri UI veya data katmanlarınızda ele almalısınız.

<mark style="background-color:lightblue">Not: Bu sayfadaki öneriler ve best practiceler, ölçeklenmelerini sağlamak, kaliteyi ve sağlamlığı artırmak ve test edilmelerini kolaylaştırmak için geniş bir uygulama yelpazesine uygulanabilir. Ancak, bunları kılavuz olarak ele almalı ve gerektiğinde gereksinimlerinize göre uyarlamalısınız.


[Architecture: The Domain Layer - MAD Skills](https://youtu.be/gIhjCh3U88I)

### Naming conventions in this guide
Bu kılavuzda, use case'ler sorumlu oldukları tek bir eylemden sonra adlandırılır. Kurallar aşağıdaki gibidir:

şimdiki zamanda fiil + isim/ne (isteğe bağlı) + UseCase.

Örneğin: FormatDateUseCase, LogOutUserUseCase, GetLatestNewsWithAuthorsUseCase veya MakeLoginRequestUseCase.

### Dependencies
Tipik bir uygulama mimarisinde, use case sınıfları UI katmanındaki ViewModel'ler ile data katmanındaki repository'ler arasında yer alır. Bu, use case sınıflarının genellikle repository sınıflarına bağlı olduğu ve UI katmanı ile repository'lerin yaptığı gibi iletişim kurduğu anlamına gelir - ya callback'ler (Java için) ya da coroutine'ler (Kotlin için) kullanarak. Bu konuda daha fazla bilgi edinmek için [data layer](data-layer/about-the-data-layer) sayfasına bakın.

Örneğin, uygulamanızda bir haber repository'sinden ve bir yazar repository'sinden veri alan ve bunları birleştiren bir use case sınıfınız olabilir:
```kotlin
class GetLatestNewsWithAuthorsUseCase(
  private val newsRepository: NewsRepository,
  private val authorsRepository: AuthorsRepository
) { /* ... */ }
```
Use case'ler yeniden kullanılabilir lojik içerdiğinden, diğer use case'ler tarafından da kullanılabilirler. Domain katmanında birden fazla use case seviyesi olması normaldir. Örneğin, aşağıdaki örnekte tanımlanan use case, UI katmanındaki birden fazla sınıfın ekranda uygun mesajı görüntülemek için saat dilimlerine güvenmesi durumunda FormatDateUseCase use case'ini kullanabilir:
```kotlin
class GetLatestNewsWithAuthorsUseCase(
  private val newsRepository: NewsRepository,
  private val authorsRepository: AuthorsRepository,
  private val formatDateUseCase: FormatDateUseCase
) { /* ... */ }
```
[Example dependency graph for a use case that depends on other use cases](/assets/images/domain-layer-img1.png)

### Call use cases in Kotlin
Kotlin'de, invoke() fonksiyonunu operatör modifier ile tanımlayarak use case sınıf instance'larını fonksiyon olarak çağrılabilir hale getirebilirsiniz. Aşağıdaki örneğe bakın:
```kotlin
class FormatDateUseCase(userRepository: UserRepository) {

    private val formatter = SimpleDateFormat(
        userRepository.getPreferredDateFormat(),
        userRepository.getPreferredLocale()
    )

    operator fun invoke(date: Date): String {
        return formatter.format(date)
    }
}
```
Bu örnekte, FormatDateUseCase'deki invoke() metodu, sınıfın instance'larını fonksiyonlarmış gibi çağırmanıza olanak tanır. invoke() metodu belirli bir imza ile sınırlandırılmamıştır; herhangi bir sayıda parametre alabilir ve herhangi bir tip döndürebilir. Ayrıca invoke() metodunu sınıfınızda farklı imzalarla overload edebilirsiniz. Yukarıdaki örnekteki use case'i aşağıdaki gibi çağırırsınız:
```kotlin
class MyViewModel(formatDateUseCase: FormatDateUseCase) : ViewModel() {
    init {
        val today = Calendar.getInstance()
        val todaysDate = formatDateUseCase(today)
        /* ... */
    }
}
```
invoke() operatörü hakkında daha fazla bilgi edinmek için [Kotlin dokümanları](https://kotlinlang.org/docs/operator-overloading.html#invoke-operator)na bakın.

### Lifecycle
Use Case'lerin kendi yaşam döngüleri yoktur. Bunun yerine, onları kullanan sınıfa göre scopelandırılırlar. Bu, use case'leri UI katmanındaki sınıflardan, hizmetlerden veya Application sınıfının kendisinden çağırabileceğiniz anlamına gelir. Use case'ler mutable veriler içermemesi gerektiğinden, bir use case sınıfını bağımlılık olarak her ilettiğinizde bu sınıfın yeni bir instance'ını oluşturmanız gerekir.

### Threading
Domain katmanındaki Use Case'ler main-safe olmalıdır; başka bir deyişle, main thread'den çağrılmaları güvenli olmalıdır. Use case sınıfları uzun süren bloklama işlemleri gerçekleştiriyorsa, bu lojiği uygun iş parçacığına taşımaktan sorumludurlar. Ancak bunu yapmadan önce, bu engelleme işlemlerinin hiyerarşinin diğer katmanlarına yerleştirilmesinin daha iyi olup olmayacağını kontrol edin. Tipik olarak, karmaşık hesaplamalar yeniden kullanılabilirliği veya önbelleğe almayı teşvik etmek için data katmanında gerçekleşir. Örneğin, büyük bir liste üzerindeki yoğun kaynak gerektiren bir işlem, sonucun uygulamanın birden fazla ekranında yeniden kullanılabilmesi için önbelleğe alınması gerekiyorsa, domain katmanından ziyade data katmanına daha iyi yerleştirilir.

Aşağıdaki örnekte, çalışmasını bir background thread üzerinde gerçekleştiren bir use case gösterilmektedir:
```kotlin
class MyUseCase(
    private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {

    suspend operator fun invoke(...) = withContext(defaultDispatcher) {
        // Long-running blocking operations happen on a background thread.
    }
}
```

### Common Tasks
Bu bölümde, yaygın domain katmanı görevlerinin nasıl gerçekleştirileceği açıklanmaktadır.

Reusable simple business logic
UI katmanında bulunan tekrarlanabilir business logic'i bir use case sınıfında encapsulate etmelisiniz. Bu, logic'in kullanıldığı her yerde herhangi bir değişikliği gerçekleştirmeyi kolaylaştırır. Ayrıca lojiği izole bir şekilde test etmenize de olanak tanır.

Daha önce açıklanan FormatDateUseCase örneğini düşünün. Tarih biçimlendirmeyle ilgili iş gereksinimleriniz gelecekte değişirse, kodu yalnızca tek bir merkezi yerde değiştirmeniz gerekir.

<mark style="background-color:lightblue">Not: Bazı durumlarda, use case'lerde bulunabilecek lojik, bunun yerine Util sınıflarındaki statik metotların bir parçası olabilir. Ancak, Util sınıflarını bulmak genellikle zor olduğundan ve işlevlerini keşfetmek zor olduğundan, ikincisi önerilmez. Ayrıca, use case'ler temel sınıflarda thread ve error handling gibi ortak işlevleri paylaşabilir ve bu da ölçek olarak daha büyük ekiplere fayda sağlayabilir.

### Combine repositories
Bir haber uygulamasında, sırasıyla haber ve yazar veri işlemlerini gerçekleştiren NewsRepository ve AuthorsRepository sınıflarına sahip olabilirsiniz. NewsRepository'nin sunduğu Article sınıfı yalnızca yazarın adını içerir, ancak ekranda yazar hakkında daha fazla bilgi görüntülemek istersiniz. Yazar bilgileri AuthorsRepository'den elde edilebilir.
![Combine repositeries](/assets/images/domain-layer-img2.png)
Logic birden fazla repository içerdiğinden ve karmaşık hale gelebileceğinden, logic'i ViewModel'den soyutlamak ve daha okunabilir hale getirmek için bir GetLatestNewsWithAuthorsUseCase sınıfı oluşturursunuz. Bu aynı zamanda logic'in tek başına test edilmesini ve uygulamanın farklı bölümlerinde yeniden kullanılabilir olmasını kolaylaştırır.
```kotlin
/**
 * This use case fetches the latest news and the associated author.
 */
class GetLatestNewsWithAuthorsUseCase(
  private val newsRepository: NewsRepository,
  private val authorsRepository: AuthorsRepository,
  private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    suspend operator fun invoke(): List<ArticleWithAuthor> =
        withContext(defaultDispatcher) {
            val news = newsRepository.fetchLatestNews()
            val result: MutableList<ArticleWithAuthor> = mutableListOf()
            // This is not parallelized, the use case is linearly slow.
            for (article in news) {
                // The repository exposes suspend functions
                val author = authorsRepository.getAuthor(article.authorId)
                result.add(ArticleWithAuthor(article, author))
            }
            result
        }
}
```
Logic, haber listesindeki tüm öğeleri mapler; bu nedenle data katmanı main-safe olsa da, bu iş main thread'i bloke etmemelidir çünkü kaç öğeyi process edeceğini bilemezsiniz. Bu nedenle use case, varsayılan dispatcher'ı kullanarak işi bir background thread'e taşır.

<mark style="background-color:lightblue">
Not: Room kütüphanesi, bir veritabanındaki farklı entityler arasındaki ilişkileri sorgulamanızı sağlar. Veritabanı source of truth ise, tüm bu işi sizin için yapan bir query oluşturabilirsiniz. Bu durumda, bir use case yerine NewsWithAuthorsRepository gibi bir repository sınıfı oluşturmak daha iyidir.



### Other consumers
UI katmanının yanı sıra, domain katmanı servisler ve Application sınıfı gibi diğer sınıflar tarafından da yeniden kullanılabilir. Ayrıca, TV veya Wear gibi diğer platformlar mobil uygulama ile kod tabanını paylaşıyorsa, UI katmanları da domain katmanının yukarıda bahsedilen tüm avantajlarını elde etmek için use case'leri yeniden kullanabilir.

### Data layer access restriction
Domain katmanını implemente ederken göz önünde bulundurmanız gereken bir diğer husus da UI katmanından data katmanına doğrudan erişime izin vermeniz ya da her şeyi domain katmanı üzerinden yapmaya zorlamanız gerekip gerekmediğidir.
![Dependency graph showing UI Layer being denied access to the data layer.](/assets/images/domain-layer-img3.png)

Bu kısıtlamayı yapmanın bir avantajı, örneğin data katmanına her erişim isteğinde analitik loglama yapıyorsanız, UI'nizin domain katmanı logic'ini bypass etmesini engellemesidir.

Bununla birlikte, potansiyel olarak önemli dezavantajı, sizi data katmanına basit fonksiyon çağrıları olsa bile use case'ler eklemeye zorlamasıdır, bu da çok az fayda için karmaşıklık yaratabilir.

İyi bir yaklaşım, use case'leri yalnızca gerektiğinde eklemektir. UI katmanınızın verilere neredeyse yalnızca use case'ler aracılığıyla eriştiğini fark ederseniz, verilere yalnızca bu şekilde erişmek mantıklı olabilir.

Nihayetinde data katmanına erişimi kısıtlama kararı, kod tabanınıza ve katı kuralları mı yoksa daha esnek bir yaklaşımı mı tercih ettiğinize bağlıdır.
Testing
Domain katmanını test ederken genel test kılavuzu geçerlidir. Diğer UI testleri için geliştiriciler genellikle sahte repository'ler kullanır ve domain katmanını test ederken de sahte repository'ler kullanmak iyi bir yöntemdir.

### [Sample](https://developer.android.com/topic/architecture/domain-layer#samples)
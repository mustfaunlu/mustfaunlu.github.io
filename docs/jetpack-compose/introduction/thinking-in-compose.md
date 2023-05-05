---
layout: default
title: Thinking in Compose
parent: Introduction
grand_parent: Jetpack Compose
nav_order: 4
---

# Thinking in Compose

[Intuitive: Thinking in Compose - MAD Skills](https://youtu.be/4zf30a34OOA)

Jetpack Compose, Android için modern bir deklaratif UI Toolkit'tir. Compose, frontend view'ları zorunlu olarak değiştirmeden uygulama kullanıcı arayüzünüzü oluşturmanıza olanak tanıyan deklaratif bir API sağlayarak uygulama kullanıcı arayüzünüzü yazmayı ve korumayı kolaylaştırır. Bu terminoloji biraz açıklama gerektiriyor, ancak sonuçları uygulama tasarımınız için önemlidir.

## The declarative programming paradigm

Tarihsel olarak, bir Android view hiyerarşisi UI widget'larından oluşan bir ağaç olarak temsil edilebilir. Kullanıcı etkileşimleri gibi nedenlerle uygulamanın state'i değiştikçe, UI hiyerarşisinin mevcut verileri gösterecek şekilde güncellenmesi gerekir. UI'yi güncellemenin en yaygın yolu, findViewById() gibi fonksiyonları kullanarak ağaçta dolaşmak ve button.setText(String), container.addChild(View) veya img.setImageBitmap(Bitmap) gibi yöntemleri çağırarak nodları değiştirmektir. Bu yöntemler widget'ın dahili state'ini değiştirir.

View'ları manuel olarak değiştirmek hata olasılığını artırır. Bir veri parçası birden fazla yerde işleniyorsa, onu gösteren view'lardan birini güncellemeyi unutmak kolaydır. Ayrıca, iki güncelleme beklenmedik bir şekilde çakıştığında illegal stateler oluşturmak da kolaydır. Örneğin, bir güncelleme, kullanıcı arayüzünden yeni kaldırılmış bir node'un değerini ayarlamaya çalışabilir. Genel olarak, yazılım bakım karmaşıklığı güncelleme gerektiren view sayısı ile birlikte artar.

Son birkaç yıldır tüm endüstri, kullanıcı arayüzlerinin oluşturulması ve güncellenmesiyle ilgili mühendisliği büyük ölçüde basitleştiren deklaratif bir kullanıcı arayüzü modeline geçmeye başlamıştır. Bu teknik, kavramsal olarak tüm ekranı sıfırdan yeniden oluşturarak ve ardından yalnızca gerekli değişiklikleri uygulayarak çalışır. Bu yaklaşım, stateful bir view hiyerarşisini manuel olarak güncellemenin karmaşıklığını önler. Compose, deklaratif bir kullanıcı arayüzü framework'üdür.

Tüm ekranı yeniden oluşturmanın zorluklarından biri, zaman, bilgi işlem gücü ve pil kullanımı açısından potansiyel olarak pahalı olmasıdır. Bu maliyeti azaltmak için Compose, herhangi bir zamanda kullanıcı arayüzünün hangi bölümlerinin yeniden çizilmesi gerektiğini akıllıca seçer. Bunun, [Recomposition](#recomposition)'da tartışıldığı gibi, UI bileşenlerinizi nasıl tasarladığınızla ilgili bazı etkileri vardır.

## A simple composable function
Compose kullanarak, veri alan ve kullanıcı arayüzü öğeleri emit eden bir dizi composable fonksiyon tanımlayarak kullanıcı arayüzünüzü oluşturabilirsiniz. Basit bir örnek olarak, bir String alan ve bir selamlama mesajı görüntüleyen bir Text widget'ı emit eden bir Greeting widget'ı verilebilir.

![A simple composable function](https://developer.android.com/static/images/jetpack/compose/mmodel-simple.png)

Bu fonksiyon ile ilgili birkaç kayda değer şey:

* Fonksiyona @Composable notasyonu eklenmiştir. Tüm Composable fonksiyonları bu annotation'a sahip olmalıdır; bu annotation Compose derleyicisine bu fonksiyonun verileri UI'ye dönüştürmek için tasarlandığını bildirir.

* Fonksiyon veri alır. Composable fonksiyonlar, uygulama lojiğinin kullanıcı arayüzünü tanımlamasına olanak tanıyan parametreleri kabul edebilir. Bu durumda, widget'ımız kullanıcıyı adıyla selamlayabilmek için bir String kabul eder.

* Fonksiyon, kullanıcı arayüzünde metin görüntüler. Bunu, aslında metin UI öğesini oluşturan Text() composable fonksiyonunu çağırarak yapar. Composable fonksiyonlar, diğer composable fonksiyonları çağırarak UI hiyerarşisini emit eder.

* Fonksiyon hiçbir şey döndürmez. UI emit eden Compose fonksiyonlarının herhangi bir şey döndürmesi gerekmez, çünkü UI widget'ları oluşturmak yerine istenen ekran state'ini tanımlarlar.

* Bu fonksiyon hızlı, [idempotent](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning) ve side-effectsizdir.

  * Fonksiyon aynı argümanla birden fazla kez çağrıldığında aynı şekilde davranır ve global değişkenler veya random() çağrıları gibi diğer değerleri kullanmaz.
  * Fonksiyon, özellikleri veya global değişkenleri değiştirmek gibi herhangi bir side-effect olmaksızın kullanıcı arayüzünü tanımlar.

Genel olarak, [Recomposition](#recomposition) bölümünde tartışılan nedenlerden dolayı tüm composable fonksiyonlar bu özelliklerle yazılmalıdır.

## The declarative paradigm shift
Birçok emperatif nesne yönelimli kullanıcı arabirimi araç setinde, kullanıcı arabirimini bir widget ağacını instantiate ederek başlatırsınız. Bunu genellikle bir XML layout dosyasını inflate ederek yaparsınız. Her widget kendi iç state'ini korur ve uygulama mantığının widget ile etkileşime girmesini sağlayan getter ve setter yöntemlerini ortaya çıkarır.

Compose'un deklaratif yaklaşımında, widget'lar nispeten statelessdir ve setter veya getter fonksiyonlarını açığa çıkarmazlar. Aslında, widget'lar nesne olarak gösterilmez. Kullanıcı arayüzünü farklı argümanlarla aynı composable fonksiyonunu çağırarak güncellersiniz. Bu, App Architecture Guide'da açıklandığı gibi ViewModel gibi mimari modellere durum sağlamayı kolaylaştırır. Ardından, composable'larınız, gözlemlenebilir veriler her güncellendiğinde mevcut uygulama state'ini bir UI'ye dönüştürmekten sorumludur.

![](https://developer.android.com/static/images/jetpack/compose/mmodel-flow-data.png)
Şekil 2. Uygulama lojiği, top-level composable fonksiyona veri sağlar. Bu fonksiyon, diğer composable'ları çağırarak kullanıcı arayüzünü tanımlamak için verileri kullanır ve uygun verileri bu composable'lara ve hiyerarşide aşağıya doğru iletir.

Kullanıcı kullanıcı arayüzü ile etkileşime girdiğinde, kullanıcı arayüzü onClick gibi event'leri tetikler. Bu event'ler uygulama mantığını bilgilendirmeli ve bu da uygulamanın state'ini değiştirebilmelidir. State değiştiğinde, composable fonksiyonlar yeni verilerle tekrar çağrılır. Bu, UI öğelerinin yeniden çizilmesine neden olur - bu işleme recomposition denir.
![](https://developer.android.com/static/images/jetpack/compose/mmodel-flow-events.png)
Şekil 3. Kullanıcı bir UI öğesiyle etkileşime girerek bir event'in tetiklenmesine neden oldu. Uygulama mantığı olaya yanıt verir, ardından composable fonksiyonlar gerekirse yeni parametrelerle otomatik olarak tekrar çağrılır.

## Dynamic content
Composable fonksiyonlar XML yerine Kotlin ile yazıldıkları için diğer Kotlin kodları kadar dinamik olabilirler. Örneğin, bir kullanıcı listesini selamlayan bir kullanıcı arayüzü oluşturmak istediğinizi varsayalım:
```kotlin
@Composable
fun Greeting(names: List<String>) {
    for (name in names) {
        Text("Hello $name")
    }
}
```

Bu fonksiyon bir isim listesi alır ve her kullanıcı için bir karşılama mesajı oluşturur. Composable fonksiyonlar oldukça sofistike olabilir. Belirli bir UI öğesini göstermek isteyip istemediğinize karar vermek için if deyimlerini kullanabilirsiniz. Döngüler kullanabilirsiniz. Yardımcı fonksiyonları çağırabilirsiniz. Temel dilin tüm esnekliğine sahip olursunuz. Bu güç ve esneklik Jetpack Compose'un en önemli avantajlarından biridir.

## Recomposition
Emperatif bir UI modelinde, bir widget'ı değiştirmek için, widget'ın dahili state'ini değiştirmek üzere bir setter çağırırsınız. Compose'da, composable fonksiyonu yeni verilerle tekrar çağırırsınız. Bunu yapmak, fonksiyonun yeniden oluşturulmasına neden olur - fonksiyon tarafından emit edilen widget'lar, gerekirse yeni verilerle yeniden çizilir. Compose framework akıllı bir şekilde yalnızca değişen bileşenleri yeniden oluşturabilir.

Örneğin, bir buton görüntüleyen bu composable fonksiyonu düşünün:

```kotlin
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}
```

Butona her tıklandığında, caller clicks değerini günceller. Compose, yeni değeri göstermek için lambda'yı Text işleviyle birlikte yeniden çağırır; bu işleme recomposition denir. Değere bağlı olmayan diğer fonksiyonlar recompose edilmez.

Daha önce de belirttiğimiz gibi, tüm UI ağacını yeniden oluşturmak hesaplama açısından pahalı olabilir ve bu da hesaplama gücünü ve pil ömrünü tüketir. Compose, bu akıllı recomposition ile bu sorunu çözer.

Recomposition, girdiler değiştiğinde composable fonksiyonlarınızı tekrar çağırma işlemidir. Bu, fonksiyonun girdileri değiştiğinde gerçekleşir. Compose yeni girdilere göre recomposition yaparken, yalnızca değişmiş olabilecek fonksiyonları veya lambdaları çağırır ve diğerlerini atlar. Değişen parametrelere sahip olmayan tüm fonksiyonları veya lambdaları atlayarak, Compose verimli bir şekilde yeniden oluşturabilir.

Bir fonksiyonun recomposition'ı atlanabileceğinden, composable fonksiyonların çalıştırılmasından kaynaklanan side-effects'e asla güvenmeyin. Bunu yaparsanız, kullanıcılar uygulamanızda garip ve öngörülemeyen davranışlarla karşılaşabilir. Side-effect, uygulamanızın geri kalanı tarafından görülebilen herhangi bir değişikliktir. Örneğin, bu eylemlerin hepsi tehlikeli side-effectlerdir:

* Shared bir nesnenin property'sine yazma
* ViewModel'de bir gözlemlenebiliri güncelleme
* Sharedprefleri güncelleme

Composable fonksiyonlar, bir animasyon işlenirken olduğu gibi her karede yeniden çalıştırılabilir. Animasyonlar sırasında sıkıntıyı önlemek için composable fonksiyonlar hızlı olmalıdır. Shared preferences'dan okuma gibi pahalı işlemler yapmanız gerekiyorsa, bunu bir background coroutine içinde yapın ve değer sonucunu parametre olarak composable fonksiyona aktarın.

Örnek olarak, bu kod SharedPreferences'daki bir değeri güncellemek için bir composable oluşturur. Composable, shared preferences'ın kendisini okumamalı veya yazmamalıdır. Bunun yerine, bu kod okuma ve yazma işlemlerini bir background coroutine içinde bir ViewModel'e taşır. Uygulama mantığı, güncellemeyi tetiklemek için geçerli değeri bir callback ile iletir.

```kotlin
@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    onValueChanged: (Boolean) -> Unit
) {
    Row {
        Text(text)
        Checkbox(checked = value, onCheckedChange = onValueChanged)
    }
}

```

Bu belgede Compose kullanırken dikkat etmeniz gereken bazı hususlar ele alınmaktadır:

* Composable fonksiyonlar herhangi bir sırada çalıştırılabilir.
* Composable fonksiyonlar paralel olarak çalışabilir.
* Recomposition mümkün olduğunca çok sayıda composable fonksiyonu ve lambdayı atlar.
* Recomposition optimistiktir ve iptal edilebilir.
* Composable bir fonksiyon, bir animasyonun her karesi gibi oldukça sık çalıştırılabilir.
  
İlerleyen bölümlerde, recomposition'ı desteklemek için composable fonksiyonların nasıl oluşturulacağı ele alınacaktır. Her durumda, best practice composable fonksiyonlarınızı hızlı, idempotent ve side-effect-free tutmaktır.

### Composable functions can execute in any order
Composable bir fonksiyonun koduna bakarsanız, kodun göründüğü sırada çalıştırıldığını varsayabilirsiniz. Ancak bu her zaman doğru değildir. Composable bir fonksiyon diğer Composable fonksiyonlara çağrılar içeriyorsa, bu fonksiyonlar herhangi bir sırada çalışabilir. Compose, bazı UI öğelerinin diğerlerinden daha yüksek önceliğe sahip olduğunu kabul etme ve önce onları çizme seçeneğine sahiptir.

Örneğin, bir tab layout'ta üç ekran çizmek için aşağıdaki gibi bir kodunuz olduğunu varsayalım:
```kotlin
@Composable
fun ButtonRow() {
    MyFancyNavigation {
        StartScreen()
        MiddleScreen()
        EndScreen()
    }
}
```
StartScreen, MiddleScreen ve EndScreen çağrıları herhangi bir sırada gerçekleşebilir. Bu, örneğin StartScreen() fonksiyonunun global bir değişkeni ayarlamasını (bir side-effect) ve MiddleScreen() fonksiyonunun bu değişiklikten yararlanmasını sağlayamayacağınız anlamına gelir. Bunun yerine, bu fonksiyonların her birinin kendi içinde bağımsız olması gerekir.

### Composable functions can run in parallel
Compose, composable fonksiyonları paralel olarak çalıştırarak recomposition'ı optimize edebilir. Bu, Compose'un birden fazla çekirdekten yararlanmasını ve ekranda olmayan composable fonksiyonları daha düşük bir öncelikte çalıştırmasını sağlar.

Bu optimizasyon, composable bir fonksiyonun bir background thread havuzu içinde çalışabileceği anlamına gelir. Composable bir fonksiyon ViewModel üzerinde bir fonksiyon çağırırsa, Compose bu fonksiyonu aynı anda birkaç thread'den çağırabilir.

Uygulamanızın doğru şekilde çalıştığından emin olmak için, tüm composable fonksiyonların hiçbir side-effect'i olmamalıdır. Bunun yerine, side-effectleri her zaman UI thread üzerinde çalışan onClick gibi callback'lerden tetikleyin.

Composable bir fonksiyon çağırıldığında, çağırma işlemi çağırandan farklı bir thread üzerinde gerçekleşebilir. Bu da, hem bu tür kodlar thread-safe olmadığından hem de composable lambda'nın izin verilmeyen bir side-effect'i olduğundan, composable lambda'daki değişkenleri değiştiren kodlardan kaçınılması gerektiği anlamına gelir.

İşte bir listeyi ve sayısını gösteren bir composable örneği:

```kotlin
@Composable
fun ListComposable(myList: List<String>) {
    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
            }
        }
        Text("Count: ${myList.size}")
    }
}
```

Bu kod side-effect'sizdir ve input listesini UI'ye dönüştürür. Bu, küçük bir listeyi görüntülemek için harika bir koddur. Ancak, fonksiyon lokal bir değişkene yazıyorsa, bu kod thread-safe veya doğru olmayacaktır:

```kotlin
@Composable
@Deprecated("Example with bug")
fun ListWithBug(myList: List<String>) {
    var items = 0

    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
                items++ // Avoid! Side-effect of the column recomposing.
            }
        }
        Text("Count: $items")
    }
}
```

Bu örnekte, öğeler her yeniden düzenlemede değiştirilir. Bu, bir animasyonun her karesi veya liste güncellendiğinde olabilir. Her iki durumda da kullanıcı arayüzü yanlış sayıyı gösterecektir. Bu nedenle, Compose'da bu gibi yazmalar desteklenmez; bu yazmaları yasaklayarak, framework'ün composable lambdaları çalıştırmak için thread'leri değiştirmesine izin veririz.



### Recomposition skips as much as possible
Kullanıcı arayüzünüzün bazı bölümleri geçersiz olduğunda, Compose yalnızca güncellenmesi gereken bölümleri yeniden oluşturmak için elinden geleni yapar. Bu, UI ağacında üstündeki veya altındaki composable'ların hiçbirini çalıştırmadan tek bir Button'ın composable'ını yeniden çalıştırmayı atlayabileceği anlamına gelir.

Her composable fonksiyon ve lambda kendi başına yeniden oluşturulabilir. İşte bir liste oluştururken yeniden oluşturmanın(recomposition) bazı öğeleri nasıl atlayabileceğini gösteren bir örnek:

```kotlin
/**
 * Kullanıcının tıklayabileceği isimlerin listesini bir başlıkla görüntüleme
 */
@Composable
fun NamePicker(
    header: String,
    names: List<String>,
    onNameClicked: (String) -> Unit
) {
    Column {
        // bu, [header] değiştiğinde yeniden oluşturacak, ancak [names] değiştiğinde oluşturmayacaktır
        Text(header, style = MaterialTheme.typography.bodyLarge)
        Divider()

        // LazyColumn,  RecyclerView'in Compose versiyonudur.
        // items() fonksiyonuna geçirilen lambda bir RecyclerView.ViewHolder'a benzer.
        LazyColumn {
            items(names) { name ->
                // Bir öğenin [name]'i güncellendiğinde, o öğenin adaptörü
                // yeniden oluşturacaktır. Bu, [header] değiştiğinde yeniden oluşturulmayacaktır
                NamePickerItem(name, onNameClicked)
            }
        }
    }
}

/**
 * Kullanıcının tıklayabileceği tek bir ad görüntüleyin.
 */
@Composable
private fun NamePickerItem(name: String, onClicked: (String) -> Unit) {
    Text(name, Modifier.clickable(onClick = { onClicked(name) }))
}
```
Bu scope'ların her biri, recomposition sırasında çalıştırılacak tek şey olabilir. Compose, başlık değiştiğinde ebeveynlerinden hiçbirini çalıştırmadan Column lambda'sına atlayabilir. Ve Column'u çalıştırırken Compose, isimler değişmediyse LazyColumn'un öğelerini atlamayı seçebilir.

Yine, tüm composable fonksiyonların veya lambdaların çalıştırılması side-effect içermemelidir. Bir side-effect gerçekleştirmeniz gerektiğinde, bunu bir callback'ten tetikleyin.

### Recomposition is optimistic
Compose bir composable'ın parametrelerinin değişmiş olabileceğini düşündüğünde recomposition başlar. Recomposition iyimserdir, yani Compose parametreler tekrar değişmeden önce recomposition işleminin bitmesini bekler. Recomposition bitmeden önce bir parametre değişirse, Compose recomposition'ı iptal edebilir ve yeni parametre ile yeniden başlatabilir.

Recomposition iptal edildiğinde, Compose UI ağacını recomposition'dan atar. Görüntülenen kullanıcı arayüzüne bağlı olan herhangi bir side-effectiniz varsa, composition iptal edilse bile side-effect uygulanacaktır. Bu, tutarsız uygulama state'ine yol açabilir.

İyimser recomposition işlemini gerçekleştirmek için tüm composable fonksiyonların ve lambdaların idempotent ve side effect free olduğundan emin olun.

### Composable functions might run quite frequently
Bazı durumlarda, bir UI animasyonunun her karesi için composable bir fonksiyon çalışabilir. Fonksiyon, cihaz depolama alanından okuma gibi pahalı işlemler gerçekleştiriyorsa, UI sıkıntısına neden olabilir.

Örneğin, widget'ınız cihaz ayarlarını okumaya çalışırsa, bu ayarları saniyede yüzlerce kez okuyabilir ve uygulamanızın performansı üzerinde feci etkiler yaratabilir.

Composable fonksiyonunuz veriye ihtiyaç duyuyorsa, veri için parametreler tanımlamalıdır. Daha sonra pahalı işleri composition dışında başka bir thread'e taşıyabilir ve mutableStateOf veya LiveData kullanarak verileri Compose'a aktarabilirsiniz.

## Learn More
Compose ve composable fonksiyonlarda nasıl düşüneceğiniz hakkında daha fazla bilgi edinmek için aşağıdaki ek kaynaklara göz atın.

### Videos
+ [Composable funtions - MAD Skills](https://www.youtube.com/watch?v=fFLBCgoHHys)
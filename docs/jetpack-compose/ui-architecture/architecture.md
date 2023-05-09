---
layout: default
title: Architecture
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 5
---

# Architecting your Compose UI
Compose'da UI immutable'dır - çizildikten sonra onu güncellemenin bir yolu yoktur. Kontrol edebileceğiniz şey UI'nizin state'idir. UI'nin state'i her değiştiğinde, [Compose UI ağacının değişen kısımlarını yeniden oluşturur.](/docs/jetpack-compose/introduction/thinking-in-compose/#recomposition) Composable'lar state kabul edebilir ve event'leri açığa çıkarabilir; örneğin bir TextField bir değer kabul eder ve callback handler'dan değeri değiştirmesini isteyen bir callback onValueChange sunar.
```kotlin
var name by remember { mutableStateOf("") }
OutlinedTextField(
    value = name,
    onValueChange = { name = it },
    label = { Text("Name") }
)
```
Composable'lar state kabul ettiğinden ve event'leri açığa çıkardığından, tek yönlü veri akış modeli(unidirectional data flow pattern) Jetpack Compose ile iyi uyum sağlar. Bu kılavuz, Compose'da tek yönlü veri akışı modelinin nasıl uygulanacağına, event'lerin ve state holder'ların nasıl uygulanacağına ve Compose'da ViewModel'lerle nasıl çalışılacağına odaklanmaktadır.

{: .note }
Not: Uygulamanızın diğer katmanları (data katmanı ve business katmanı) Jetpack Compose'un benimsenmesinden etkilenmez. Uygulamanızın tüm katmanlarını tasarlama hakkında daha fazla bilgi edinmek için [uygulama mimarisi kılavuzu](/docs/app-architecture/guide-to-app-architecture/about-app-architecture/)na göz atın.

## Unidirectional data flow
Tek yönlü veri akışı (UDF), state'in aşağı ve event'lerin yukarı aktığı bir tasarım modelidir. Tek yönlü veri akışını izleyerek, UI'da state'i görüntüleyen composable'ları uygulamanızın state'i depolayan ve değiştiren kısımlarından ayırabilirsiniz.

Tek yönlü veri akışını kullanan bir uygulamanın UI güncelleme döngüsü şu şekildedir:

- **Event**: UI'nin bir kısmı bir event üretir ve bunu yukarı doğru iletir, mesela ViewModel'e ele alması için iletilen bir buton tıklaması gibi; veya uygulamanızın diğer katmanlarından bir event iletilir, mesela kullanıcı oturumunun süresinin dolduğunu belirtmek gibi.


- **Update state**: Bir event handler state'i değiştirebilir.


- **Display state**: State holder state'i aşağı aktarır ve UI bunu görüntüler.

![](https://developer.android.com/static/images/jetpack/compose/state-unidirectional-flow.png)

Sekil 1.Unidirectional data flow (tek yönlü veri akışı) modeli


Jetpack Compose kullanırken bu modeli takip etmek çeşitli avantajlar sağlar:

- **Test edilebilirlik**: State'i, onu görüntüleyen UI'dan ayırmak, her ikisini de izole olarak test etmeyi kolaylaştırır.


- **State encapsulation**: State yalnızca tek bir yerde güncellenebildiğinden ve bir composable'ın state'i için yalnızca tek bir doğruluk kaynağı olduğundan, tutarsız state'ler nedeniyle hata oluşturma olasılığınız daha düşüktür.


- **UI tutarlılığı**: Tüm state güncellemeleri, StateFlow veya LiveData gibi gözlemlenebilir state holder'lar kullanılarak anında UI'ye yansıtılır.

### Unidirectional data flow in Jetpack Compose
Composables state ve event'lere dayalı olarak çalışır. Örneğin, bir TextField yalnızca değer parametresi güncellendiğinde güncellenir ve bir onValueChange callback (değerin yenisiyle değiştirilmesini isteyen bir event) sunar. Compose, State nesnesini bir value tutucu olarak tanımlar ve state değerindeki değişiklikler bir recomposition'ı tetikler. Değeri ne kadar süreyle hatırlamanız gerektiğine bağlı olarak state'i bir remember { mutableStateOf(value) } veya bir rememberSaveable { mutableStateOf(value) içinde tutabilirsiniz.

TextField composable'ın değerinin türü String'dir, dolayısıyla bu değer herhangi bir yerden gelebilir; sabit kodlanmış bir değerden, bir ViewModel'den veya üst composable'dan aktarılabilir. Bunu bir State nesnesinde tutmanız gerekmez, ancak onValueChange çağrıldığında değeri güncellemeniz gerekir.

{: .note }
Önemli Noktalar
- **mutableStateOf(value)**, Compose'da gözlemlenebilir bir tür olan bir MutableState oluşturur. Değerindeki herhangi bir değişiklik, bu değeri okuyan herhangi bir composable fonksiyonunun yeniden oluşturulmasını(recomposition) programlayacaktır.
- **remember** nesneleri composition'da saklar ve remember'ı çağıran composable composition'dan kaldırıldığında nesneyi unutur.
- **rememberSaveable**, bir Bundle'a kaydederek konfigürasyon değişiklikleri boyunca durumu korur.


{: .note }
Not: Compose'da state ve state hoisting hakkında daha fazla bilgi edinmek için [State ve Jetpack Compose](/docs/jetpack-compose/ui-architecture/managing-state/overview/#state-and-jetpack-compose) bölümüne bakın.

### Define composable parameters
Bir composable'ın state parametrelerini tanımlarken aşağıdaki soruları aklınızda tutmalısınız:

- *Composable ne kadar yeniden kullanılabilir veya esnektir?*

- *State parametreleri bu composable'ın performansını nasıl etkiler?*

Decoupling'i ve yeniden kullanımı teşvik etmek için, her bir composable mümkün olan en az miktarda bilgiyi tutmalıdır. Örneğin, bir haber makalesinin başlığını tutmak için bir composable oluştururken, tüm haber makalesi yerine yalnızca görüntülenmesi gereken bilgileri aktarmayı tercih edin:

```kotlin
@Composable
fun Header(title: String, subtitle: String) {
    // Başlık veya altyazı değiştiğinde yeniden oluşturur.
}

@Composable
fun Header(news: News) {
    // Yeni bir News instance'ı gecildiginde yeniden oluşturur.
}
```
Bazen tek tek parametreler kullanmak da performansı artırır - örneğin, News yalnızca başlık ve altyazıdan daha fazla bilgi içeriyorsa, Header(news) öğesine yeni bir News instance'ı geçildiğinde, başlık ve altyazı değişmemiş olsa bile composable yeniden oluşturulur.

Geçtiğiniz parametre sayısını dikkatlice düşünün. Çok fazla parametresi olan bir fonksiyona sahip olmak fonksiyonun ergonomisini azaltır, bu nedenle bu durumda bunları bir sınıfta gruplamak tercih edilir.


## Events in Compose
Uygulamanıza yapılan her girdi bir event olarak temsil edilmelidir: dokunmalar, metin değişiklikleri ve hatta zamanlayıcılar veya diğer güncellemeler. Bu event'ler UI'nızın state'ini değiştirdiğinden, bunları ele alacak ve UI state'ini güncelleyecek olan ViewModel olmalıdır.

UI katmanı asla bir event handler dışında state değiştirmemelidir çünkü bu uygulamanızda tutarsızlıklara ve hatalara yol açabilir.

State ve event handler lambdaları için immutable değerler geçirmeyi tercih edin. Bu yaklaşımın aşağıdaki faydaları vardır:

- Yeniden kullanılabilirliği artırırsınız.

- UI'nizin state değerini doğrudan değiştirmemesini sağlarsınız.

- State'in başka bir thread tarafından değiştirilmediğinden emin olduğunuz için eş zamanlılık(concurrency) sorunlarından kaçınırsınız.

- Genellikle kod karmaşıklığını azaltırsınız.

Örneğin, parametre olarak bir String ve bir lambda kabul eden bir composable birçok bağlamdan(context) çağrılabilir ve yüksek oranda yeniden kullanılabilir. Uygulamanızdaki app bar'ın her zaman metin gösterdiğini ve bir geri butonu olduğunu varsayalım. Metni ve geri butonu handle'ını parametre olarak alan daha genel bir MyAppTopAppBar composable'ı tanımlayabilirsiniz:

```kotlin
@Composable
fun MyAppTopAppBar(topAppBarText: String, onBackPressed: () -> Unit) {
    TopAppBar(
        title = {
            Text(
                text = topAppBarText,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxSize()
                    .wrapContentSize(Alignment.Center)
            )
        },
        navigationIcon = {
            IconButton(onClick = onBackPressed) {
                Icon(
                    Icons.Filled.ArrowBack,
                    contentDescription = localizedString
                )
            }
        },
        // ...
    )
}
```
### ViewModels, states, and events: an example
`ViewModel` ve `mutableStateOf` kullanarak, aşağıdakilerden biri doğruysa uygulamanızda tek yönlü veri akışı(unidirectional data flow) da sağlayabilirsiniz:

- UI'nizin state'i, `StateFlow` veya `LiveData` gibi gözlemlenebilir state holder'lar aracılığıyla açığa çıkar.


- ViewModel, UI'dan veya uygulamanızın diğer katmanlarından gelen eventleri ele alır ve state holder'ı eventlere göre günceller.

Örneğin, bir oturum açma ekranı uygularken, Oturum Aç butonuna dokunmak uygulamanızın bir progress spinner ve bir ağ çağrısı görüntülemesine neden olmalıdır. Oturum açma işlemi başarılı olduysa uygulamanız farklı bir ekrana gider; hata durumunda ise uygulama bir Snackbar gösterir. Ekran state'ini ve event'i şu şekilde modelleyebilirsiniz:

Ekranın dört state'i vardır:

- **Oturum kapatıldı**: kullanıcı henüz oturum açmadığında.


- **Devam ediyor**: uygulamanız şu anda bir ağ çağrısı gerçekleştirerek kullanıcıyla oturum açmaya çalışıyorsa.


- **Hata**: oturum açma sırasında bir hata oluştuğunda.


- **Oturum açıldı**: kullanıcı oturum açtığında.

Bu state'leri sealed bir sınıf olarak modelleyebilirsiniz. ViewModel state'i bir State olarak gösterir, ilk state'i ayarlar ve gerektiğinde state'i günceller. `ViewModel` ayrıca bir `onSignIn()` metodu göstererek oturum açma olayını yönetir.

```kotlin
class MyViewModel : ViewModel() {
    private val _uiState = mutableStateOf<UiState>(UiState.SignedOut)
    val uiState: State<UiState>
        get() = _uiState

    // ...
}
```
MutableStateOf API'sine ek olarak Compose, LiveData, Flow ve Observable için bir dinleyici olarak kaydolmak ve değeri bir state olarak temsil etmek için [extension fonksiyonlar](https://developer.android.com/jetpack/compose/migrate#streams) sağlar.

```kotlin
class MyViewModel : ViewModel() {
    private val _uiState = MutableLiveData<UiState>(UiState.SignedOut)
    val uiState: LiveData<UiState>
        get() = _uiState

    // ...
}

@Composable
fun MyComposable(viewModel: MyViewModel) {
    val uiState = viewModel.uiState.observeAsState()
    // ...
}
```
## Learn More
Jetpack Compose'da mimari hakkında daha fazla bilgi edinmek için aşağıdaki kaynaklara başvurun:

### Samples
- [Jetnews sample](https://github.com/android/compose-samples/tree/main/JetNews)
- [Jetchat sample](https://github.com/android/compose-samples/tree/main/Jetchat)
- [Now in Android App](https://github.com/android/nowinandroid/tree/main)
- [Sunflower with Compose](https://github.com/android/sunflower/tree/main)
- [Crane sample](https://github.com/android/compose-samples/tree/main/Crane)
- [Architecture](https://github.com/android/architecture-samples/tree/main)
---
layout: default
title: Navigation
parent: UI Architecture
grand_parent: Jetpack Compose
nav_order: 8
---
# Navigating with Compose
[Navigation komponenti](https://developer.android.com/guide/navigation) [Jetpack Compose](https://developer.android.com/jetpack/compose) uygulamaları için destek sağlar. Navigation komponentinin altyapısından ve özelliklerinden yararlanırken composable'lar arasında gezinebilirsiniz.

{: .note }
Not: Compose hakkında bilgi sahibi değilseniz, devam etmeden önce [Jetpack Compose](https://developer.android.com/jetpack/compose) kaynaklarını inceleyin.

## Setup
Compose'u desteklemek için, uygulama modülünüzün `build.gradle' dosyasında aşağıdaki bağımlılığı kullanın:
```groovy
dependencies {
    def nav_version = "2.5.3"

    implementation "androidx.navigation:navigation-compose:$nav_version"
}
```
## Getting Started
[NavController](https://developer.android.com/reference/androidx/navigation/NavController), Navigasyon komponenti için merkezi API'dir. Stateful'dur ve uygulamanızdaki ekranları oluşturan composable'ların back stack'ini ve her bir ekranın state'ini takip eder.

composable'ınızda [rememberNavController()](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary#rememberNavController(kotlin.Array)) metodunu kullanarak bir NavController oluşturabilirsiniz:

```kotlin
val navController = rememberNavController()
```
`NavController`'ı, composable hiyerarşinizde ona referans vermesi gereken tüm composable'ların erişebileceği bir yerde oluşturmalısınız. Bu, [state hoisting](/docs/jetpack-compose/ui-architecture/managing-state/overview/#state-hoisting) ilkelerini takip eder ve NavController'ı ve [currentBackStackEntryAsState()](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary#(androidx.navigation.NavController).currentBackStackEntryAsState()) aracılığıyla sağladığı state'i, ekranlarınızın dışındaki composable'ları güncellemek için doğruluk kaynağı olarak kullanmanıza olanak tanır. Bu işlevselliğin bir örneği için [Bottom Navbar ile Entegrasyon bölümü](#integration-with-the-bottom-nav-bar)ne bakın.

{: .note }
Not: Fragmentler için Navigation component kullanıyorsanız, Compose'da yeni navigasyon grafikleri tanımlamanız veya NavHost composables kullanmanız gerekmez. Daha fazla bilgi için [Birlikte Çalışabilirlik](#interoperability) bölümüne bakın.

## Creating a NavHost
Her [NavController](https://developer.android.com/reference/androidx/navigation/NavController) tek bir [NavHost](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary#NavHost(androidx.navigation.NavHostController,kotlin.String,androidx.compose.ui.Modifier,kotlin.String,kotlin.Function1)) composable ile ilişkilendirilmelidir. NavHost, NavController'ı aralarında gezinebileceğiniz composable hedefleri belirten bir navigasyon grafiğine bağlar. Composable'lar arasında gezinirken, NavHost'un içeriği otomatik olarak [yeniden oluşturulur](/docs/jetpack-compose/introduction/thinking-in-compose/#recomposition). Navigasyon grafiğinizdeki her bir composable hedef bir `route` ile ilişkilendirilir.

{: .note }
Anahtar Terim: Route, composable'ınıza giden yolu tanımlayan bir String'dir. Bunu, belirli bir hedefe götüren örtük bir deep link olarak düşünebilirsiniz. Her hedefin benzersiz bir route'u olmalıdır.

NavHost'un oluşturulması için daha önce rememberNavController() aracılığıyla oluşturulan NavController ve grafiğinizin başlangıç hedefinin route'u gerekir. NavHost oluşturma, navigasyon grafiğinizi oluşturmak için [Navigation Kotlin DSL](https://developer.android.com/guide/navigation/navigation-kotlin-dsl#navgraphbuilder)'deki lambda sentaksını kullanır. [composable()](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary#(androidx.navigation.NavGraphBuilder).composable(kotlin.String,kotlin.collections.List,kotlin.collections.List,kotlin.Function1)) metodunu kullanarak navigasyon yapınıza eklemeler yapabilirsiniz. Bu yöntem, bir route ve hedefe bağlanması gereken composable'ı sağlamanızı gerektirir:
```kotlin
NavHost(navController = navController, startDestination = "profile") {
    composable("profile") { Profile(/*...*/) }
    composable("friendslist") { FriendsList(/*...*/) }
    /*...*/
}
```

{: .note }
Not: Navigasyon Komponenti, [Navigasyon İlkeleri](https://developer.android.com/guide/navigation/navigation-principles#fixed_start_destination)ne uymanızı ve sabit bir başlangıç hedefi kullanmanızı gerektirir. startDestination route için composable bir değer kullanmamalısınız.

## Navigate to a composable
Navigasyon grafiğinde composable bir hedefe gitmek için [navigate](https://developer.android.com/reference/androidx/navigation/NavController#navigate(kotlin.String,androidx.navigation.NavOptions,androidx.navigation.Navigator.Extras)) metodunu kullanmanız gerekir. `navigate`, hedefin route'unu temsil eden tek bir String parametresi alır. Navigasyon grafiği içindeki bir composable'dan navigasyon yapmak için `navigate`'i çağırın:
```kotlin
navController.navigate("friendslist")
```
Varsayılan olarak, [navigate](https://developer.android.com/reference/androidx/navigation/NavController#navigate(kotlin.String,androidx.navigation.NavOptions,androidx.navigation.Navigator.Extras)) yeni hedefinizi back stack'e ekler. navigate() çağrımıza ek navigasyon seçenekleri ekleyerek navigate'in davranışını değiştirebilirsiniz:
```kotlin
// "friendslist" hedefine gitmeden önce her şeyi back stack'ten "home" 
// hedefine kadar cikarin(pop)
navController.navigate("friendslist") {
    popUpTo("home")
}

// "friendslist" hedefine gitmeden önce "home" hedefine kadar ve "home" hedefi de dahil olmak üzere her şeyi back stack'ten çıkarın(pop edin)
navController.navigate("friendslist") {
    popUpTo("home") { inclusive = true }
}

// "search" hedefine yalnızca zaten "search" hedefinde değilsek gidin, 
// böylece back stack'in tepesinde birden fazla kopyadan kaçınmış oluruz
navController.navigate("search") {
    launchSingleTop = true
}
```
Daha fazla kullanım örneği için [popUpTo](https://developer.android.com/guide/navigation/navigation-navigate#pop) kılavuzuna bakın.

{: .note }
Not: [Animasyon bloğu](https://developer.android.com/reference/kotlin/androidx/navigation/NavOptionsBuilder#anim(kotlin.Function1)) Navigation Compose ile kullanılamaz. Navigation Compose'daki Geçiş Animasyonları [bu özellik talebi](https://issuetracker.google.com/172112072)nde takip edilmektedir.

### Navigate calls triggered by other composable functions
`NavController`ın navigate fonksiyonu, `NavController`ın dahili state'ini değiştirir. [Tek doğruluk kaynağı](/docs/app-architecture/guide-to-app-architecture/about-app-architecture/#single-source-of-truth) ilkesine mümkün olduğunca uymak için, yalnızca NavController instance'ını hoist eden composable fonksiyon veya state holder ve NavController'ı parametre olarak alan composable fonksiyonlar navigasyon çağrıları yapmalıdır. UI hiyerarşisinde daha aşağıda yer alan diğer composable fonksiyonlardan tetiklenen navigasyon event'lerinin, fonksiyonları kullanarak bu event'leri arayan kişiye uygun şekilde göstermesi gerekir.

Aşağıdaki örnekte, NavController instance'ı için tek doğruluk kaynağı olarak MyAppNavHost composable fonksiyonu gösterilmektedir. `ProfileScreen` bir event'i, kullanıcı bir button'a dokunduğunda çağrılan bir fonksiyon olarak sunar. Uygulamadaki farklı ekranlarda gezinmenin sorumlusu olan `MyAppNavHost`, `ProfileScreen`'i çağırırken doğru hedefe navigasyon çağrısı yapar.
```kotlin
@Composable
fun MyAppNavHost(
    modifier: Modifier = Modifier,
    navController: NavHostController = rememberNavController(),
    startDestination: String = "profile"
) {
    NavHost(
        modifier = modifier,
        navController = navController,
        startDestination = startDestination
    ) {
        composable("profile") {
            ProfileScreen(
                onNavigateToFriends = { navController.navigate("friendsList") },
                /*...*/
            )
        }
        composable("friendslist") { FriendsListScreen(/*...*/) }
    }
}

@Composable
fun ProfileScreen(
    onNavigateToFriends: () -> Unit,
    /*...*/
) {
    /*...*/
    Button(onClick = onNavigateToFriends) {
        Text(text = "See friends list")
    }
}
```
Her recomposition'da `navigate()` fonksiyonunu çağırmaktan kaçınmak için navigate() fonksiyonunu composable'ınızın bir parçası olarak değil, sadece bir callback'in parçası olarak çağırmalısınız.

#### Best Practices
Uygulamadaki belirli bir mantığın nasıl işleneceğini bilen caller'lara composable fonksiyonlardan event'leri göstermek, [Compose'da state'i hoist ederken iyi bir pratiktir](/docs/jetpack-compose/ui-architecture/managing-state/overview/#state-hoisting).

Eventleri ayrı lambda parametreleri olarak göstermek fonksiyon imzasını overload edebilecek olsa da, composable fonksiyon sorumluluklarının ne olduğunun görünürlüğünü maksimize eder. Bir bakışta ne yaptığını görebilirsiniz.

Fonksiyon bildirimindeki parametre sayısını azaltabilecek diğer alternatifler başlangıçta daha rahat yazılabilir ancak uzun vadede bazı dezavantajları gizleyebilir. Örneğin, tüm eventleri tek bir yerde merkezileştiren ProfileScreenEvents gibi bir sarmalayıcı sınıf oluşturmak. Bunu yapmak, fonksiyon tanımından geçerken composable'ın ne yaptığının görünürlüğünü azaltır, proje sayınıza başka bir sınıf ve metot ekler ve zaten bu composable fonksiyonunu her çağırdığınızda bu sınıfın instance'larını oluşturmanız ve hatırlamanız gerekir. Ayrıca, bu sarmalayıcı sınıfı mümkün olduğunca yeniden kullanmak için, bu model, [composable'lara sadece ihtiyaç duyduklarını iletmek](/docs/jetpack-compose/ui-architecture/architecture/#define-composable-parameters) gibi en iyi pratik yerine, bu sınıfın bir instance'ını UI hiyerarşisinde aşağıya doğru geçirmeyi teşvik eder.


## Navigate with arguments
Navigation Compose, composable hedefler arasında argüman aktarımını da destekler. Bunu yapmak için, temel navigasyon kütüphanesini kullanırken bir [deep linke argüman eklediğinize](https://developer.android.com/guide/navigation/navigation-deep-link#implicit) benzer şekilde rotanıza argüman yer tutucuları eklemeniz gerekir:
```kotlin
NavHost(startDestination = "profile/{userId}") {
    ...
    composable("profile/{userId}") {...}
}
```
Varsayılan olarak, tüm argümanlar string olarak ayrıştırılır. `composable()` metodunun `arguments` parametresi bir [NamedNavArguments](https://developer.android.com/reference/androidx/navigation/NamedNavArgument) listesi kabul eder. [navArgument](https://developer.android.com/reference/kotlin/androidx/navigation/package-summary#navArgument(kotlin.String,kotlin.Function1)) metodunu kullanarak hızlı bir şekilde bir NamedNavArgument oluşturabilir ve ardından tam `tipini` belirtebilirsiniz:

```kotlin
NavHost(startDestination = "profile/{userId}") {
    ...
    composable(
        "profile/{userId}",
        arguments = listOf(navArgument("userId") { type = NavType.StringType })
    ) {...}
}
```
Argümanları `composable() ` lambda'sında bulunan [NavBackStackEntry](https://developer.android.com/reference/kotlin/androidx/navigation/NavBackStackEntry)'den çıkarmalısınız.
```kotlin
composable("profile/{userId}") { backStackEntry ->
    Profile(navController, backStackEntry.arguments?.getString("userId"))
}
```
Argümanı hedefe iletmek için, navigate çağrısını yaptığınızda rotaya eklemeniz gerekir:
```kotlin
navController.navigate("profile/user1234")
```
Desteklenen türlerin listesi için [Hedefler arasında veri aktarma](https://developer.android.com/guide/navigation/navigation-pass-data#supported_argument_types) bölümüne bakın.

### Retrieving complex data when navigating
Navigasyon sırasında kompleks veri nesnelerinin aktarılmaması, bunun yerine navigasyon eylemleri gerçekleştirilirken benzersiz bir tanımlayıcı veya başka bir ID biçimi gibi gerekli minimum bilgilerin argüman olarak aktarılması şiddetle tavsiye edilir:
```kotlin
// Yeni bir hedefe giderken argüman olarak yalnızca kullanıcı ID'sini geçirin
navController.navigate("profile/user1234")
```
Kompleks nesneler, data katmanı gibi tek bir doğruluk kaynağında veri olarak saklanmalıdır. Navigasyondan sonra hedefinize ulaştığınızda, aktarılan ID'yi kullanarak tek bir doğruluk kaynağından gerekli bilgileri yükleyebilirsiniz. Data katmanına erişmekten sorumlu olan ViewModel'inizdeki argümanları almak için ViewModel'in [SavedStateHandle](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-savedstate#savedstatehandle)'ını kullanabilirsiniz:
```kotlin
class UserViewModel(
    savedStateHandle: SavedStateHandle,
    private val userInfoRepository: UserInfoRepository
) : ViewModel() {

    private val userId: String = checkNotNull(savedStateHandle["userId"])

   // İletilen userId argümanına dayalı olarak data katmanından, 
   // yani userInfoRepository'den ilgili kullanıcı bilgilerini getirin
    private val userInfo: Flow<UserInfo> = userInfoRepository.getUserInfo(userId)

// …

}
```
Bu yaklaşım, konfigürasyon değişiklikleri sırasında veri kaybını ve söz konusu nesne güncellenirken veya mutasyona uğrarken oluşabilecek tutarsızlıkları önlemeye yardımcı olur.

Kompleks verileri argüman olarak geçirmekten neden kaçınmanız gerektiğine dair daha ayrıntılı bir açıklama ve desteklenen argüman türlerinin bir listesi için bkz. [Hedefler arasında veri aktarma](https://developer.android.com/guide/navigation/navigation-pass-data#supported_argument_types).

### Adding optional arguments
Navigation Compose isteğe bağlı navigasyon argümanlarını da destekler. İsteğe bağlı argümanlar, gerekli argümanlardan iki şekilde farklıdır:

- Sorgu parametresi sentaksı kullanılarak dahil edilmelidirler (`"?argName={argName}"`)

- Bir defaultValue ayarına sahip olmalı veya `nullable = true` (varsayılan değeri dolaylı olarak `null` olarak ayarlar) olmalıdır

Bu, tüm isteğe bağlı argümanların `composable()` fonksiyonuna bir liste olarak açıkça eklenmesi gerektiği anlamına gelir:

```kotlin
composable(
    "profile?userId={userId}",
    arguments = listOf(navArgument("userId") { defaultValue = "user1234" })
) { backStackEntry ->
    Profile(navController, backStackEntry.arguments?.getString("userId"))
}
```
Artık, hedefe herhangi bir argüman iletilmese bile, bunun yerine `defaultValue`, `"user1234"` kullanılır.

Argümanları rotalar aracılığıyla ele alma yapısı, composable'larınızın Navigation'dan tamamen bağımsız kalması anlamına gelir ve onları çok daha test edilebilir hale getirir.

## Deep links
Navigation Compose, composable() fonksiyonunun bir parçası olarak tanımlanabilen örtük deeplink'leri de destekler. deepLinks parametresi, [navDeepLink](https://developer.android.com/reference/kotlin/androidx/navigation/package-summary#navDeepLink(kotlin.Function1)) metodu kullanılarak hızlı bir şekilde oluşturulabilen [NavDeepLinks](https://developer.android.com/reference/androidx/navigation/NavDeepLink) listesini kabul eder:
```kotlin
val uri = "https://www.example.com"

composable(
    "profile?id={id}",
    deepLinks = listOf(navDeepLink { uriPattern = "$uri/{id}" })
) { backStackEntry ->
    Profile(navController, backStackEntry.arguments?.getString("id"))
}
```
Bu deep linkler belirli bir URL'yi, action'ı veya mime tipini composable ile ilişkilendirmenizi sağlar. Varsayılan olarak, bu deep linkler harici uygulamalara açık değildir. Bu deep linkleri harici olarak kullanılabilir hale getirmek için uygulamanızın `manifest.xml` dosyasına uygun` <intent-filter>` öğelerini eklemeniz gerekir. Yukarıdaki deep link'i etkinleştirmek için manifest'in `<activity>` elementinin içine aşağıdakileri eklemelisiniz:

```kotlin
<activity …>
  <intent-filter>
    ...
    <data android:scheme="https" android:host="www.example.com" />
  </intent-filter>
</activity>
```
Navigasyon, deep link başka bir uygulama tarafından tetiklendiğinde otomatik olarak o composable'a deep link verir.

Aynı deep linkler, bir composable'dan uygun deep link ile bir `PendingIntent` oluşturmak için de kullanılabilir:
```kotlin
val id = "exampleId"
val context = LocalContext.current
val deepLinkIntent = Intent(
    Intent.ACTION_VIEW,
    "https://www.example.com/$id".toUri(),
    context,
    MyActivity::class.java
)

val deepLinkPendingIntent: PendingIntent? = TaskStackBuilder.create(context).run {
    addNextIntentWithParentStack(deepLinkIntent)
    getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT)
}
```
Daha sonra uygulamanızı deeplink hedefinde açmak için bu `deepLinkPendingIntent`'i diğer `PendingIntent`'ler gibi kullanabilirsiniz.


## Nested Navigation
Hedefler, uygulamanızın UI'sindeki belirli bir akışı modülerleştirmek için [iç içe geçmiş bir grafikte (nested graph)](https://developer.android.com/guide/navigation/navigation-design-graph#nested_graphs) gruplandırılabilir. Bunun bir örneği, kendi kendine çalışan bir oturum açma akışı olabilir.

İç içe grafik(nested graph), hedeflerini encapsulate eder. Root grafiğinde(root graph) olduğu gibi, iç içe geçmiş bir grafiğin de rotası tarafından başlangıç hedefi olarak tanımlanan bir hedefi olmalıdır. Bu, iç içe grafikle ilişkili rotaya gittiğinizde gidilen hedeftir.

NavHost'unuza iç içe bir grafik eklemek için [navigation](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary#(androidx.navigation.NavGraphBuilder).navigation(kotlin.String,kotlin.String,kotlin.collections.List,kotlin.collections.List,kotlin.Function1)) extension fonksiyonunu kullanabilirsiniz:

```kotlin
NavHost(navController, startDestination = "home") {
    ...
    // Grafiğe otomatik olarak rotası ('login') üzerinden gitmek için
    // grafiğin başlangıç hedefine gider - 'username' 
    // Bu nedenle grafiğin dahili rota belirleme mantığını enkapsüle eder
    navigation(startDestination = "username", route = "login") {
        composable("username") { ... }
        composable("password") { ... }
        composable("registration") { ... }
    }
    ...
}
```
Grafiğin boyutu büyüdükçe navigasyon grafiğinizi birden fazla metoda bölmeniz şiddetle tavsiye edilir. Bu aynı zamanda birden fazla modülün kendi navigasyon grafiklerine katkıda bulunmasına olanak tanır.
```kotlin
fun NavGraphBuilder.loginGraph(navController: NavController) {
    navigation(startDestination = "username", route = "login") {
        composable("username") { ... }
        composable("password") { ... }
        composable("registration") { ... }
    }
}
```
Metodu [NavGraphBuilder](https://developer.android.com/reference/kotlin/androidx/navigation/NavGraphBuilder) üzerinde bir extension metodu haline getirerek, önceden oluşturulmuş `navigasyon`, `composable` ve `dialog` extension metotlarının yanında kullanabilirsiniz:
```kotlin
NavHost(navController, startDestination = "home") {
    ...
    loginGraph(navController)
    ...
}
```

## Integration with the bottom nav bar
`NavController`'ı composable hiyerarşinizde daha yüksek bir seviyede tanımlayarak Navigation'ı bottom navigation component gibi diğer componentlere bağlayabilirsiniz. Bunu yapmak, bottom bardaki ikonları seçerek navigasyon yapmanızı sağlar.

[BottomNavigation](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#BottomNavigation(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.Dp,kotlin.Function1)) ve [BottomNavigationItem](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#(androidx.compose.foundation.layout.RowScope).BottomNavigationItem(kotlin.Boolean,kotlin.Function0,kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Function0,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color)) komponentlerini kullanmak için Android uygulamanıza androidx.compose.material bağımlılığını ekleyin.
```grovy
dependencies {
    implementation "androidx.compose.material:material:1.4.3"
}

android {
    buildFeatures {
        compose true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.4.7"
    }

    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```
Bottom navigation bar'daki öğeleri navigasyon grafiğinizdeki rotalara bağlamak için, burada görülen `Screen` gibi, hedefler için rota ve String resource ID'sini içeren bir sealed sınıf tanımlamanız önerilir.
```kotlin
sealed class Screen(val route: String, @StringRes val resourceId: Int) {
    object Profile : Screen("profile", R.string.profile)
    object FriendsList : Screen("friendslist", R.string.friends_list)
}
```
Ardından bu öğeleri [BottomNavigationItem](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#(androidx.compose.foundation.layout.RowScope).BottomNavigationItem(kotlin.Boolean,kotlin.Function0,kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Function0,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color)) tarafından kullanılabilecek bir listeye yerleştirin:
```kotlin
val items = listOf(
   Screen.Profile,
   Screen.FriendsList,
)
```
[BottomNavigation](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#BottomNavigation(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.Dp,kotlin.Function1)) composable'ınızda, currentBackStackEntryAsState() fonksiyonunu kullanarak şu anki NavBackStackEntry'yi alın. Bu girdi size geçerli NavDestination'a erişim sağlar. Her BottomNavigationItem öğesinin seçili state'i daha sonra [NavDestination](https://developer.android.com/reference/androidx/navigation/NavDestination) hiyerarşisi aracılığıyla öğenin rotası geçerli hedefin ve onun üst hedeflerinin ([iç içe navigasyon](#nested-navigation) kullandığınız durumları ele almak için) rotasıyla karşılaştırılarak belirlenebilir.

Öğenin rotası aynı zamanda onClick lambda'sını navigasyon çağrısına bağlamak için de kullanılır, böylece öğeye dokunulduğunda o öğeye gidilir. saveState ve restoreState flag'lerini kullanarak, botttom navigation öğeleri arasında geçiş yaparken bu öğenin state'i ve back stack'i doğru şekilde kaydedilir ve geri yüklenir.
```kotlin
val navController = rememberNavController()
Scaffold(
  bottomBar = {
    BottomNavigation {
      val navBackStackEntry by navController.currentBackStackEntryAsState()
      val currentDestination = navBackStackEntry?.destination
      items.forEach { screen ->
        BottomNavigationItem(
          icon = { Icon(Icons.Filled.Favorite, contentDescription = null) },
          label = { Text(stringResource(screen.resourceId)) },
          selected = currentDestination?.hierarchy?.any { it.route == screen.route } == true,
          onClick = {
            navController.navigate(screen.route) {
                // Kullanıcılar öğeleri seçtikçe back stack üzerinde büyük bir 
                // hedef stack'i oluşmasını önlemek için 
                // grafiğin başlangıç hedefine pop edilir
              popUpTo(navController.graph.findStartDestination().id) {
                saveState = true
              }
              // Aynı öğeyi yeniden seçerken aynı hedefin birden fazla kopyasını önlemek
              launchSingleTop = true
              // Önceden seçilmiş bir öğe yeniden seçildiğinde state'i geri yüklemek
              restoreState = true
            }
          }
        )
      }
    }
  }
) { innerPadding ->
  NavHost(navController, startDestination = Screen.Profile.route, Modifier.padding(innerPadding)) {
    composable(Screen.Profile.route) { Profile(navController) }
    composable(Screen.FriendsList.route) { FriendsList(navController) }
  }
}
```
Burada `NavController.currentBackStackEntryAsState()` metodundan yararlanarak navController state'ini NavHost fonksiyonundan çıkarır ve BottomNavigation komponentiyle paylaşırsınız. Bu, `BottomNavigation`'un otomatik olarak en güncel state'e sahip olduğu anlamına gelir.

## Type safety in Navigation Compose
[Type safe, multi-module best practices with Navigation Compose](https://youtu.be/goFpG25uoc8)

Bu sayfadaki kod type-safe değildir. Navigate() fonksiyonunu mevcut olmayan rotalarla veya yanlış argümanlarla çağırabilirsiniz. Ancak, Navigasyon kodunuzu çalışma zamanında type-safe olacak şekilde yapılandırabilirsiniz. Bunu yaparak, çökmeleri önleyebilir ve şunlardan emin olabilirsiniz:

- Bir hedefe veya navigasyon grafiğine giderken sağladığınız argümanlar doğru türdedir ve gerekli tüm argümanlar mevcuttur.

- SavedStateHandle'dan aldığınız argümanlar doğru türdedir.

Bu konuda daha fazla bilgi için [Navigasyon type safety](https://developer.android.com/guide/navigation/navigation-type-safety) belgelerine göz atın.

## Interoperability
Navigation component'i Compose ile kullanmak istiyorsanız iki seçeneğiniz vardır:

- Fragmentler için Navigation bileşeni ile bir navigasyon grafiği tanımlayın.

- Compose hedeflerini kullanarak Compose'da bir NavHost ile bir navigasyon grafiği tanımlayın. Bu, yalnızca navigasyon grafiğindeki tüm ekranlar composable ise mümkündür.


Bu nedenle, karma Compose ve Views uygulamaları için öneri Fragment tabanlı Navigasyon komponentini kullanmaktır. Fragmentler daha sonra View tabanlı ekranları, Compose ekranlarını ve hem Views hem de Compose kullanan ekranları tutacaktır. Her bir Fragment'ın içeriği Compose'da olduğunda, bir sonraki adım tüm bu ekranları Navigation Compose ile birbirine bağlamak ve tüm Fragment'ları kaldırmaktır.

### Navigate from Compose with Navigation for fragments
Compose kodu içindeki hedefleri değiştirmek için, hiyerarşideki herhangi bir composable'a aktarılabilen ve onlar tarafından tetiklenebilen event'leri açığa çıkarırsınız:
```kotlin
@Composable
fun MyScreen(onNavigate: (Int) -> ()) {
    Button(onClick = { onNavigate(R.id.nav_profile) } { /* ... */ }
}
```
Fragmentinizde, NavController'ı bularak ve hedefe giderek Compose ile fragment tabanlı Navigation komponenti arasında köprü kurarsınız:
```kotlin
override fun onCreateView( /* ... */ ) {
    setContent {
        MyScreen(onNavigate = { dest -> findNavController().navigate(dest) })
    }
}
```
Alternatif olarak, NavController'ı Compose hiyerarşinizden aşağı aktarabilirsiniz. Ancak, basit fonksiyonları açığa çıkarmak çok daha yeniden kullanılabilir ve test edilebilirdir.


## Testing
### Testing the NavHost
### Testing navigation actions

## Learn more
Jetpack Navigasyon hakkında daha fazla bilgi edinmek için [Navigasyon komponentini kullanmaya başlayın](https://developer.android.com/guide/navigation/navigation-getting-started) bölümüne bakın veya [Jetpack Compose Navigation codelab](https://developer.android.com/codelabs/jetpack-compose-navigation)'ına katılın.

Uygulamanızın navigasyonunu farklı ekran boyutlarına, yönlere ve form faktörlerine uyum sağlayacak şekilde nasıl tasarlayacağınızı öğrenmek için [Responsive UI'lar için Navigasyon](https://developer.android.com/guide/topics/large-screens/navigation-for-responsive-uis) bölümüne bakın.

İç içe grafikler ve bottom navigation bar entegrasyonu gibi kavramlar da dahil olmak üzere modülerleştirilmiş bir uygulamada daha gelişmiş Navigation Compose uygulaması hakkında bilgi edinmek için [Now in Android repository](https://github.com/android/nowinandroid)sine göz atın.

### Samples
- [Jetnews sample](https://github.com/android/compose-samples/tree/main/JetNews)
- [Sunflower with Compose](https://github.com/android/sunflower/tree/main)
- [Now in Android App](https://github.com/android/nowinandroid/tree/main)

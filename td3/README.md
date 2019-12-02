# TD 3: L'Internet

## Avant de commencer

Les APIs qui nous allons utiliser exigent qu'une personne soit connectée, dans ce TD nous allons simuler que la personne est connectée, en passant un `token` dans les `headers` de nos requêtes HTTP.

- Nous allons utiliser ce site: https://android-tasks-api.herokuapp.com/api-docs/index.html
- Lisez rapidement la documentation de l'API
- Créez vous un compte directement depuis la documentation et copiez le token généré en utilisant ce JSON (celui qui est donné est incomplet 🤷‍♂️):

```json
{
  "user": {
    "firstname": "string",
    "lastname": "string",
    "email": "string",
    "password": "string",
    "password_confirmation": "string"
  }
}
```
(vous pourrez le récuperer à nouveau en vous re-loggant)

## Accèder à l'internet

Afin de communiquer avec le réseau internet (wifi, ethernet ou mobile), il faut ajouter la permission dans le fichier `AndroidManifest`

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Ajout des dépendances

Dans le fichier `app/build.gradle`, ajouter : 

```groovy
  implementation "com.squareup.retrofit2:retrofit:2.6.2"
  implementation 'com.squareup.retrofit2:converter-moshi:2.6.2'
  implementation "com.squareup.moshi:moshi:1.8.0"
  implementation "com.squareup.moshi:moshi-kotlin:1.8.0"
  implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.1.0"
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.2"
```


## HeaderFragment

Dans un premier temps vous allez afficher votre nom d'utilisateur dans le `HeaderFragment`

### Retrofit

- Vous pouvez créer un package `network` qui contiendra les classes en rapport avec les échanges réseaux
- Créer un `object` `Api` (ses membres et méthodes seront donc `static`)
- Ajoutez y les constantes qui serviront à faire les requêtes:

```kotlin
object Api {
  private const val BASE_URL = "https://android-tasks-api.herokuapp.com/api/"
  private const val TOKEN = "AJOUTEZ VOTRE TOKEN ICI !"
}
```

- Créer une instance de moshi pour parser le JSON renvoyé par le serveur:

```kotlin
object Api {
  // ...
  private val moshi = Moshi.Builder().build()
}
```

- Créer le client HTTP en ajoutant un intercepteur pour ajouter le `header` d'authentification avec votre `Token`:

```kotlin
object Api {
  // ...
  private val okHttpClient by lazy {
    OkHttpClient.Builder()
      .addInterceptor { chain ->
        val newRequest = chain.request().newBuilder()
          .addHeader("Authorization", "Bearer $TOKEN")
          .build()
        chain.proceed(newRequest)
      }
      .build()
  }
}
```

- Créer une instance de retrofit qui permettra d'implémenter les interfaces que nous allons créer ensuite:

```kotlin
object Api {
  // ...
  private val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .client(okHttpClient)
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .build()
}
```

#### Ajout du UserService

- Créez l'interface `UserService` pour requêter les infos de l'utilisateur:

```kotlin
interface UserService {
    @GET("users/info")
    suspend fun getInfo(): Response<UserInfo>
}
```

- Utilisez retrofit pour créer une implémentation de ce service (gràce aux annotations):

```kotlin
object Api {
  // ...
  val userService: UserService by lazy { retrofit.create(UserService::class.java) }
}

```

#### Moshi - UserInfo

Exemple de json renvoyé par la route `/info`:

```json
{
  "email": "email",
  "firstname": "john",
  "lastname": "doe"
}
```

Créer la `data class` `UserInfo` avec des annotations Moshi pour récupérer ces données:

```kotlin
data class UserInfo(
    @field:Json(name = "email")
    val email: String,
    @field:Json(name = "firstname")
    val firstName: String,
    @field:Json(name = "lastname")
    val lastName: String
)
```

#### Retour au fragment

- Overrider la méthode `onResume` pour y récuperer les infos de l'utilisateur et l'afficher dans le header:

```kotlin
Api.userService.getInfo()
```

La méthode getInfo étant déclarée comme `suspend`, vous aurez besoin de la lancer dans une dans un `couroutineScope`
Pour cela on peut utiliser `GlobalScope`, mais une meilleure façon est d'en créer un "vrai":

```kotlin
private val coroutineScope = MainScope()
// Pour utiliser:
coroutineScope.launch {...}
// Dans onDestroy():
coroutineScope.cancel()
```

**NB:** Une vraiment bonne façon est d'utiliser les scopes fournis par android, notamment: `viewModelScope`, mais pour l'instant on implémente tout dans le fragment comme des 🐷

## TasksFragment

Il est temps de récuperer les tâches depuis le serveur !

- Créer un nouveau service `TaskService`

```kotlin
interface TasksService {
    @GET("tasks")
    suspend fun getTasks(): Response<List<Task>>
}
```

- Utiliser l'instance de retrofit comme précédemment pour créer une instance de `TaskService` dans l'objet `Api`

- Modifier `Task` pour la rendre Moshi-compatible

### TasksRepository

Le Repository va chercher des data dans une ou plusieurs sources de données (ex: DB locale et API distante)

Créer la classe `TasksRepository`avec:
- une méthode publique `getTasks` qui renvoie des LiveData (auquel va s'abonner le fragment)
- une méthode privée `loadTasks` qui récupère la liste en asynchrone

```kotlin
class TasksRepository {
    private val tasksService = TaskApi.tasksService
	private val coroutineScope = MainScope()

    fun getTasks(): LiveData<List<Task>?> {
        val tasks = MutableLiveData<List<Task>?>()
        coroutineScope.launch { tasks.postValue(loadTasks()) }
        return tasks
    }

    private suspend fun loadTasks(): List<Task>? {
        val tasksResponse = tasksService.getTasks()
        Log.e("loadTasks", tasksResponse.toString())
        return if (tasksResponse.isSuccessful) tasksResponse.body() else null
    }
}
```

### S'abonner au LiveData

- Dans `TasksFragment`, ajouter une instance de `TasksRepository` 
- Modifier l'adapteur pour qu'il utilise une liste locale au fragment et plus la liste `static` du faux ViewModel
- Modifier également la fonction deleteTask pour que votre code compile

```kotlin
private val tasksRepository = TasksRepository()
private val tasks = mutableListOf<Task>()
private val tasksAdapter= TasksAdapter(tasks,....)
```

Dans `onCreate`, "abonnez" le fragment aux modifications des tâches et mettez à jour la liste et l'`adapter` avec le résultat:

```kotlin
tasksRepository.getTasks().observe(this, Observer {
	if (it != null) {
	  tasks.clear()
	  tasks.addAll(it)
	  tasksAdapter.notifyDataSetChanged()
	}
})
```

## Compléter TasksService

Modifier `TasksService` et ajoutez y les routes suivantes:

```kotlin
  @DELETE("tasks/{id}")
  suspend fun deleteTask(@Path("id") id: String): Response<String>

  @POST("tasks")
  suspend fun createTask(@Body task: Task): Response<Task>

  @PATCH("tasks/{id}")
  suspend fun updateTask(@Path("id") id: String, @Body task: Task): Response<Task>
```

## Suppression, Ajout et Édition d'une tâche

**NB:** Vous pouvez créer des tâches dans l'interface web, en spécifiant votre token dans avec le bouton "Authorize" en haut

- Modifier l'action lorsqu'on clique sur le bouton "supprimer" et effectuer un call réseau afin de la supprimer dans le serveur puis supprimer la dans la liste locale `tasks`

- Avant de fermer l'Activity qui permet de créer/editer des tâches, effectuer un call réseau et vérifier qu'il n'y a pas d'erreurs avant de la fermer et de réafficher l'écran des tâches

# Bonus: TasksViewModel

Mettre toute la logique dans le fragment est une trés mauvaise pratique: les `ViewModel` permettent d'extraire une partie logique du fragment.

Créer une classe `TasksViewModel` qui hérite de `ViewModel`: elle contiendra la liste des taches, l'adapteur et le Repository.

Vous pourrez la récupérer dans le fragment grâce au `ViewModelProviders`:

```kotlin
class TasksViewModel: ViewModel() {
  private val repository
  private val tasks
  val tasksAdapter
  fun loadTasks() { ... }
}

class TasksFragment: Fragment() {
  private val tasksViewModel by lazy {
    ViewModelProviders.of(this).get(TasksViewModel::class.java)
  }

  override fun onCreateView(...) {
    // ...
    view.tasks_recycler_view.adapter = tasksViewModel.tasksAdapter
  }

  override fun onResume(...) {
    // ...
    tasksViewModel.loadTasks()
  }
}
```

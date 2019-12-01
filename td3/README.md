# TD 3: Internet

## Avant de commencer
Les api qui nous allons utiliser exige qu'une personne soit connecté, dans ce td nous allons simuler que la personne est connecté, en passant dans les headers un token.

- Allez sur ce site (qui contient la documentationd de l'API): https://android-tasks-api.herokuapp.com/api-docs/index.html
- Créer vous un compte et copier le token généré, il sera utile plus tard, le json pour la création du compte, celui en exemple ne fonctionne pas...
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
- Si vous ne le copiez pas, vous pourrez toujours le récuperer en vous logguant

## Accèder à l'internet
Afin de communiquer avec le réseau internet (wifi ou 3g), il faut ajouter la permission dans le fichier `AndroidManifest`

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
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.2"
```


## HeaderFragment
Dans un premier temps nous allons afficher le nom de l'utilisateur dans le `HeaderFragment`

### Retrofit
  - Créer un nouveau package `services` qui contiendra les classes en rapport avec les échanges réseaux
  - Créer ensuite un objet `Api`, pour rappel les objets sont l'équivalent du `static`

#### Setup de l'API
- Tout d'abord, ajoutez les constantes qui nous seront utiles

```kotlin
object Api {
  private const val BASE_URL = "https://android-tasks-api.herokuapp.com/api/"
  private const val TOKEN = "ajouter votre token"
}
```

- ajoutez ensuite une instance de moshi qui va nous servir à convertir le JSON renvoyé par le serveur

```kotlin
object Api {
  // ...

  private val moshi = Moshi.Builder()
    .add(KotlinJsonAdapterFactory())
    .build()
}
```

- Le httpClient nous permet d'intercepter les requêtes et d'ajouter les `headers` d'authentification ou de faire des actions tels que loggués le retour du serveur

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

- Enfin nous allons pourvoir set une instance de retrofit qui effectuera les calls réseaux

```kotlin
Object Api {
  // ...
  private val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .client(okHttpClient)
    .addConverterFactory(MoshiConverterFactory.create(moshi))
    .build()
}
```

#### Ajout du UserService

- Ajouter le service qui va nous permettre de récupérer les infos de l'utilisateur

```kotlin
interface UserService {
    @GET("info")
    suspend fun getInfo(): Response<UserInfo>
}

object Api {
  // ...
  val userService: UserService by lazy { retrofit.create(UserService::class.java) }
}

```

#### Moshi - UserInfo
Créer la data classe UserInfo et faites en sortes qu'elle puisse être instanciable depuis du json.

Voila le json renvoyé par la route `/info`
```json
{
  "email": "email",
  "firstname": "john",
  "lastname": "doe"
}
```

#### Retour au fragment
- Dans la methode `OnResume`, déclencher le call allant récuperer les infos de l'utilisateurs et de l'afficher dans le header.

```kotlin
TaskApi.userService.getInfo()
```

La méthode getInfo étant déclarer comme `suspend`, vous aurez besoin de la lancer dans une coroutine !
```kotlin
GlobalScope.launch {
  //...
}
```



## TasksFragment

Il est temps de récuperer les tâches depuis le serveur !

- Nous allons utiliser la même instance de retrofit pour créer un nouveau service `TaskService` et déclarez le dans l'objet `Api`

```kotlin
interface TasksService {
    @GET("tasks")
    suspend fun getTasks(): Response<List<Task>>
}
```

N'oubliez pas de modifier la data class `Task`.


### TasksRepository
Le Repository est une classe qui va pourvoir aller chercher des data dans plusieurs sources de données (exemple : dans une base de donnée locale, et via une API)


Créer la classe `TasksRepository`, elle contient 2 fonctions, une fonction `getTasks` qui renvoie des LiveData (auquel va s'abonner le fragment) et une fonction `loadTasks` qui va s'excuter en Asynchrone et va récuperer la liste des taches.

```kotlin
class TasksRepository {
    private val tasksService = TaskApi.tasksService

    fun getTasks(): LiveData<List<Task>?> {
        val tasks =  MutableLiveData<List<Task>?>()
        GlobalScope.launch {
            tasks.postValue(loadTasks())
        }
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

Dans le TasksFragment, ajouter une instance de TasksRepository et modifier l'adapteur pour qu'ils prennent en parametre une liste locale au fragment et non plus la liste static du ViewModel. Modifier également la fonction deleteTask pour que votre code compile.


```kotlin
private val tasksRepository = TasksRepository()

private val tasks = mutableListOf<Task>()
private val tasksAdapter= TasksAdapter(tasks,....)
```


Enfin ajouter la méthode `fetchTasks`, cette méthode
- Récupere les tâches depuis le serveur
- Ajoute un observer sur la LiveData
- Quand la liveData notifie qu'il y a du changement, on reset la liste des taches `tasks` et notifie l'adapteur que la liste a changé


```kotlin
private fun fetchTasks() {
  tasksRepository.getTasks().observe(this, Observer {
    if (it != null) {
      tasks.clear()
      tasks.addAll(it)
      tasksAdapter.notifyDataSetChanged()
    }
  })
}
```

Vous pouvez appeler cette fonction dans le `OnResume`

### Bonus - TasksViewModel
**Ne le faites pas si vous ne le sentez pas ! N'est pas nécessaire à la suite du projet**

Mettre toute la logique dans le fragment est une trés mauvaise pratique !
Les `ViewModel` sont des classes permettant d'enlever un bout de complexité du fragment

Créer une classe `TasksViewModel` qui hérite de `ViewModel`
C'est elle qui contiendra la liste des taches, l'adapteur et le Repository.

Vous pourrez la récuprer dans le fragment grace au `ViewModelProviders`

```kotlin
class TasksViewModel: ViewModel() {
  private val repository
  private val tasks

  val tasksAdapter

  fun loadTasks(lifecycleOwner: LifecycleOwner) { ... }
}


class TasksFragment: Fragment() {
  private val tasksViewModel by lazy {
    ViewModelProviders.of(this).get(TasksViewModel::class.java)
  }

  OnCreateView(...) {
    // ...
    view.tasks_recycler_view.adapter = tasksViewModel.tasksAdapter
  }

  onResume(...) {
    // ...
    tasksViewModel.loadTasks(this)
  }
}

```



## Suppresion d'un tache

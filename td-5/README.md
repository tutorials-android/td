# TD 5: Register/Login et navigation

## Navigation dans l'AuthenticationActivity


### Ajout des dépendances

Dans le fichier `app/build.gradle`, ajouter :

```groovy
implementation "androidx.navigation:navigation-fragment-ktx:2.1.0"
implementation "androidx.navigation:navigation-ui-ktx:2.1.0"
```


### Nouvelle activity
- créer une nouvelle activity : `AuthenticationActivity`
- Ajoutez la dans l'`AndroidManifest` et déclarer comme étant le point d'entrée de votre application (ce n'est plus MainActivity)

- Le layout associé est un fragment de type `NavHostFragment`. A partir d'un graphe de navigation, le `NavHostFragment` va gerer la navigation...

```xml
<?xml version="1.0" encoding="utf-8"?>
<fragment
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_host_fragment"
    android:name="androidx.navigation.fragment.NavHostFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"

    app:defaultNavHost="true"
    app:navGraph="@navigation/nav_graph" />
```

Vous pouvez <kbd>Ctrl + enter</kbd> pour créer le fichier de navigation, nous y reviendrons plus tard.

### Les fragments
Créer 3 nouveaux fragments et leur layout:

* AuthenticationFragment
  - contient 2 buttons "Se connecter" et "S'enregistrer"

* LoginFragment
  - contient 2 `EditText`, un pour l'email, le second pour le password et un bouton "Se connecter"
  - L'attribut `android:hint` permet d'ajouter des placeholders
  - L'attribut `android:inputType` permet de gerer le type d'input (password par exemple)

* SignupFragment
 - Plusieurs `EditText`: firstname, lastname, email, password, password_confirmation
 - Un bouton "S'enregistrer"

### La navigation !

- Si le fichier `navigation/nav_graph.xml` n'existe pas créez le dans le dossier `res`


* Ajouter les 3 fragments précédents
* Définissez `AuthenticationFragment` comme `start` (la petite maison)
* Définissez ensuite les enchainements entre les fragments
  - l'`AuthenticationFragment` permet d'ouvrir les 2 autres

** Jettez un coup d'oeil à la vesion text, vous devriez remarquer 2 actions dans l'`onBoardingFragment`, ces actions vont permettre la navigation dans le code **


### AuthenticationFragment
- Dans le fragment, ajoutez des clickListener sur les 2 boutons `se connecter` et `s'enregisterr`
- A présent, vous pouvez naviguer entre les fragments grace au `NavController`

```kotlin
view.findNavController().navigate(R.id.action_onBoardingFragment_to_)
```


## Refacto de l'API
Dans l'`API`, nous allons avoir besoin d'un contexte, sauf qu'il s'agit d'un objet et que nous n'avons pas moyen de lui passer un contexte. De plus il faudrai faire de l'injection de dépendance pour pouvoir ajouter le contexte la ou on a besoin ou bien passé un contexte en argument partout..

Pour vous éviter tout ça, nous avons décidé d'introduit un leak de mémoire volontairement en transformant l'`Api` en singleton initilialisé au lancement de l'application et qui contient le context de l'application

**Ceci est exceptionnel, nous vous le deconseillons fortement dans vos projets**

- Créer une classe App
```kotlin
class App: Application() {
    override fun onCreate() {
        super.onCreate()
        Api.INSTANCE = Api(this)
    }
}

```

Dans l'AndroidManifest, ajoutez l'attribut name avec votre classe
```kotlin
    <application
        android:name="path.to.App"
        ....
        />
```

- Dans l'`Api`
```kotlin
class Api(private val context: Context) {
    companion object {
        private const val BASE_URL = "https://android-tasks-api.herokuapp.com/api/"
        private const val TOKEN = "votre token"
        lateinit var INSTANCE: Api
    }

    // le reste ne change pas
}
```


- Partout ailleurs remplacer Api par Api.INSTANCE
- votre application doit etre 100% fonctionnel, changer l'activity lancé au lancement de l'application pour vérfier


## Login
### Data class
- Créer la data class `LoginForm` qui contient un email et un password de type `String`
- Créer la data class `TokenResponse` qui contient un token de type `String`

### UserService
- Ajouter un nouveau call réseau
```kotlin
@POST("users/login")
suspend fun login(@Body user: UserLogin): Response<TokenResponse>
```


### Soumission du formulaire


## Signup

# Android: TD 2

L'objectif de ce TD est d'implementer un écran affichant une liste de tâches, de permettre de créer des nouvelles tâches, de les supprimer et de les partager dans une autre application.


## Ajout des fragments
- Cycle de vie d'une `Activity` ([Documentation][1])
- Cycle de vie d'un `Fragment` ([Documentation][2])

#### HeaderFragment
- Créer une classe `HeaderFragment` qui hérite de Fragment
- Créer le layout associé `header_fragment.xml` et ajouter une `<TextView...>` vide qui permettra d'afficher plus tard le nom de l'utilisateur
- Grace à la balise `<fragment...>`, ajouter à votre activité principale le `HeaderFragment` et afficher "Hello World" dans la TextView

#### TasksFragment
De la même façon, ajouter en dessous du Header un nouveau fragment `TasksFragment` qui va nous permettre d'afficher la liste des tâches.

#### La liste des taches
Pour commencer la liste des tâches sera simplement un tableau de string

```kotlin
private val tasks = listOf("Task 1", "Task 2", "Task 3")
```

- Dans le layout du `TasksFragment`, ajouter la balise `<RecyclerView...>`:

```xml
<androidx.recyclerview.widget.RecyclerView
  android:id="@+id/tasks_recycler_view"
  android:layout_width="match_parent"
  android:layout_height="match_parent"/>
```


- Dans `TasksFragment`, lors de l'appel de la fonction `onCreateView`, il faut initializer la `recyclerView` avec un `LinearLayoutManager` et un `adapter`

**Rappel**: l'Adapteur recycle les cellules (`ViewHolder`) pour afficher la liste de tâches (`Model`) dans la `RecyclerView` (`View`) donc correspond plus ou moins au `Controller` de l'architecture MVC

- Créer une nouvelle classe `TasksAdapter`

```kotlin
class TasksAdapter(private val tasks: List<String>) : RecyclerView.Adapter<TaskViewHolder>() {}
```

- Créer également la classe `TaskViewHolder` (il s'agit de la vue dans le design pattern MVC)

```kotlin
class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
	fun bind(task: String) {}
}
```

- Créer également un nouveau layout `item_task.xml`

```xml
<LinearLayout 
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:orientation="horizontal" 
  android:layout_width="match_parent"
  android:layout_height="wrap_content">

  <TextView
      android:id="@+id/task_title"
      android:background="@android/holo_blue_bright"
      android:layout_width="match_parent"
      android:layout_height="wrap_content" />
</LinearLayout>
```

#### Il est temps de tout connecter !

Dans le `TasksAdapter` :
- implementer la methode `getItemCount` qui renvoie la taille de la liste de tâche à afficher
- implementer la methode `onCreateViewHolder` de façon à ce qu'elle crée un nouveau `TaskViewHolder` en utilisant le layout `item_task.xml`
- Pour finir implémenter la méthode `onBindViewHolder` qui va relier la cellule (`ViewHolder`) à la donnée (ici le titre de la tâche) en fonction de la position dans la liste.

**Astuce**: Utilisez l'IDE pour faciliter l'implémentation des méthodes en cliquant sur le nom de votre classe (qui doit être soulignée en rouge) et cliquez sur l'ampoule jaune ou tapez `Alt` + `ENTER` (sinon, `CTRL` + `O` n'importe où dans la classe)

Votre code doit compiler maintenant et vous devez voir 3 tâches sur fond bleu.

#### Ajout de la data class Task

- Dans un nouveau fichier, créer la data class `Task` qui a 3 attributs, un id, un titre et une description. Ajouter une valeur par défaut à la description.
- Dans le `TasksFragment`, remplacer la valeur de `tasks` par

 ```kotlin       
private val tasks = arrayOf(
	Task(id = "id_1", title = "Task 1", description = "description 1"), 
	Task(id = "id_2", title = "Task 2"), 
	Task(id = "id_3", title = "Task 3")
)
```
        
- Corriger votre code afin qu'il compile de nouveau
- Enfin afficher la description en dessous du titre

Votre RecyclerView est prête !

## Suppression d'une tache

Dans le layout de votre ViewHolder, ajouter un bouton afin de pouvoir supprimer la tache associé. Vous pouvez utiliser l'icone `@android:drawable/ic_menu_delete`

- Transformer votre liste de taches `tasks` en `MutableList` afin de pouvoir la modifier 
- Dans l'adapteur, ajouter une lambda `onDeleteClickListener` qui prends en arguments une tache et ne renvoie rien
- Relier cette callback au `onClickListener` de l'image que vous avez ajoutée précédemment
- Dans le fragment, implementer le `onDeleteClickListener`, il doit supprimer la tache passée en argument de la liste **et notifier l'adapteur**.

### Changements de configuration
Que se passe-t-il si vous tournez votre téléphone ? 🤔

Pour régler ce problème, implémentez les méthodes suivantes:

```kotlin
override fun onSaveInstanceState(outState: Bundle)
override fun onActivityCreated(savedInstanceState: Bundle?)
```

## Création d'une nouvelle tache

#### ⚠️ Ne faites pas ça chez vous ! ⚠️

Afin de pouvoir créer et éditer les taches nous allons créer un `object`, un object est un élément ne contenant que des fonctions /attributs statiques. 
C'est une trés **mauvaise pratique** mais elle va nous permettre de mettre en place rapidement un faux `ViewModel`

Dans les prochains TDs, nous verrons comment implémenter cela proprement.

#### TaskViewModel

- Créer l'objet `TaskViewModel`
- Déplacer le `val tasks = ...` dedans
- Déplacer également le code qui supprime la tache de la liste

#### Ajout du Floating Action Button (FAB)

- Ajouter le FAB dans le layout de l'activité principale
- Dans le `onClickListener` du FAB, ajoutez la possibilité d'ouvrir l' 'activité `TaskFormActivity`
- Pour ouvrir une activité, on utilise les `Intent` et la méthode `startActivity`

#### TaskFormActivity

- Créer la nouvelle `Activity`, n'oubliez pas de la déclarer dans le manifest
- Créer un layout contenant 2 `EditText`, pour le titre et la description et un bouton pour valider
- Lorsqu'on valide le formulaire, cela ajoute dans une nouvelle tache dans la liste et ferme l'activity
- N'oubliez pas de donner un `id` à votre tache avant de l'ajouter !
- Lorsqu'on clique sur le bouton "Back", la TaskFormActivity doit se fremer: ajouter une popup de confirmation si l'utilisateur a commencer à taper des informations
- Faites en sorte que la nouvelle tache s'affiche à notre retour sur l'activité principale.

## Edition d'une tache

- De la meme maniere que vous avez ajouté le boutton supprimer, ajouter une bouton permettant d'éditer
- Au lieu de supprimer la tache, ouvrir l'activité `TaskFormActivity` pré-rempli avec les informations de la tache.
- Pour transmettre des infos d'une activité à l'autre, vous pouvez utiliser la méthode `putExtra` depuis une instance d'`Intent`
- Vous pouvez ensuite récuperer dans le onCreate de l'activité les infos que vous avez passées
- Vérifier que les infos éditées s'affichent bien à notre retour sur l'activité principale.

## Partager

- Ajouter la possibilité de partager du texte **depuis** les autres applications et ouvrir le formulaire de création de tâche pré-rempli ([Documentation][3])
- Ajouter la possibilité de partager du texte **vers** les autres applications avec un `OnLongClickListener` sur les tâches ([Documentation][4])

[1]: https://developer.android.com/guide/components/activities/activity-lifecycle#alc

[2]: https://developer.android.com/reference/androidx/fragment/app/Fragment.html

[3]: https://developer.android.com/training/sharing/receive

[4]: https://developer.android.com/training/sharing/send

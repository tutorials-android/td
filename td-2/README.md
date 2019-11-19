# TD 2 Android

L'objectif de ce td est d'implementer un écran affichant une liste de tâches, de permettre de créer des nouvelles tâches et de partager une tâche dans une autre application.


## Ajout des fragments
#### Documentation
Cycle de vie d'une activité : https://developer.android.com/guide/components/activities/activity-lifecycle#alc

Fragment : https://developer.android.com/reference/androidx/fragment/app/Fragment.html

#### HeaderFragment
- Créer un fragment `HeaderFragment` qui hérité de Fragment
- Créer le layout associé `header_fragment` et ajouter une TextView vide qui permettra d'afficher plus tard le nom de l'utilisateur
- Grace à la balise `fragment`, ajouter à votre activité principale le `HeaderFragment` et afficher "Hello World" dans la TextView

#### TasksFragment
De la même façon, ajouter en dessous du header Fragment un nouveau fragment `TasksFragment` qui va nous permettre d'afficher la liste des tâches.

#### La liste des taches
Pour commencer la liste des tâches sera simplement un tableau de string
    `private val tasks = arrayOf("Task 1", "Task 2", "Task 3")`

- Dans le layout du TasksFragment, ajouter la recyclerview

      <androidx.recyclerview.widget.RecyclerView
          android:id="@+id/tasks_recycler_view"
          android:layout_width="match_parent"
          android:layout_height="match_parent"/>

- Dans TasksFragment, lors de l'appel de la fonction `onCreateView`, il faut initializer la recyclerView avec un `LinearLayoutManager` et un adapter
- créer une nouvelle classe `TasksAdapter`, pour rappel l'adapteur fera le lien entre l'affichage et les données à afficher (il s'agit du controller dans le design pattern MVC)

      class TasksAdapter(private val tasks: Array<String>) : RecyclerView.Adapter<TaskViewHolder>()

- Créer également la classe `TaskViewHolder` (il s'agit de la vue dans le design pattern MVC)

      class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)

- Créer également un nouveau layout `task_view_holder.xml`

      <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
          android:orientation="vertical" android:layout_width="match_parent"
          android:layout_height="wrap_content">

          <TextView
              android:id="@+id/task_title"
              android:background="@android/holo_blue_bright"
              android:layout_width="match_parent"
              android:layout_height="wrap_content" />
      </LinearLayout>


#### Il est temps de tout connecter !
Dans le `TasksAdapter` :
- implementer la methode getItemCount qui renvoie la liste de tâche à afficher
- implementer la methode `onCreateViewHolder` de façon à ce qu'elle renvoie un nouveau `TaskViewHolder` qui contient le layout `task_view_holder`
- Pour finir implémenter la méthode `onBindViewHolder` qui afficher le titre de la tâche en fonction de la position du curseur.

Votre code doit compiler maintenant et vous devez 3 tâches sur fond bleu.


#### Ajout de la data class Task
- Dans un nouveau fichier, créer la data class `Task` qui a 2 attributs, un titre et une description. Ajouter une valeur par défaut à la description.
- Dans le `TasksFragment`, remplacer la valeur de `tasks` par `arrayOf(Task("Task 1", "description 1"), Task("Task 2"), Task("Task 3"))`
- Corriger votre code afin qu'il compile de nouveau
- Enfin afficher la description en dessous du titre

Votre RecyclerView est prête !

## Share Intent
A écrire

## FAB Button - création d'un nouvelle tache - LiveData
A écrire

## D'autres sujets ?
Moshi et Retrofit, semaine d'aprés plutot non ? Je preferais bien focus sur ces points la avant

## Exploration pour les plus smart : (l'un d'entre eux je ne sais pas encore lequel !)
- Faire en sorte qu'on puisse partager du texte depuis les autres app vers mon app et ouvrir le formulaire de création d'une tache.
- Mini mode offline

[lien][1]

[1]: https://developer.android.com/guide/components/activities/activity-lifecycle#alc

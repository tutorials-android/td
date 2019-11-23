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
- Dans un nouveau fichier, créer la data class `Task` qui a 3 attributs, un id, un titre et une description. Ajouter une valeur par défaut à la description.
- Dans le `TasksFragment`, remplacer la valeur de `tasks` par
        
        private val tasks = arrayOf(
            Task(id = "id_1", title = "Task 1", description = "description 1"), 
            Task(id = "id_2", title = "Task 2"), 
            Task(id="id_3", title = "Task 3")
        )
- Corriger votre code afin qu'il compile de nouveau
- Enfin afficher la description en dessous du titre

Votre RecyclerView est prête !

## Suppression d'une tache
Dans le layout de votre ViewHolder, ajouter un bouton afin de pouvoir supprimer la tache associé. Vous pouvez utiliser l'icone `@android:drawable/ic_menu_delete`

- Transformer votre liste de taches `tasks` en MutableList afin de pouvoir la modifier 
- Dans l'adapteur, ajouter le callback `onDeleteClickListener` qui prends en arguments une tache et ne renvoie rien
- Linker ce callback au onClick de l'image que vous avez ajouté précédemment
- Dans le fragment implementer le `onDeleteClickListener`, il doit supprimer la tache passée en argument de la liste et notifie l'adapteur.


- Que se passe t'il si vous tournez votre téléphone ?

## Floating Action Button (FAB) - Création d'une nouvelle tache
#### Ne faites pas ça chez vous !
Afin de pouvoir créer et éditer les taches nous allons créer un `object`, un object est un élément ne contenant que des fonctions/attributs statiques. C'est une trés **mauvaise pratique** mais a l'avantage de puvoir etre mis en place facilement.

Dans le prochain cours, nous verrons les ViewModelProviders les LiveData qui permet à plusieurs élément d'écouter et d'etre notifier de tous les changements sur un objet (comme une liste de tâche par exemple)


#### TaskViewModel
- Créer l'objet `TaskViewModel`
- Déplacer le `val tasks = ...` dedans
- Déplacer également le code qui supprime de la tache de la liste

#### Ajout du FAB
- Ajouter le FAB dans le layout de l'activité principale
- Dans l'activité principale onClick ouvrira un nouvelle activité, `TaskFormActivity`
- Pour ouvrir une activité, on utilise les `intent` et la méthode `startActivity`

#### TaskFormActivity
- Créer la nouvelle Activity, n'oubliez pas de la déclarer dans le manifest
- Créer un layout contenant 2 `EditText`, pour le titre et la description et un bouton pour valider
- Lorsqu'on valide le formulaire, cela ajoute dans une nouvelle tache dans la liste et ferme l'activity
- N'oubliez pas de set un Id à votre tache avant de l'ajouter !
- Lorsqu'on clique sur le bouton "Back", fermer la TaskFormActivity


- Faites en sorte que la nouvelle tache s'affiche à notre retour sur l'activité principale.


## Edition d'une tache
- De la meme maniere que vous avez ajouter le boutton supprimer, ajouter une bouton permettant d'éditer
- Au lieu de supprimer la tache, ouvrir l'activité `TaskFormActivity` pré-rempli avec les informations de la tache.

- Pour transmettre des infos d'une activité à l'autre, vous pouvez utiliser la méthode `putExtra` depuis une instance d'`Intent`
- Vous pouvez ensuite récuperer dans le onCreate de l'activité les infos que vous avez passer


- Vérifier que la tache éditée s'affiche à notre retour sur l'activité principale.


## Bonus - Pour ceux qui sont rapides !
- Faire en sorte qu'on puisse partager du texte depuis les autres app vers mon app et ouvrir le formulaire de création d'une tache, [Documentation][1]

- OnLongClick sur les taches permettre de partager la tache dans une autre application, [Documentation][2]

[1]: https://developer.android.com/training/sharing/receive

[2]: https://developer.android.com/training/sharing/receive

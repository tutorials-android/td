# Sujets Exploratoires

## Room :
Implémenter un cache des données serveur avec `Room`:

- Gestion offline des informations de l'utilisateur
- Gestion offline des tâches: liste, ajout, edition, supression

**Documentation:**

- [Codelab "Room with a view"](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin) (sur Room de façon générale)
- [Codelab "Repository"](https://codelabs.developers.google.com/codelabs/kotlin-android-training-repository) (sur la mise en place d'un Cache avec Room)
  


## Pagination
Utiliser une PagedList pour afficher une liste infinie de tâche de façon efficace à la place de la RecyclerView actuelle qui n'affiche que la première page retournée par le serveur 

**Documentation:** [Paging Library](https://developer.android.com/topic/libraries/architecture/paging)


## Espresso (test de votre application):
- Créer des tests UI (tests d'intégration) sur les différentes features de l'applications, par ex:
  - Affichage de la liste des tâches
  - Résultat des actions sur les boutons: ajout, suppression, etc...
  - ...
- Mocker les appels réseaux

**Documentation:**: [Espresso Testing](https://developer.android.com/training/testing/ui-testing/espresso-testing)


## Data-Binding:
Utiliser le data-binding pour afficher et éditer les données de l'application directement dans les layouts:

- Commencer par les infos affichées dans `HeaderFragment`
- Faire de même dans les `TaskViewHolder`
- Permettre d'éditer directement dans `TaskFormActivity`
    
**Documentation:** 

- [Data Binding](https://developer.android.com/topic/libraries/data-binding) (affichage)
- [Two Way Data Binding](https://developer.android.com/topic/libraries/data-binding/two-way) (édition)

## Work Manager:
Implémenter des `Worker` pour executer des tâches de fond sur les images et utiliser `WorkManager` pour les exécuter de façon efficace et avec les contraintes nécessaires (ex: réseau disponible):
  
  - Commencer par la compression et l'upload de l'image 
  - Ajouter un Worker appliquant un filtre sépia
  - Afficher dans l'app ou dans une notification l'état de progrès du travail
  
**Documentation:**

- [WorkManager: Getting Started](https://developer.android.com/topic/libraries/architecture/workmanager/basics.html)
- [CoroutineWorker](https://developer.android.com/topic/libraries/architecture/workmanager/advanced/coroutineworker)
- [Tuto RayWanderlitch](https://www.raywenderlich.com/6040-workmanager-tutorial-for-android-getting-started)
- [Article ProAndroidDev](https://proandroiddev.com/exploring-the-stable-android-jetpack-workmanager-82819d5d7c34)


## Notifications et AlarmManager: 
Ajouter une fonctionnalité "Rappel" à vos tâches:

- Récupérer dans le model `Task` la date retournée par le serveur (`due_date`)
- Dans `TaskFormActivity`, ajouter un `DatePicker` et un `TimePicker` permettant de modifier cette date
- Vous pouvez aussi utiliser `DatePickerDialog` et `TimePickerDialog`
- Utiliser `AlarmManager` pour envoyer une notification 5 minutes avant la `due_date`
- Ajouter un `PendingIntent` qui ouvre `TaskFormActivity` avec le détail de la tâche
- Ajouter une action supplémentaire `Mark as Done` dans la notification qui supprime la `Task`

**Documentation:**

- [Alarms](https://developer.android.com/training/scheduling/alarms)
- [Pickers](https://developer.android.com/guide/topics/ui/controls/pickers)
- [PickerDialog](https://www.journaldev.com/9976/android-date-time-picker-dialog)
- [Notifications](https://developer.android.com/guide/topics/ui/notifiers/notifications)

## PreferenceScreen:
Ajouter une page "Settings" en utilisant un `PreferenceScreen`:

- Ajout un OptionsMenu menu pour y acceder
- Permettre de changer l'aspect de l'app:
    - la couleur de la police 
    - La couleur de l'`AppBar`
    - le titre dans l'`AppBar`
    - ...

**Documentation:**

- [Menus](https://developer.android.com/guide/topics/ui/menus)
- [Settings](https://developer.android.com/guide/topics/ui/settings.html)


## RxJava : 
Garder les mêmes fonctionnalités (list, add, edit, remove) mais en utilisant RxJava à la place des Coroutines et des LiveData

**Documentation:** [RxJava](https://github.com/ReactiveX/RxJava)

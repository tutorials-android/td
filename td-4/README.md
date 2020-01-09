# TD 4: Téléchargement d'images et permissions !

L'objectif de cette partie va être de permettre à l'utilisateur d'uploader un avatar et de l'afficher.

## Afficher une image téléchargée depuis un serveur distant: Glide


### Ajout des dépendances

Dans le fichier `app/build.gradle`, ajouter :

```groovy
    implementation 'com.github.bumptech.glide:glide:4.10.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.10.0'
```


### Dans le headerFragment

- Ajouter une imageView à coté du nom qui va permettre d'afficher l'avatar de l'utilisateur.
- Ajouter dans le onResume :

```kotlin
Glide.with(this).load("https://goo.gl/gEgYUd").into(image_view)
```

- À partir de la documentation d'Android et de Glide: [https://github.com/bumptech/glide](https://github.com/bumptech/glide), afficher l'image sous la forme d'un cercle
- Attention l'image peut être rectangulaire, tester avec plusieurs formats image pour vérifier que votre code fonctionne correctement

## Upload d'images

### Nouvelle activité
- Créer une nouvelle activité `UserInfoActivity`
- Ajouter 2 boutons "Prendre une photo" et "Choisir une image"

```xml
    <Button
        android:id="@+id/upload_image_button"
        app:layout_constraintTop_toTopOf="parent"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Choisir une Image" />

    <Button
        android:id="@+id/take_picture_button"
        app:layout_constraintTop_toBottomOf="@+id/upload_image_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Prendre une photo" />
```

- Attention si ça ne compile pas, vérifiez que vous avez bien déclaré votre Activity dans le Manifest

### Prendre une photo

#### Les permissions
- `AndroidManifest`: ajouter la permissions `android.permission.CAMERA`
- `UserInfoActivity` : sur le `take_picture_button`, ajouter un onClickListener qui appele la méthode `askCameraPermissionAndOpenCamera`


```kotlin
private fun askCameraPermissionAndOpenCamera() {
  if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
    if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA)) {
        // l'OS dit d'expliquer pourquoi on a besoin de cette permission:
        showDialogBeforeRequest()    
    } else {
        // l'OS ne demande pas d'explication, on demande directement:
        requestCameraPermission()
    }
  } else {
    openCamera()
  }
}

private fun showDialogBeforeRequest() {
    // Affiche une popup (Dialog) d'explications: 
    AlertDialog.Builder(this).apply {
        setMessage("On a besoin de la caméra sivouplé ! 🥺")
        setPositiveButton(android.R.string.ok) { _, _ -> requestCameraPermission() }
        setCancelable(true)
        show()
    }
}

private fun requestCameraPermission() {
    // CAMERA_PERMISSION_CODE est défini par nous et sera récupéré dans onRequestPermissionsResult
    ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.CAMERA), CAMERA_PERMISSION_CODE )
}

private fun openCamera() {
    // On va utiliser un Intent implicite
}

companion object {
    const val CAMERA_PERMISSION_CODE = 42
}
```

- Prenez le temps de lire et comprendre ce pavé 🤔
- `CAMERA_PERMISSION_CODE` est un code passé à la popup de permission qui sera repassé à la fonction `onRequestPermissionsResult` aprés la décision de l'utilisateur

- Implémenter la méthode `onRequestPermissionsResult`, si l'utilisateur à donné accès à la camera `PackageManager.PERMISSION_GRANTED`, ouvrez la camera sinon vous pouvez afficher un Toast 

```kotlin
Toast.makeText(this, "We need access to your camera to take a picture :'(", Toast.LENGTH_LONG).show()
```

#### Ouvrir la camera
- Il est possible d'ouvrir des `Intent` et de récuperer des informations grâce à la fonction `startActivityForResult` qui est jumelée à la fonction `onActivityResult`



```kotlin
private fun openCamera() {
  val cameraIntent = Intent(android.provider.MediaStore.ACTION_IMAGE_CAPTURE)
  startActivityForResult(cameraIntent, CAMERA_REQUEST_CODE)
}
```

- Déclarer la constante `CAMERA_REQUEST_CODE`

```kotlin
const val CAMERA_REQUEST_CODE = 2001
```

Cette constante sera passée à la fonction `onActivityResult` une fois que l'utilisateur aura pris une photo.

- Implémenter la fonction `onActivityResult` qui appelera la fonction `handlePhotoTaken(data: Intent?)`:


```kotlin
private fun handlePhotoTaken(data: Intent?) {
  val image = data?.extras?.get("data") as? Bitmap
  val imageBody = imageToBody(image)

  // Afficher l'image

  // Plus tard : Envoie de l'avatar au serveur
}

// Vous pouvez ignorer cette fonction...
private fun imageToBody(image: Bitmap?): MultipartBody.Part? {
  val f = File(cacheDir, "tmpfile.jpg")
  f.createNewFile()
  try {
      val fos = FileOutputStream(f)
      image?.compress(Bitmap.CompressFormat.PNG, 100, fos)

      fos.flush()
      fos.close()
  } catch (e: FileNotFoundException) {
      e.printStackTrace()
  } catch (e: IOException) {
      e.printStackTrace()

  }

  val body = RequestBody.create(MediaType.parse("image/png"), f)
  return MultipartBody.Part.createFormData("avatar", f.path ,body)
}
```

- Ajouter une imageView dans la `UserInfoActivity`
- Dans la fonction `handlePhotoTaken`, afficher la photo à l'aide de Glide

#### Envoyer la photo au serveur
- Dans l'interface `UserService`, ajouter une nouvelle fonction

```kotlin
@Multipart
@PATCH("users/update_avatar")
suspend fun updateAvatar(@Part avatar: MultipartBody.Part): Response<UserInfo>
```

- Dans l'activity, appelez cette fonction pour mettre à jour le serveur avec le nouvel avatar.
- Modifier la `data class UserInfo` pour ajouter un champ `avatar` renvoyé depuis le serveur
- Enfin au chargement de l'activité, afficher l'avatar renvoyé depuis le serveur


### Rebelotte : Accès au Storage !
- Permettez à l'utilisateur d'uplaoder une image qu'il avait déjà sur son téléphone
- Ajouter dans le manifest la permission `android.permission.READ_EXTERNAL_STORAGE`



## Bonus - Édition champs utilisateurs
- Dans L'activité `UserInfoActivity`, afficher les informations nom/prénom/email
- Ajouter une boite de dialog permettant l'édition des champs (Envoyer les infos au serveur)

Documentation Dialog : [https://developer.android.com/guide/topics/ui/dialogs]()

### Route pour udpate les infos du user

```kotlin
@PATCH("users")
suspend fun updateAvatar(@Body user: UserInfo): Response<UserInfo>
```

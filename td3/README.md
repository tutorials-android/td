# TD 3: Internet

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Moshi

```kotlin
@JsonClass(generateAdapter = true)
data class Task (
      	@Json(name = "id")
      	val id: Int,
      	@Json(name = "title") 
      	val title: String,
      	@Json(name = "description") 
  		val description: String? = null
)

private val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .build()

```

## Retrofit

Create `TodoApiService`:

```kotlin
object TasksApi {

private const val BASE_URL = 
   "https://heroku.blabla.com/bloublou"
   
val tasksService: TasksService by lazy { retrofit.create(TasksService::class.java) }

private val retrofit = Retrofit.Builder()
        .client(okHttpClient)
        .baseUrl(BASE_URL)
        .addConverterFactory(MoshiConverterFactory.create(moshi))
        //.addCallAdapterFactory(CoroutineCallAdapterFactory())
        .build()
}

interface TasksService {
    @GET("tasks")
    suspend fun getTasks(): Response<List<Task>>
}

```


## Repository

```kotlin
class TasksRepository {
    private val todoService = TasksApi.tasksService

    private suspend fun getOpenTasks(): List<Task>? {
        val tasksResponse = todoService.getTasks()
        return if (tasksResponse.isSuccessful) tasksResponse.body() else null
    }
}
```

### LiveData


```kotlin

class TaskListViewModel : ViewModel() {
	private val tasksList = mutableListOf<Task>()
	private var _isRefreshing = MutableLiveData<Boolean>(false)
	val isRefreshing: LiveData<Boolean>
		get() = _isRefreshing
	private val todoRepository = TasksRepository()
	
	fun refreshTasks() {
        viewModelScope.launch {
            _isRefreshing.postValue(true)
            todoRepository.getTasks(reverseOrder, showCompleted)?.let { tasks ->
                tasksList.clear()
                tasksList.addAll(tasks)
                recyclerAdapter.notifyDataSetChanged()
            }
            _isRefreshing.postValue(false)
        }
    }
}
```

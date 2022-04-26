# Kodein

**Kodein-DI is a Dependency Injection library. It allows you to bind your business unit interfaces with their implementation and thus having each business unit being independent.**

# Getting started with Kodein-DI

**Kodein-DI is compatible with all platforms that the Kotlin language compiles to: JVM & compatible (Android), Javascript and all the Kotlin/Native targets.**

## Kodein setup

**In module app build.gradle file add dependencies:**

```kotlin
// Kodein

def kodein_version = "6.4.0"
implementation "org.kodein.di:kodein-di-generic-jvm:$kodein_version" 
implementation "org.kodein.di:kodein-di-framework-android-x:$kodein_version"
```

## Model and repository

**Let’s create some dependencies and a model that we want to load to UI through ViewModel:**

```kotlin
data class UserModel(val name: String, val surname: String)
```

```kotlin
interface UserRepository {
    fun getUserName(): UserModel
}
```

**And simple repository implementation:**

```kotlin
class InMemoryUserRepository :
    UserRepository {
    override fun getUserName(): UserModel {
        return UserModel("Chuck", "Spadina")
    }
}
```

**Kodein gives possibility to create module objects. Inside modules we can specify bindings. Let’s bind UserRepository to InMemoryUserRepository.**

**We can set one type of bindings:**

> **singleton — it will return and keep single instance**
> 
> **provider — it will return every time new instance**

## RepositoryModule.kt

```kotlin
val repositoryModule = Kodein.Module("repository") {
    bind<UserRepository>() with singleton { InMemoryUserRepository() }
}
```

**Now we have to add repository module to global kodein instance. First create App class. Make sure that App extends Applicaton and KodeinAware. Do not forget to add App to AndroidManifest.xml.**

```kotlin
class App : Application(), KodeinAware {

    override val kodein by Kodein.lazy {
        import(repositoryModule)
    }
}
```

## Binding ViewModel

**First step to bind ViewModel is preparing ViewModelFactory. ViewModelFactory will provide ViewModel instance from Kodein injector. Factory is only responsible for create new instance of the ViewModel class.**

```kotlin
class ViewModelFactory(private val injector: DKodein) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return injector.instanceOrNull<ViewModel>(tag = modelClass.simpleName) as T?
            ?: modelClass.newInstance()
    }
}
```

**After that we have to prepare some extension functions. We need to write functions that we will use to bind ViewModel and to get instance inside Fragment or Activity. There will be functions for getting ViewModel instance per Activity, per Fragment and in Fragment but in Activity scope.**

## KodeinExt.kt

```kotlin
import androidx.appcompat.app.AppCompatActivity
import androidx.fragment.app.Fragment
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import org.kodein.di.Kodein
import org.kodein.di.KodeinAware
import org.kodein.di.direct
import org.kodein.di.generic.bind
import org.kodein.di.generic.instance


inline fun <reified T : ViewModel> Kodein.Builder.bindViewModel(overrides: Boolean? = null): Kodein.Builder.TypeBinder<T> {
    return bind<T>(T::class.java.simpleName, overrides)
}

inline fun <reified VM : ViewModel, T> T.kodeinViewModel(): Lazy<VM> where T : KodeinAware, T : AppCompatActivity {
    return lazy { ViewModelProvider(this, direct.instance())[VM::class.java] }
}

inline fun <reified VM : ViewModel, T> T.kodeinViewModel(): Lazy<VM> where T : KodeinAware, T : Fragment {
    return lazy { ViewModelProvider(this, direct.instance())[VM::class.java] }
}


```

**Last step is just create module that provide ViewModel instance and get instance inside Fragment, but first change constructor of the HomeViewModel and put there UserRepository:**

```kotlin
class HomeViewModel(private val userRepository: UserRepository) : ViewModel() {

    private val _text = MutableLiveData<String>().apply {
        val user = userRepository.getUserName()
        value = "${user.name} ${user.surname}"
    }
    val text: LiveData<String> = _text
}
```

## ViewModelModule.kt

```kotlin
val viewModelModule = Kodein.Module("viewModel") {
    bindViewModel<HomeViewModel>() with provider {
        HomeViewModel(
            instance()
        )
    }
}
```

**bindViewModel will provide new instance of HomeViewModel and instance() method will inject here implementation of the UserRepository from RepositoryModule. Do not forget to add viewModelModule to kodein instance in the App class.**

## App.kt

```kotlin
class App : Application(), KodeinAware {

    override val kodein by Kodein.lazy {
        import(repositoryModule)
        import(viewModelModule)

        bind<ViewModelProvider.Factory>() with singleton { ViewModelFactory(kodein.direct) }

    }

}
```

**The last thing is to use kodein in HomeFragment and inject here HomeViewModel. We use here activityScopedFragmentViewModel method to let our ViewModel lives even when fragment is destroyed. So the final version of HomeFragment should be kodeinAware and it should look like below:**

```kotlin
class HomeFragment : Fragment(), KodeinAware {

    override val kodein: Kodein by closestKodein()

    private val homeViewModel: HomeViewModel by kodeinViewModel()

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val root = inflater.inflate(R.layout.fragment_home, container, false)
        val textView: TextView = root.findViewById(R.id.text_home)
        homeViewModel.text.observe(viewLifecycleOwner, Observer {
            textView.text = it
        })
        return root
    }
}
```

**To sum up — in the App we can use:**

- **by activityViewModel to provide instance per Activity**
- **by fragmentViewModel to provide instance per Fragment**
- **by activityScopedFragmentViewModel to provide instance inside fragment that lives as long as parent Activity is not destroyed**

**if you don't want to create module, you can create instance directly in App.kt, for example:**

```kotlin
class UserApplication : Application(), KodeinAware {
    override val kodein: Kodein = Kodein.lazy {

        import(androidXModule(this@UserApplication))
        bind() from singleton { UserDatabase(instance()) }
        bind() from singleton { UserLocalDataSource(instance()) }
        bind() from singleton { UserRepository(instance()) }
        bind<ViewModelProvider.Factory>() with singleton { ViewModelFactory(kodein.direct) }
        bindViewModel<UserViewModel>() with provider {
            UserViewModel(instance())
        }

 }


}
```

## ViewModel

```kotlin
class AddFragment : Fragment(), KodeinAware {

    private lateinit var binding: FragmentAddBinding
    private lateinit var viewModel: UserViewModel
    override val kodein by closestKodein()

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        binding = FragmentAddBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

 

    }


```

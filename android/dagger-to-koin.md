# Dagger2 -> Koin

안드로이드에서는 DI를 위해서 Dagger를 많이 사용합니다.

컴파일 타임에 오류를 검증하거나, 강력한 모듈/컴포넌트 기능 등의 이유로 지금까지도 많은 앱들이 대거를 사용하고 있지만, 높은 진입 장벽과 많은 Boilerplate 제대로 사용하기 어렵다는 단점이 있죠.

최근에 ViewModel에서 constructor injection을 받기 위해서 Dagger 관련한 수 많은 삽질을 거치면서, Dagger 말고 다른 대체제가 없을까 생각해보다가, Koin에 생각이 미쳤습니다.

처음 알았을 때만 해도 버전이 0.8대였는데, 어느새 2.0 RC까지 나와있네요. 당시에 쓸 대도 간결하고 Kotlin 친화적인 코드를 작성할 수 있다는 점에 굉장히 마음에 들었던 라이브러리인데, 이번 기회에 Koin을 한 번 써봐야겠다는 생각이 들었습니다.



#### Adding Dependencies

https://insert-koin.io/docs/2.0/setup/



### 모듈 선언

##### Dagger2

```kotlin
@Module
class AppModule {
    @Singleton
    @Provides
    fun provideAuthenticationRepo(
        service: AuthenticationService,
        database: AuthenticationDatabase
    ): AuthenticationRepo {
        return AuthenticationRepoImpl(service, database)
    }

    @Provides
    fun provideProjection(): Projection {
        return ProjectionImpl()
    }
}
```



##### Koin

```kotlin
val repositoryModule = module {
    single<AuthenticationRepo> {
        AuthenticationRepoImpl.instance(get(),get())
    }

    factory<Projection> {
        return ProjectionImpl()
    }
}
```

Koin은 모듈 선언부터 간단합니다.

- `single` - 싱글턴입니다. 요청할 때마다 같은 객체를 반환합니다.
- `factory` - 요청할 때마다 매 번 생성합니다.



모듈 선언 도중 필요한 Dependency가 있으면 get()으로 요청하면 되죠.



#### Koin: Disadvantages

Koin은 컴파일 타임이 아니라 런타임에 의존성을 주입하기 때문에 JVM의 Type erasure와 연관된 문제들이 있습니다.



```kotlin
@Module
class SubjectModule {
    @Singleton
    @Provides
    fun provideQuestionSubject(): PublishSubject<List<Question>> = PublishSubject.create()

    @Singleton
    @Provides
    fun providePassageSubject(): PublishSubject<List<Passage>> = PublishSubject.create()
}
```

Dagger에서는 위와 같이 PublishSubject라는 한 객체에 대해 Type parameter에 따라서 선언하는 것이 가능했지만, Koin에서는 이것이 불가능합니다.



대신 `named()` Qualifier를 사용해서 이름을 지정해서 선언하는 방법을 취해야 합니다.

```kotlin
val subjectModule = module {
    single(named("question")) { PublishSubject.create<List<Question>>() }
    single(named("passage")) { PublishSubject.create<List<Passage>>() }
}
```



이렇게 선언한 모듈은 다음과 같이 사용할 수 있습니다.

```kotlin
private val passageStream: PublishSubject<List<Passage>> by inject(named("passage"))

private val questionStream: PublishSubject<List<Question>> by inject(named("question"))
```



문제가 있습니다..

```kotlin
single(named("section")) { PublishSubject.create<SectionData>() }
single(named("question")) { PublishSubject.create<List<QuestionData>>() }
single(named("passage")) { PublishSubject.create<List<Passage>>() }
```

위와 같이 선언한 뒤, 아래와 같이 사용해도 에러가 나지 않습니다.

```kotlin
private val sectionStream: PublishSubject<SectionModel> by inject(named("section"))
private val questionStream: PublishSubject<List<QuestionModel>> by inject(named("question"))
private val passageStream: PublishSubject<List<PassageModel>> by inject(named("passage"))
```

이것도 역시 런타임 시간에 바인딩을 한다라는 점 때문에 발생하는 문제겠죠..








### 사용 선언

Dagger로 따졌을 때 컴포넌트를 정의하는 부분 역시, 간단하게 작성할 수 있습니다.

```
@Singleton
@Component(
    modules = [
        ApiModule::class,
        ProjectionModule::class,
        DatabaseModule::class,
        RepositoryModule::class
    ]
)
interface AppComponent {

    fun inject(a: SignInActivity)
    fun inject(a: MainActivity)
    fun inject(a: PaperActivity)

    @Component.Builder
    interface Builder {
        fun build(): AppComponent
        fun databaseModule(databaseModule: DatabaseModule): Builder
    }
}
```
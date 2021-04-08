Important: When using Activities, inject Dagger in the Activity's onCreate method before calling super.onCreate to avoid issues with fragment restoration. In super.onCreate, an Activity during the restore phase will attach fragments that might want to access activity bindings.
Important - Best practices
- An Activity injects Dagger in the onCreate method before calling super.
- A Fragment injects Dagger in the onAttach method after calling super.



For AppComponent, we can use the @Singleton scope annotation that is the only scope annotation that comes with the javax.inject package. If we annotate a Component with @Singleton, all the classes also annotated with @Singleton will be scoped to its lifetime.


 Subcomponents are components that inherit and extend the object graph of a parent component. Thus, all objects provided in the parent component will be provided in the subcomponent too. In this way, an object from a subcomponent can depend on an object provided by the parent component.

 There are two different ways to interact with the Dagger graph:
- Declaring a function that returns Unit and takes a class as a parameter allows field injection in that class (e.g. fun inject(activity: MainActivity)).
- Declaring a function that returns a type allows retrieving types from the graph (e.g. fun registrationComponent(): RegistrationComponent.Factory).


In this case, we could call this scope @RegistrationScope but this is not a good practice. The scope annotation's name should not be explicit to the purpose it fulfills. It should be named depending on the lifetime it has since annotations can be reused by sibling Components (e.g. LoginComponent, SettingsComponent, etc). That's why instead of calling it @RegistrationScope, we call it @ActivityScope.


Scoping rules:
    - When a type is marked with a scope annotation, it can only be used by Components that are annotated with the same scope.
    - When a Component is marked with a scope annotation, it can only provide types with that annotation or types that have no annotation.
    - A subcomponent cannot use a scope annotation used by one of its parent Components.

Components also involve subcomponents in this context.

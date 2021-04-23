
 Subcomponents are components that inherit and extend the object graph of a parent component. Thus, all objects provided in the parent component will be provided in the subcomponent too. In this way, an object from a subcomponent can depend on an object provided by the parent component.

In this case, we could call this scope @RegistrationScope but this is not a good practice. The scope annotation's name should not be explicit to the purpose it fulfills. It should be named depending on the lifetime it has since annotations can be reused by sibling Components (e.g. LoginComponent, SettingsComponent, etc). That's why instead of calling it @RegistrationScope, we call it @ActivityScope.


Scoping rules:
    - When a type is marked with a scope annotation, it can only be used by Components that are annotated with the same scope.
    - When a Component is marked with a scope annotation, it can only provide types with that annotation or types that have no annotation.
    - A subcomponent cannot use a scope annotation used by one of its parent Components.

Components also involve subcomponents in this context.


Best practices summary
Note: If you're already familiar with Dagger, check out these best practices. If not, read the content on this page and come back to this later.

    Use constructor injection with @Inject to add types to the Dagger graph whenever it's possible. When it's not:
        Use @Binds to tell Dagger which implementation an interface should have.
        Use @Provides to tell Dagger how to provide classes that your project doesn't own.
    You should only declare modules once in a component.
    Name the scope annotations depending on the lifetime where the annotation is used. Examples include @ApplicationScope, @LoggedUserScope, and @ActivityScope.

    368454985
    23230185
    24230836

# Feature Flag

At a basic level, feature flags take the form of simple conditionals (e.g., if-else statements) that determine the code path that will be executed.  

- Feature flags change the traditional deployment workflow by decoupling deploy and release, allowing new code to exist in a production deploy but not be executed
- Feature flags can be used via a configuration file, often created in environment variables, a markup language like YAML, or configuration values in an external service like AWS AppConfig
- It is a best practice for development teams to regularly assign a day, whether monthly or quarterly, for flag cleanup.
---
title: "Feature flags"
date: "2020-10-02"
layout: post
---

Are you a developer working on an incomplete feature and you have to release it for CI/CD? Or do you require the option of a quick feature rollback if something unexpected happens? Or do you prefer to experiment with a feature to a smaller set of users before making it available to everyone? Then a feature flag is your friend. But you need to be careful about how you use a feature flag in the code, else it can become a nightmare to manage them.

## Usage of feature flag conditions

In simple terms, a feature flag check is an if condition. This condition will decide the code flow as per the feature status (on or off). But sometimes, I have observed developers abusing this if condition, making it complex. For e.g. a feature can have multiple entry points and some developer can add this feature check condition to all the entry points. This will make your code look like a spaghetti of if conditions. Also, when you are extending this feature, you might miss wrapping this check to the extended part.

Let us assume that we are working on caching feature and want to release it behind a feature flag. The caching feature has 2 methods: put and get. With no feature flag, it should look like this.

![Cache use case](/assets/feature-flag/flow-diagram.png)


And we can write the feature controlled code like this:

```java
public class MyCacheImplementation<T> implements Cache<T> {

  public Optional<T> get(String key) {
    if(isCacheFeatureEnabled()) {
      return getValueFromCache(key);
    } else {
      return Optional.empty();
    }
  }

  public void put(String key, T value) {
    if(isCacheFeatureEnabled()) {
      return putInCache(key, value);
    }
  }
}
```

### What is wrong in this approach?

These are few issues in this approach:

- The cache implementation code needs to know about feature enablement service, which is unnecessary. MyCacheImplementation class is violating the [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) by doing caching and feature check work.
-  If you are adding a new method in the implementation, then you have to make sure you are adding the "if condition" to check cache feature is enable or not. Forgetting to add that can lead to some unwanted behavior of the code. 
- Lots of duplicate code as we have the same "if condition" in multiple places. It makes it harder to remove the feature flag code after the expiration of feature flag. 
- We are also violating the [open-closed principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle). If we have to change the feature flag related code, then we might have to make the changes at multiple places.
- The [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) of all the methods is higher because that "if condition" is creating 2 outcomes for each method.

### How to code it better?

Use polymorphism to control the behavior according to the feature state. Instead of keeping this check in the MyCacheImplementation class, let us move this to the place where we are injecting MyCacheImplementation class as a dependency.

```java
public class CacheFactory {

  public Cache<T> getCacheImplementation(){
    if(isCacheFeatureEnabled()) {
      return new MyCacheImplementation();
    } else {
      return new NullCacheImplemenation();
    }
  }

  private static class NullCacheImplementation<T> implements Cache<T> {
    
    public Optional<T> get(String key) {
      return Optional.empty();
    }

    public void put(String key, T value) {
      return;
    }
  }
}
```

Using this code, we can remove all the if conditions from the MyCacheImplementation class. It also makes it easier to clean up after the feature flag expiry. Once the feature flag expires, we can just remove NullCacheImplementation class. 

## Lifecycle of a feature flag

Whenever you add a feature flag, make sure you manage its life cycle. Mainly following phases:

- Addition of feature flag in the code
- Experimentation with the feature flag by changing its state
- Take a decision about the feature
- Removal of the feature flag from the code

It is never a good idea to keep the flag in the code for a long duration. As time goes on, it will become harder to remember which flag controls what. Every new flag state will also increase the number of combinations in which a user can use the product. New team members will have trouble in understanding about how the code executes. They will have to know about which flag they need to enable and which one to disable to mimic some use-case. 
There could be a few scenarios in which you would like to keep the flags in your system. Try to minimize such scenarios.

## Disabled by default

Always code your feature flag check service in such a way that if developers do not configure the feature flag in an environment, then it should consider that feature as disabled. Follow the always production ready mindset. Err on the side of disabled feature. You should enable it strictly to the environment where you want to release the feature. This will protect you from accidently launching an incomplete feature on production.

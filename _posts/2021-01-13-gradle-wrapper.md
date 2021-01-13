---
title: "The Gradle Wrapper"
---

# My usage of Gradle: The Gradle Wrapper

Gradle is great but comes with a lot of flexibility.
I have started to take some notes about the way I am using it.
I apply those conventions to all my projects.
I am not a gradle expert, so I am happy to discuss my patterns in the [issues](https://github.com/j2r2b/j2r2b.github.io/issues) of this blog.

## The Gradle Wrapper

See the official doc: [the Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)

### Definition in the root build file

I have decided to put the version I am using in my root `build.gradle` file:

```
wrapper {
    gradleVersion = 'a.b.c'
    // distributionType = Wrapper.DistributionType.ALL
}
```

Most often I do not need the `ALL` distribution type, even if this is suggested by some IDEs.

### Update the wrapper version

My how-to:

* Change the value of the `gradleVersion` attribute.
* Run `./gradlew wrapper` to update the jar. (can be `gradle` if this is the first time the wrapper is installed)
* Run `./gradlew wrapper` a second time.

The second run is necessary because at this point in time the targeted gradle version will be used, meaning that the scripts `gradlew` and `gradlew.bat` will be generated with this version of Gradle.

### Alias for the terminal

To not have to type `./gradlew` each time I need gradle, I have this entry in my `~/.bash_profile` file:

```
alias gw='./gradlew'
```

### GitHub action

As recommended on the documentation page, for my public repositories on GitHub I have activated the [GitHub action](https://github.com/marketplace/actions/gradle-wrapper-validation) to verify the integrity of the wrapper.

This project reproduces a Gradle bug.

```
$ ./gradlew :p1:shadowJar
Selecting project :shared(onePrefRuntimeElements) from [project :shared(onePrefRuntimeElements), project :shared(oneRuntimeElements)]
Selecting project :shared(twoPrefRuntimeElements) from [project :shared(twoPrefRuntimeElements), project :shared(twoRuntimeElements)]

FAILURE: Build failed with an exception.

* What went wrong:
Could not determine the dependencies of task ':p1:shadowJar'.
> Could not resolve all dependencies for configuration ':p1:runtimeClasspath'.
   > Problems reading data from Binary store in /private/var/folders/wc/1z_z159n7nxf31gbbnmldcmc0000gn/T/gradle589409329810110249.bin offset 203 exists? true
```

## Setup
- `shared` exposes four variants: `{one,two}{,Pref}`.
- `one` and `onePref` share a common capability `o:n:e`. `two` and `twoPref` share `t:w:o`.
- Each variant also has its own unique capability that looks like `g:one-preferred:v`.
- `twoPref` depends on `onePref`, and `two` depends on `one`. **This repro is dependent on the direction/order of this dependency**
- `p1` consumes the two "preferred" capabilities: `{one,two}Pref`.
- `p2` consumes the two "non-preferred" capabilities: `{one,two}`.
- `p1` also has a dependency on `p2`.
- `p1` has a resolution strategy that chooses the "preferred" variants whenever a capability conflict occurs.

## Notes
- The direction of `shared`'s intra-project dependency matters. If you flip the dependency to be `one->two` it all works.
- The `printRuntimeClasspath` task succeeds, however it prints both `one-preferred.jar` and `one.jar`, suggesting an inconsistent graph (they both provide `o:n:e`).
- The shadow plugin is just how I discovered this bug, there may be other ways to reproduce.

## Environment
- Reproduces on Gradle nightly `6.8-20201005220043+0000` and `6.6.1`.
- Shadow plugin `6.0.0`
- macOS
- java 8

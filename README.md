# Java Buildpack Memory Calculator

The Java buildpack memory calculator determines values for JVM memory options with the goal of enabling
an application to perform well while not exceeding the total memory available in a container
(which results in the application being killed).

The buildpack provides the following inputs to the memory calculator:
* the total memory available to the application,
* an estimate of the number of threads that will be used by the application,
* an estimate of the number of classes that will be loaded,
* the type of JVM pool used in the calculation ('permgen' for Java 7 and 'metaspace' for Java 8 and later),
* any JVM options specified by the user.

The memory calculator prints the JVM memory option settings described below, _excluding_ any the user has specified,
which are assumed to be correct.

For Java 8 and later, the memory calculator sets the maximum metaspace size (`-XX:MaxMetaspaceSize`)
and compressed class space size (`-XX:CompressedClassSpaceSize`) based on the number of classes that will be
loaded and sets the reserved code cache size (`-XX:ReservedCodeCacheSize`) to 240 Mb.

For Java 7, it sets the maximum permanent generation size (`-XX:MaxPermSize`) based on the
number of classes that will be loaded and sets the reserved code cache size (`-XX:ReservedCodeCacheSize`)
to 48 Mb.

It sets the maximum direct memory size (`-XX:MaxDirectMemorySize`) to 10 Mb.

It sets the stack size (`-Xss`) to a default value (unless the user has specified the stack size) and then
calculates the amount of memory that will be consumed by the application's thread stacks.

Finally, it sets the heap size (`-Xmx`) to total memory minus the above values.

If the values need to be adjusted, the user can either increase the total memory available
or set one or more JVM memory options to suitable values. Unless the user specifies the heap size (`-Xmx`),
increasing the total memory available results in the heap size setting increasing by the additional total memory.
Similarly, changing the value of other options affects the heap size. For example, if the user increases the
maximum direct memory size from its default value of 10 Mb to 20 Mb, then this will reduce the calculated heap
size by 10 Mb.

If the estimated number of threads or loaded classes needs to be modified, this can be achieved by configuring
the buildpack. For example, when the OpenJDK JRE is used, the number of threads can be modified as in
the following example:

```bash
$ cf set-env my-application JBP_CONFIG_OPEN_JDK_JRE '{ memory_calculator: { stack_threads: 200 } }'
```

and the number of loaded classes can be modified as in the following example:
```bash
$ cf set-env my-application JBP_CONFIG_OPEN_JDK_JRE '{ memory_calculator: { class_count: 1000 } }'
```

Please consult the [Java Buildpack][] documentation for up to date configuration information.

The document [Java Buildpack Memory Calculator v3][] provides some rationale for the memory calculator externals.

### Getting started

[Install Go][] and then `get` the memory calculator (in the Go source tree).

We run our tests with [Ginkgo/Gomega][] and manage dependencies with [Godep][].
Ginkgo is one of the dependencies we manage, so get Godep before starting work:

```shell
go get -v github.com/cloudfoundry/java-buildpack-memory-calculator
cd src/github.com/cloudfoundry/java-buildpack-memory-calculator

go get -v github.com/tools/godep
```

(The `-v` options on `go get` are there so you can see what packages are compiled under the covers.)

The (bash) script `ci/test.sh` uses (the correct version of) Ginkgo to
run the tests (using the correct versions of the dependencies). `test.sh`
will recompile Ginkgo if necessary.

The parameters to `runTests` are passed directly to Ginkgo.  For example:

```shell
ci/test.sh -r=false memory
```

will run the tests in the memory subdirectory *without* recursion into lower
subdirectories (which is the default).

The current Go environment is not modified by `test.sh`.

### Development

To develop against the code, you should issue:

```shell
godep restore
```
in the project directory before building or running tests directly from the command line.

If you wish to develop against a particular tagged *version* then, in the
project directory, you need to checkout this version (using
`git checkout <tag>`) and re-issue `godep restore` before proceeding.

If `godep restore` fails, it is because one of the dependencies cannot be
obtained, or else it cannot be (re)set to the version this project depends on.
Normally `go get -u <project>` for the dependency in error will then allow
`godep restore` to complete normally.

### Release binaries

The executables are built for more than one platform, so the Go compiler must exist
for the target platforms we need (currently linux and darwin). The shell script (`ci/build.sh`)
will use the Go compiler with the `GOOS` environment variable to generate the executables.

This will not work if the Go installation doesn't support all these platforms, so you may have to
ensure Go is installed with cross-compiler support.

## License

The Java Buildpack Memory Calculator is Open Source software released under the
[Apache 2.0 license][].

[Java Buildpack] https://github.com/cloudfoundry/java-buildpack
[Java Buildpack Memory Calculator v3] https://docs.google.com/document/d/1vlXBiwRIjwiVcbvUGYMrxx2Aw1RVAtxq3iuZ3UK2vXA/edit?usp=sharing
[Install Go]: http://golang.org/doc/install
[Godep]: http://github.com/tools/godep
[Ginkgo/Gomega]: http://github.com/onsi/ginkgo
[Apache 2.0 license]: http://www.apache.org/licenses/LICENSE-2.0.html
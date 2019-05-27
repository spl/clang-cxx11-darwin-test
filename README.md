This repository demonstrates a problem when running `clang++` and
`--target=x86_64-apple-darwin` on older versions of macOS using [Travis-CI] to
run the same command on all of the [supported macOS versions].

[Travis-CI]: https://travis-ci.com/spl/clang-cxx11-darwin-test/jobs/203303926
[macOS versions]: https://docs.travis-ci.com/user/reference/osx/#macos-version

The issue is that when you run `clang` in the way shown for Xcode version 9.4.1
(Apple LLVM version 9.1.0, clang-902.0.39.2) and less, you get the following
error:

```
$ clang++ unique_ptr.cpp -stdlib=libc++ --target=x86_64-apple-darwin
error: invalid deployment target for -stdlib=libc++ (requires OS X 10.7 or later)
```

The argument `-stdlib=libc++` tells `clang` to [use `libc++`] instead of
`libstd++`, which is needed for `clang`'s support of C++11. In newer versions of
`clang`, `libc++` is the default, but `-stdlib=libc++` is needed for older
versions.

[use `libc++`]: https://libcxx.llvm.org/docs/UsingLibcxx.html

Solutions include:

* Don't use the `--target` flag.
* Use `--target=x86_64-apple-darwin$(uname -r)`, which adds the host Darwin
  version (`uname -r`) to the target name.

To my limited knowledge, this issue was first posted at [pantsbuild/pants#4975]
and led to changes at [alexcrichton/cmake-rs#38] and [boringssl#21984].

[pantsbuild/pants#4975]: https://github.com/pantsbuild/pants/issues/4975
[alexcrichton/cmake-rs#38]: https://github.com/alexcrichton/cmake-rs/issues/38
[boringssl#21984]: https://boringssl-review.googlesource.com/c/boringssl/+/21984

The Travis-CI [build #5] for this repository shows the results of running the
following 3 commands:

[build #5]: https://travis-ci.com/spl/clang-cxx11-darwin-test/builds/113311182

```
$ clang++ unique_ptr.cpp -stdlib=libc++
$ clang++ unique_ptr.cpp -stdlib=libc++ --target=x86_64-apple-darwin$(uname -r)
$ clang++ unique_ptr.cpp -stdlib=libc++ --target=x86_64-apple-darwin
```

The first two commands pass for all versions of Xcode support, but the third
command fails for `xcode9.4` and below.

Because this project uses `cgo` and tries to cross-compile for linux/windows/macos, you'll need a
windows compiler. I've only tried this on Debian, but since go1.11, you just need mingw installed.

```sh
sudo apt install gcc-mingw-w64 binutils-mingw-w64-x86-64 tmux socat
go get github.com/audibleblink/gorsh/...
GOOS=windows go get github.com/audibleblink/gorsh/...
cd $GOPATH/src/github.com/audibleblink/gorsh
```

While it is often required during cross-compilation to set variables like $CC, $CXX, $AS, $LD, ...
it is not required as go1.11 linux/amd64 picks up on the presence of the toolchain it needs.

## Troubleshooting

Understanding issues related to building (USER CAN IGNORE THIS)

## Build problems

#### Not enabling CGO

If you are experiencing build problems, specifically this error:

```
# github.com/valyala/gozstd
../src/github.com/valyala/gozstd/stream.go:31:59: undefined: CDict
../src/github.com/valyala/gozstd/stream.go:35:64: undefined: CDict
../src/github.com/valyala/gozstd/stream.go:47:20: undefined: Writer
```

or

```
# github.com/valyala/gozstd
../src/github.com/valyala/gozstd/stream.go:31:59: undefined: CDict
../src/github.com/valyala/gozstd/stream.go:35:64: undefined: CDict
../src/github.com/valyala/gozstd/stream.go:47:20: undefined: Writer
```

The reason is because you need `CGO_ENABLED=1` before your command. Simple fix.

#### 32/64 issue when compiling on native architecture for two word register sizes

If you are seeing the following:

```
# github.com/valyala/gozstd
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_common.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_compress.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_double_fast.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_fast.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_lazy.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_ldm.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_opt.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zstd_decompress.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(zdict.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(entropy_common.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(error_private.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(fse_decompress.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(xxhash.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(fse_compress.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(hist.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(huf_compress.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(huf_decompress.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(cover.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(divsufsort.o)' is incompatible with i386 output
/usr/bin/ld: i386:x86-64 architecture of input file `/home/debian/goprojects/src/github.com/valyala/gozstd/libzstd_linux.a(pool.o)' is incompatible with i386 output
collect2: error: ld returned 1 exit status
Makefile:43: recipe for target 'linux32' failed
make: *** [linux32] Error 2
```

This is a result of installing a golang module that previously built a static library from native code using a different GOARCH but same GOOS. THe solution is
to run `go clean -i github.com/valyala/gozstd/` to remove it. This dependency should be removed and retrieved again with GOOARC and GOOS set, otherwise
it will continuously be incompatible with the host. Unfortunately, this doesn't seem to work very well. See the TODO section at the bootom.

#### Not setting CC to the correct mingw compiler for your system

If you are seeing this, specifically for the Windows target(s):

```
$ make windows64
CGO_ENABLED=1 GOOS=windows GOARCH=amd64 go build --ldflags "-w -X main.connectString=: -X main.fingerPrint=$(openssl x509 -fingerprint -sha256 -noout -in local/server.pem | cut -d '=' -f2) -H=windowsgui" -o binaries/windows/gorsh64.exe gorsh.go
# runtime/cgo
gcc: error: unrecognized command line option ‘-mthreads’; did you mean ‘-pthread’?
Makefile:43: recipe for target 'windows64' failed
make: *** [windows64] Error 2
```

The solution is to set CC in your environment to the correct mingw 64bit-64bit gcc-posix compiler. For example, on a native 64-bit system running Debian:

`export CC=/usr/bin/x86_64-w64-mingw32-gcc-6.3-posix`

After this, `make windows64` should work just fine

## Developer Notes

### Issue #1 - Ensuring appropriate mingw compilers are available on your system

DEBIAN: `sudo apt-get install -y gcc-mingw-w64 g++-mingw-w64 binutils-mingw-w64-x86-64` 

REDHAT: `sudo yum install mingw64-gcc mingw64-g+++ mingw64-binutils` 

ARCH: 	`sudo packman -S mingw-w64-gcc mingw-w64-g++ mingw-w64-binutils`

### Issue #2 - Multilib builds of GCC are problematic when building on native host

You will notice that on a system with a multilib-enabled toolchain, `make linux64 && make linux32`
will succeed on the first make but fail on the second make, unless the GOOS is different. This is
because a library requiring native-c compilating was built using the hosts native architecture, and
`go get` and/or `go build` did not respect `GOARCH=386`

This is not a huge problem, but it is an annoyance when having to build packages often.

Aside from not running a multilib-capable compiler on your system (this doesn't mean necessarily
you won't have a multilib system), the solution may be to maintain a pair of dedicated
(non-multilib) compilers, and ensure that the `go get` or `go build` process uses those.
Alternately, look into how one can pass CFLAGS during `go get` or `go build` of an external
package. 

While maintaining toolchains sounds like an awful lot of work, it isn't terrible if you utilize
https://githug.com/richfelker/musl-cross-make and some pre-built toolchain config.mak files for
forcing 32 bit builds on 64 bit hosts. Placing [this
activate](https://github.com/mzpqnxow/gdb-static-cross/blob/44bddbd2f1fe3bdc41d401d754202aab67e7c3f4/activate-script-helpers/activate-musl-toolchain.env)
file in the root of each resulting musl toolchain and sourcing it before building for the 32-bit
architecture should more or less solve the problem. When building the i386 toolchain, ensure that
the config.mak is not enabled for multilib! An example config.mak is available [here](

An issue and/or PR will be soon to follow if there is interest in fixing this "hidden" problem. It
may be very rare in practice that *any* 32-bit binaries for *any* architecture are required. On the
other hand, a 32-bit executable might throw off some analysis engines attempting to identify the
project while deplpoyed/in-use.

# c-sky toolchains built

You could find x86_64-linux-gnu, mingw, even i686-linux-gnu toolchains in [upstream](https://occ.t-head.cn/community/download?id=3885366095506644992) (See also [this AUR](https://aur.archlinux.org/packages/csky-toolchain-bin))
However, I could not find any prebuilt for `aarch64-linux-gnu` (Arm64 Linux) and `apple-aarch64-darwin` (Arm64 macOS i.e. M1/M2 series) anywhere on the Internet. So I built them by myself.

You could find the prebuilt toolchains in [releases](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.1).

## How to build

If you want to build the toolchains by yourself, here are some tips.

### Linux

Pretty straightforward. Just follow [c-sky/toolchain-build](https://github.com/c-sky/toolchain-build) make sure you have enough disk space (at least 80GB) and RAM (at least 8GB).

```bash
git clone https://github.com/c-sky/toolchain-build --recursive --depth=1
# if you miss the --recursive flag
# git submodule update --init
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
# I would say you should use jobs=$(nproc)-2 if you don't want your computer to freeze
# especially when you are using a desktop environment
./build-csky-gcc.py csky-gcc --src ./ --triple csky-unknown-elf --jobs=-1
```

When the build is done, you could find the toolchains in `build-gcc-csky-unknown-elf/Xuantie-800-gcc-elf-newlib-x86_64`. The build script just assumed you are using `x86_64` and don't even bother to check your actual architecture. It's just a name anyway.

### MinGW

I were using [ArchLinux](https://archlinux.org) to cross compile this. Follow the steps like [Windows: Cross-compilation using Linux](https://www.wireshark.org/docs/wsdg_html_chunked/ChSetupCross.html).

```bash
pacman -S mingw-w64
# since it's building from source, it might take a while
paru -S mingw-w64-libmpc mingw-w64-mpfr mingw-w64-gmp
```

You'll find `/usr/i686-w64-mingw32` and the cross compiler should be able pick up the dependencies from it.

```
cd toolchain-build
./build-csky-gcc.py csky-gcc --src ./ --triple csky-unknown-elf --jobs -1 --host mingw
```

### macOS

A few dependencies are required though.

```bash
# use the real gcc instead of the apple clang
# brew install gcc
export CC=gcc-13
export CXX=g++-13

# if binutils from homebrew is installed, uninstall or unlink it
# brew uninstall binutils
brew unlink binutils

# Building GCC requires GMP 4.2+, MPFR 3.1.0+ and MPC 0.8.0+
brew install gmp mpfr libmpc
```

You would have to do some modifications to the build script, luckily it has `--fake` flag.

```bash
# redirect the output to `build.sh`
./build-csky-gcc.py csky-gcc --src ./ --triple csky-unknown-elf --jobs=-1 --fake >> build.sh
```

and add the flags for each build step, just like my [build.sh](build.sh)

````
--with-gmp=/opt/homebrew/Cellar/gmp/6.2.1_1 \
--with-mpfr=/opt/homebrew/Cellar/mpfr/4.1.0-p13 \
--with-mpc=/opt/homebrew/Cellar/libmpc/1.3.1
```

Note the exact version and path should be found on your system with [`pkg-config`](https://en.wikipedia.org/wiki/Pkg-config) or `brew info`.

Shit might happens

# c-sky toolchains built

You could find x86_64-linux-gnu, mingw, even i686-linux-gnu toolchains in [upstream](https://occ.t-head.cn/community/download?id=3885366095506644992) (See also [this AUR](https://aur.archlinux.org/packages/csky-toolchain-bin))
However, I could not find any prebuilt for `aarch64-linux-gnu` (Arm64 Linux) and `apple-aarch64-darwin` (Arm64 macOS i.e. M1/M2 series) anywhere on the Internet. So I built them by myself.

You could find the prebuilt toolchains in [releases](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.1).

Architectures I have built for:

- [`x86_64-linux-gnu`](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.1.1)
- [`i686-w64-mingw32`](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.1.1)
- [`aarch64-linux-gnu`](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.1)
- [`apple-aarch64-darwin`](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.1)

You can find some useful info in [backup](https://github.com/crosstyan/Xuantie-800-gcc-elf-newlib-prebuilt/releases/tag/v0.00)

## How to build

If you want to build the toolchains by yourself, here are some tips.

### Linux

Pretty straightforward. Just follow [c-sky/toolchain-build](https://github.com/c-sky/toolchain-build) make sure you have enough disk space (at least 80GB) and RAM (at least 8GB).

#### Dependencies

```bash
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```

```bash
sudo pacman -S autoconf automake curl python3 libmpc mpfr gmp gawk base-devel bison flex texinfo gperf libtool patchutils bc zlib expat
```

```bash
git clone https://github.com/c-sky/toolchain-build --recursive --depth=1
# if you miss the --recursive flag
# git submodule update --init
# I would say you should use jobs=$(nproc)-2 if you don't want your computer to freeze
# especially when you are using a desktop environment
./build-csky-gcc.py csky-gcc --src ./ --triple csky-unknown-elf --jobs=-1
```

When the build is done, you could find the toolchains in
`build-gcc-csky-unknown-elf/Xuantie-800-gcc-elf-newlib-x86_64`. The build script
just assumed you are using `x86_64` and don't even bother to check your actual
architecture. <sup><sub>It's just a name anyway.</sub></sup>

### MinGW

I were using [ArchLinux](https://archlinux.org) to cross compile this. Follow the steps like [Windows: Cross-compilation using Linux](https://www.wireshark.org/docs/wsdg_html_chunked/ChSetupCross.html). For other architectures, refer to [MinGW-w64](https://www.mingw-w64.org/downloads/)

```bash
pacman -S mingw-w64
# since it's building from source, it might take a while
paru -S mingw-w64-libmpc mingw-w64-mpfr mingw-w64-gmp
```

You'll find `/usr/i686-w64-mingw32` and the cross compiler should be able pick up the dependencies from it.

> [!IMPORTANT]
> You should already install the `csky` toolchain in your host (in this case `x86_64`) before building the `mingw` toolchain and add the `bin` directory to your `PATH`. Somehow the building process requires the `csky` toolchain to be installed in the host.

```
cd toolchain-build
./build-csky-gcc.py csky-gcc --src ./ --triple csky-unknown-elf --jobs -1 --host mingw
```

I'm not quite sure why the result is 32bit, but it works. (try to figure out how to build a 64bit UCRT version)

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
# brew install autoconf automake curl libmpc mpfr gmp gawk bison flex texinfo gperf libtool patchutils bc zlib expat
```

You would have to do some modifications to the build script, luckily it has `--fake` flag.

```bash
# redirect the output to `build.sh`
./build-csky-gcc.py csky-gcc --src ./ --triple csky-unknown-elf --jobs=-1 --fake >> build.sh
```

and add the flags for each build step, just like my [build.sh](build.sh)

```bash
--with-gmp=/opt/homebrew/Cellar/gmp/6.2.1_1 \
--with-mpfr=/opt/homebrew/Cellar/mpfr/4.1.0-p13 \
--with-mpc=/opt/homebrew/Cellar/libmpc/1.3.1
```

> [!NOTE]
> the exact version and path should be found on your system with [`pkg-config`](https://en.wikipedia.org/wiki/Pkg-config) or `brew info`.

## TODO

- [ ] CI

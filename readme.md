# 2020-10-16


# 2020-10-17

Install Ubuntu Server 14.04.6 on VirtualBox as that is the recommended version.

Ubuntu 14.04 packages server was still at the `archive.ubuntu.com` server.

## install required packages by AOSP

```sh
sudo apt-get install git-core gnupg flex bison gperf build-essential \
zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev \
libgl1-mesa-dev libxml2-utils xsltproc unzip
```

## update `git`

the latest `git` version on Ubuntu 14.04 was `1.9.x` and it was too outdated.

### first try (failed)

```sh
sudo add-apt-repository ppa:git-core/ppa
```

## second try

compiling `git` myself

clone git on another linux machine

I am going to compile it and download the binaries from the server only. It is because these distros will have invalid SSL certificates in the future and they cannot get anything that requires SSL/TLS (https included).

```sh
git clone git://git.kernel.org/pub/scm/git/git.git
```

At the time of writing, the latest stable `git` version is `2.28.0`. I am thinking of archiving this checkpoint, pushing and compiling it on the server.

run at your workstation

```sh
git checkout tags/v2.28.0
git archive -o "git-v2.28.0.zip" HEAD

python3 -m http.server 8080
```

download the package from your server

```sh
wget http://<your-ip-address>:8080/git-v2.28.0.zip
```

While unzipping the archieve with `unzip` program, I accidentally inflate the current directory (`~/Downloads`). When I tried to execute `mv ./* ../git-v2.28.0/`, all the dotfiles (e.g. `.github`) are left behind. This is getting annoying.

I found a way to make `mv ./*` to include all the `dotfiles` next time.

```sh
shopt -s dotglob
```

It is recommended that we put that in the `.bashrc` file.

back to compiling `git`

instructions taken from the `git-scm` website

```sh
make configure
./configure --prefix=/usr
make all doc info
sudo make install install-doc install-html install-info
```

hopefully we have installed all the required packages

```sh
make configure
```

> `autoconf: not found`

```sh
sudo apt install autoconf
```

```sh
./configure --prefix=/usr
make all doc info
```

I forgot to add the `-j` option to speed up compilation.

There is some errors.

```
GITGUI_VERSION = 0.21.GITGUI
    * new locations or Tcl/Tk interpreter
    GEN git-gui
    INDEX lib/
    * tclsh failed; using unoptimized loading
    MSGFMT    po/bg.msg make[1]: *** [po/bg.msg] Error 127
make: *** [all] Error 2
```

I will try to compile with `make git-core` only as `git-core` is the package name so I hope that there is a task for it.

```sh
make clean
make configure
./configure --prefix=/usr
make -j$(nproc) git-core
```

> `make: *** No rule to make target `git-core'.  Stop.`

```sh
make -j$(nproc) core
```

> `make: *** No rule to make target `core'.  Stop.`

```sh
make -j$(nproc) all
```

```
    MSGFMT po/build/locale/bg/LC_MESSAGES/git.mo
/bin/sh: 1: msgfmt: not found
make: *** [po/build/locale/bg/LC_MESSAGES/git.mo] Error 127
```

Someone suggested that `msgfmt` is for documentation and localization so it is not essential for `git` and we can skip them with the `-i` option.

```sh
make -j$(nproc) -i all
```

Nope, I will compile it on my main machine and transfer the compiled binaries to the server.

run on my main machine

```sh
make configure
./configure
make -j$(nproc) all
```

After done, there will be a `git` executable at the build directory. Upload it to somewhere on the server.

__replace old `git`__

```sh
sudo apt purge git git-core
```

I will just add the uploaded `git` to the `PATH` variable.

## installing AOSP's `repo` tool

```sh
mkdir ~/bin
```

add that directory to the `PATH` variable in `.bashrc`

download the `repo` script on my main machine and pull it from the server later as Google use `https` to host the `repo` script.

(at the time of writing)

```sh
wget https://storage.googleapis.com/git-repo-downloads/repo
```

upload the `repo` script...

make the script executable

```sh
chmod a+x ./repo
```

It seems that the `repo` script I download required Python 3.6+ and Ubuntu 14.04 was still using Python 3.4.x.

The Python 2.7 supported `repo` script is at this URL at the time of writing.

```sh
wget https://storage.googleapis.com/git-repo-downloads/repo-1
```

## download the AOSP source

```
mkdir android_2.3_sources
cd android_2.3_sources
```

pull the source code

```
repo init -u https://android.googlesource.com/platform/manifest -b android-2.3.7_r1 --partial-clone --clone-filter=blob:limit=10M
```

```
warning: templates not found in /usr/local/share/git-core/templates
Get https://gerrit.googlesource.com/git-repo/clone.bundle
Get https://gerrit.googlesource.com/git-repo
fatal: unable to find remote helper for 'https'
fatal: cloning the git-repo repository failed, will remove '.repo/repo'
```

I need to compile and install the full `git` on the server.

```sh
make clean
make configure
./configure --prefix=/usr
make -j$(nproc) all doc
```

failed

```
GITGUI_VERSION = 0.21.GITGUI
    * new locations or Tcl/Tk interpreter
    GEN git-gui
    INDEX lib/
    * tclsh failed; using unoptimized loading
    MSGFMT    po/bg.msg make[1]: *** [po/bg.msg] Error 127
make: *** [all] Error 2
```

```sh
sudo apt install gettext
```

```sh
make clean
make configure
./configure --prefix=/usr
make -j$(nproc) all doc
```

failed

```
    * new asciidoc flags
    GEN howto-index.txt
    LINK git-remote-testsvn
    ASCIIDOC git-tools.html
/bin/sh: 1: asciidoc: not found
make[1]: *** [git-tools.html] Error 127
```

```sh
sudo apt-get install dh-autoreconf libcurl4-gnutls-dev libexpat1-dev \
  gettext libz-dev libssl-dev install-info
```

I will skip the `doc` task this time.

```sh
make clean
make configure
./configure --prefix=/usr
make -j$(nproc) all
```

```sh
sudo make install
```

```
repo init -u https://android.googlesource.com/platform/manifest -b android-2.3.7_r1 --partial-clone --clone-filter=blob:limit=10M
```

```sh
repo sync
```

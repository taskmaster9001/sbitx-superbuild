```
  ______   _______   ______  ________          __    __
 /      \ |       \ |      \|        \        |  \  |  \
|  $$$$$$\| $$$$$$$\ \$$$$$$ \$$$$$$$$        | $$  | $$
| $$___\$$| $$__/ $$  | $$     | $$    ______  \$$\/  $$
 \$$    \ | $$    $$  | $$     | $$   |      \  >$$  $$
 _\$$$$$$\| $$$$$$$\  | $$     | $$    \$$$$$$ /  $$$$\
|  \__| $$| $$__/ $$ _| $$_    | $$           |  $$ \$$\
 \$$    $$| $$    $$|   $$ \   | $$           | $$  | $$
  \$$$$$$  \$$$$$$$  \$$$$$$    \$$            \$$   \$$

Copyright 2020 - by Piyush Aggarwal, HF-Signals.
```

# Pre-requisite

To build SBIT-X from sources you need some prerequisite tools and libraries.

- build-essentials
- libgtk-3-dev
- gcc
- CMake-3.1.0 or newer
- git

> Note that these are Debian style package names, other distributions will have different package  names and package contents.

**Out of source builds are strongly recommended.**

# Installation

1. Open a shell and cd to inside the directory where you would like to set up the build environment **(preferably an empty direcrory)**.
2. Simply run the following: -

    ```
    git clone https://github.com/taskmaster9001/sbitx-superbuild
    mkdir build
    cd build
    cmake ../sbitx-superbuild .     # mind the dot at the end of the command

    make -j$(($(nproc)-1)) fftw3-build
    sudo make -j4 fftw3-install

    make -j$(($(nproc)-1)) fftw3f-build
    sudo make -j4 fftw3f-install

    make -j$(($(nproc)-1))
    sudo make -j4 install
    ```

# Testing changes

1. Apply your changes to the component sources within the build folder.
2. Re-run `sudo make install`

# (Some) Available Targets

- make -j$(($(nproc)-1)) [sbitx-build]
- make -j$(($(nproc)-1)) [sbitx-]install
- make -j$(($(nproc)-1)) [sbitx-]uninstall

- make -j$(($(nproc)-1)) fftw3-build
- make -j$(($(nproc)-1)) fftw3-install
- make -j$(($(nproc)-1)) fftw3-uninstall

- make -j$(($(nproc)-1)) fftw3f-build
- make -j$(($(nproc)-1)) fftw3f-install
- make -j$(($(nproc)-1)) fftw3f-uninstall

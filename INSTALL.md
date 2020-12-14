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

# Installation

To build SBIT-X from sources you need some prerequisite tools and libraries.

- build-essentials
- libgtk-3-dev
- gcc
- CMake-3.1.0 or newer
- git

> Note that these are Debian style package names, other distributions will have different package  names and package contents.

**Out of source builds are strongly recommended.**

# Basic Package Building

1. Open a shell and cd to inside the directory where you would like to set up the build environment **(preferably an empty direcrory)**.
2. Simply run the following: -

```
git clone https://github.com/taskmaster9001/sbitx-superbuild
mkdir build
cd build
cmake ../sbitx-superbuild .     # mind the dot at the end of the command
make -j$(($(nproc)-1))          # leaving a thread free for other stuff
```
# Testing changes

1. Apply your changes to the component sources within the build folder.
2. Re-run make

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

# SBIT-X Superbuild

This package builds the SBIT-X source.
It does not directly contain the source files, instead it consists of a CMake script that builds the application by fetching the sources from their respective source control repositories and then building the application.

SBIT-X also has a CMake build system but currently  it also  has a dependency on a library called FFTW3 which itself has to built from source. The FFTW3 source is used because it  contains  the  latest  patches.

This project combines the building of FFTW3 and SBIT-X.

## The SBIT-X project web site is: **TO BE UPDATED**

# For component-specific documentation

Please refer to the component project README files as well:

- SBIT-X : https://github.com/afarhan/sbitx
- FFTW3 : https://github.com/FFTW/fftw3

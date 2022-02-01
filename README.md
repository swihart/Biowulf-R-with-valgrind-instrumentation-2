# Biowulf-R-with-valgrind-instrumentation-2
Notes on how to get a personal R build  `-with-valgrind-instrumentation=2` for that CRAN-esque valgrind checks

---

You've never been an admin of R.  It's really hard work.  There are entire manuals for it.  Here are two.  Colin's is a "bookdown" version of it for readability.

  * [Colin Fay's R Installation Administration](https://colinfay.me/r-installation-administration/index.html)
  * [R Admin Man](https://cran.r-project.org/doc/manuals/r-release/R-admin.html#Obtaining-R)

And an old note about valgrind at the time of R configuration:

  * [Lumley's 2005 note](https://developer.r-project.org/valgrind-internal.html)


These notes below are minimal, stream-of-consciousness, and semi-personal.  They may not generalize to you.  I've benefited greatly from others posting online how they tried to tackle things, without trying to make it a catch-all tutorial.  This is my contribution in this vein, with no guarantees!

---

## 1. Obtain R
  
  * Go on biowulf
  * Get in data folder (you need some GBs and permission to pull this off)
  * make /data/swihartbj/R_valgrind_level_2
  * In a browser go to https://cran.r-project.org/ and download the top link (R-4.1.2.tar.gz as of this writing)
  * Put that R-x.y.z.tar.gz in your `R_valgrind_level_2` folder
  * At prompt, do `tar -xf R-x.y.z.tar.gz`.  
  * At prompt, do ls and see the tar.gz and the folder

## 2. Configure and Make and Check

  * `cd` into the folder
  * `unix>` `./configure LIBnn=lib -with-pcre1 -with-valgrind-instrumentation=2 -with-system-valgrind-headers`
  * lots of `checking for...` etc will whiz by. Just wait. Should end with something like

````
R is now configured for x86_64-pc-linux-gnu

  Source directory:            .
  Installation directory:      /usr/local

  C compiler:                  gcc -std=gnu11  -g -O2
  Fortran fixed-form compiler: gfortran  -g -O2

  Default C++ compiler:        g++ -std=gnu++11  -g -O2
  C++11 compiler:              g++ -std=gnu++11  -g -O2
  C++14 compiler:
  C++17 compiler:
  C++20 compiler:
  Fortran free-form compiler:  gfortran  -g -O2
  Obj-C compiler:

  Interfaces supported:        X11
  External libraries:          pcre1, readline, curl
  Additional capabilities:     PNG, JPEG, TIFF, NLS, cairo, ICU
  Options enabled:             shared BLAS, R profiling

  Capabilities skipped:
  Options not enabled:         memory profiling

  Recommended packages:        yes
````
  * `unix>` `make`
  * lots of stuff flies by.  sort of looks like when packages are compiling when you cmd+shft+B in Rstudio on your local macbook
  * 

---

## Stray Observations
  * seems I needed `LIBnn=lib` for the configure step
  * seems like I need only one dash for the configure options involving valgrind
  * `./configure` would run but error "configure: error: PCRE2 library and headers are required, or use --with-pcre1 and PCRE >= 8.32 with UTF-8 support" which google led me to [SO](https://unix.stackexchange.com/a/653785/117378). The following *did* *not* work.  However, adding option `-with-pcre1` did work!

```
 wget https://ftp.pcre.org/pub/pcre/pcre2-10.37.tar.gz
tar - zxvf pcre2-10.37.tar.gz
cd pcre2-10.37
./configure
make -j 24
make install
I am using Ubuntu 20.04 and R version 4.1.0
```

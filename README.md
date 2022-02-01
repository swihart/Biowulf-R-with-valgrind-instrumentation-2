# Biowulf-R-with-valgrind-instrumentation-2
Notes on how to get a personal R build  `-with-valgrind-instrumentation=2` for that CRAN-esque valgrind checks

---

You've never been an admin of R.  It's really hard work.  There are entire manuals for it.  Here are two.  Colin's is a "bookdown" version of it for readability.

  * [Colin Fay's R Installation Administration](https://colinfay.me/r-installation-administration/index.html)
  * [R Admin Man](https://cran.r-project.org/doc/manuals/r-release/R-admin.html#Obtaining-R)

And an old note about valgrind at the time of R configuration:

  * [Lumley's 2005 note](https://developer.r-project.org/valgrind-internal.html)


These notes are personal.  They may not generalize to you.  I've benefited greatly from others posting online how they tried to tackle things.  This is my contribution, with no guarantees!

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

  * cd into the folder
  * `./configure LIBnn=lib -with-valgrind-instrumentation=2 -with-system-valgrind-headers`
  * ./configure LIBnn=lib -with-pcre1 -with-valgrind-instrumentation=2 -with-system-valgrind-headers
  * lots of `checking for...` etc will whiz by. Just wait. 
  * `make check`

---

## Stray Observations
  * seems I needed LIBnn=lib for the configure step
  * seems like I need only one dash for the configure options involving valgrind
  * ./configure would run but error "configure: error: PCRE2 library and headers are required, or use --with-pcre1 and PCRE >= 8.32 with UTF-8 support" which google led me to [SO](https://unix.stackexchange.com/a/653785/117378). The following *did* *not* work.  However, adding option `-with-pcre1` did!

```
 wget https://ftp.pcre.org/pub/pcre/pcre2-10.37.tar.gz
tar - zxvf pcre2-10.37.tar.gz
cd pcre2-10.37
./configure
make -j 24
make install
I am using Ubuntu 20.04 and R version 4.1.0
```

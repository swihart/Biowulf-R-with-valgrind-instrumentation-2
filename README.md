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

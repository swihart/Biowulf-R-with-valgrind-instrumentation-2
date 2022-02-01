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
  * Now I see this:

````
JAVA_HOME        : /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64/jre
Java library path: $(JAVA_HOME)/lib/amd64/server
JNI cpp flags    : -I$(JAVA_HOME)/../include -I$(JAVA_HOME)/../include/linux
JNI linker flags : -L$(JAVA_HOME)/lib/amd64/server -ljvm
Updating Java configuration in /data/swihartbj/R_valgrind_level_2/R-4.1.2
Done.

make[1]: Leaving directory `/gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2'
````
 * `unix>` `make check`
 * you'll see `Testing examples for package 'base'` and this takes a while.
 * `unix>` `cd bin` cd into bin and see if you have R.  You need an interactive node to launch it, so don't do anything here/now

## 3. Use your creation, CreatoR!
 * `unix >` `sinteractive`
 * cd into `bin` folder
 * `./R -d "valgrind --tool=memcheck --leak-check=full --track-origins=yes" --vanilla`
 * should see this header (valgrind R session):

````

````

 * run your code interactively


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
---

## First go-around with repeated v1.1.4

````
[swihartbj@biowulf /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin 83]sinteractive --mem=22g
salloc: Pending job allocation 31471132
salloc: job 31471132 queued and waiting for resources
salloc: job 31471132 has been allocated resources
salloc: Granted job allocation 31471132
salloc: Waiting for resource configuration
salloc: Nodes cn0876 are ready for job
[swihartbj@cn0876 /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin 1]R -d "valgrind --tool=memcheck --leak-check=full --track-origins=yes" --vanilla
bash: R: command not found
[swihartbj@cn0876 /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin 2]ls
BATCH  check	config	INSTALL     libtool  mkinstalldirs  R	  Rd2pdf  Rdiff   Rprof    rtags  Stangle
build  COMPILE	exec	javareconf  LINK     pager	    Rcmd  Rdconv  REMOVE  Rscript  SHLIB  Sweave
[swihartbj@cn0876 /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin 3]./R -d "valgrind --tool=memcheck --leak-check=full --track-origins=yes" --vanilla
==45366== Memcheck, a memory error detector
==45366== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==45366== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==45366== Command: /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin/exec/R --vanilla
==45366==

R version 4.1.2 (2021-11-01) -- "Bird Hippie"
Copyright (C) 2021 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> install.packages("repeated")
--- Please select a CRAN mirror for use in this session ---
Secure CRAN mirrors

 1: 0-Cloud [https]
 2: Australia (Canberra) [https]
 3: Australia (Melbourne 1) [https]
 4: Australia (Melbourne 2) [https]
 5: Australia (Perth) [https]
 6: Austria [https]
 7: Belgium (Brussels) [https]
 8: Brazil (PR) [https]
 9: Brazil (RJ) [https]
10: Brazil (SP 1) [https]
11: Brazil (SP 2) [https]
12: Bulgaria [https]
13: Canada (MB) [https]
14: Canada (ON 2) [https]
15: Canada (ON 3) [https]
16: Chile (Santiago) [https]
17: China (Beijing 2) [https]
18: China (Hefei) [https]
19: China (Hong Kong) [https]
20: China (Guangzhou) [https]
21: China (Lanzhou) [https]
22: China (Nanjing) [https]
23: China (Shanghai 1) [https]
24: China (Shanghai 2) [https]
25: China (Shenzhen) [https]
26: Colombia (Cali) [https]
27: Costa Rica [https]
28: Cyprus [https]
29: Czech Republic [https]
30: Denmark [https]
31: East Asia [https]
32: Ecuador (Cuenca) [https]
33: Ecuador (Quito) [https]
34: Estonia [https]
35: France (Lyon 1) [https]
36: France (Lyon 2) [https]
37: France (Marseille) [https]
38: France (Montpellier) [https]
39: France (Paris 1) [https]
40: Germany (Erlangen) [https]
41: Germany (Leipzig) [https]
42: Germany (Göttingen) [https]
43: Germany (Münster) [https]
44: Germany (Regensburg) [https]
45: Greece [https]
46: Hungary [https]
47: Iceland [https]
48: India [https]
49: Indonesia (Jakarta) [https]
50: Iran [https]
51: Italy (Milano) [https]
52: Italy (Padua) [https]
53: Japan (Tokyo) [https]
54: Korea (Gyeongsan-si) [https]
55: Korea (Seoul 1) [https]
56: Korea (Ulsan) [https]
57: Malaysia [https]
58: Mexico (Mexico City) [https]
59: Mexico (Texcoco) [https]
60: Morocco [https]
61: Netherlands [https]
62: New Zealand [https]
63: Norway [https]
64: Russia (Moscow) [https]
65: South Africa (Johannesburg) [https]
66: Spain (A Coruña) [https]
67: Spain (Madrid) [https]
68: Sweden (Borås) [https]
69: Sweden (Umeå) [https]
70: Switzerland [https]
71: Taiwan (Taipei) [https]
72: Turkey (Denizli) [https]
73: Turkey (Istanbul) [https]
74: Turkey (Mersin) [https]
75: UK (Bristol) [https]
76: UK (London 1) [https]
77: USA (IA) [https]
78: USA (KS) [https]
79: USA (MI) [https]
80: USA (OH) [https]
81: USA (OR) [https]
82: USA (TN) [https]
83: USA (TX 1) [https]
84: Uruguay [https]
85: (other mirrors)

Selection: 1
also installing the dependency ‘rmutil’

trying URL 'https://cloud.r-project.org/src/contrib/rmutil_1.1.5.tar.gz'
Content type 'application/x-gzip' length 113667 bytes (111 KB)
==================================================
downloaded 111 KB

trying URL 'https://cloud.r-project.org/src/contrib/repeated_1.1.4.tar.gz'
Content type 'application/x-gzip' length 257650 bytes (251 KB)
==================================================
downloaded 251 KB

* installing *source* package ‘rmutil’ ...
** package ‘rmutil’ successfully unpacked and MD5 sums checked
** using staged installation
** libs
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c cutil.c -o cutil.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c dist.c -o dist.o
gfortran  -fpic  -g -O2  -c gettvc.f -o gettvc.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c rmutil_init.c -o rmutil_init.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c romberg.c -o romberg.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c toms614.c -o toms614.o
gcc -std=gnu11 -shared -L/usr/local/lib -o rmutil.so cutil.o dist.o gettvc.o rmutil_init.o romberg.o toms614.o -lgfortran -lm -lquadmath
installing to /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/00LOCK-rmutil/00new/rmutil/libs
** R
** byte-compile and prepare package for lazy loading
** help
*** installing help indices
** building package indices
** testing if installed package can be loaded from temporary location
** checking absolute paths in shared objects and dynamic libraries
** testing if installed package can be loaded from final location
** testing if installed package keeps a record of temporary installation path
* DONE (rmutil)
* installing *source* package ‘repeated’ ...
** package ‘repeated’ successfully unpacked and MD5 sums checked
** using staged installation
** libs
gfortran  -fpic  -g -O2  -c binnest.f -o binnest.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c calcs.c -o calcs.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c calcs1.c -o calcs1.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c calcs2.c -o calcs2.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c calcs3.c -o calcs3.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c calcs4.c -o calcs4.o
gfortran  -fpic  -g -O2  -c chidden.f -o chidden.o
gfortran  -fpic  -g -O2  -c cphidden.f -o cphidden.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c cutil.c -o cutil.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c dist.c -o dist.o
gfortran  -fpic  -g -O2  -c eigen.f -o eigen.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c gaps.c -o gaps.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c gar.c -o gar.o
gfortran  -fpic  -g -O2  -c gausscop.f -o gausscop.o
gfortran  -fpic  -g -O2  -c hidden.f -o hidden.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c kcountb.c -o kcountb.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c kserieb.c -o kserieb.o
gfortran  -fpic  -g -O2  -c logitord.f -o logitord.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c repeated_init.c -o repeated_init.o
gcc -std=gnu11 -I"/data/swihartbj/R_valgrind_level_2/R-4.1.2/include" -DNDEBUG   -I/usr/local/include   -fpic  -g -O2  -c romberg_sexp.c -o romberg_sexp.o
gcc -std=gnu11 -shared -L/usr/local/lib -o repeated.so binnest.o calcs.o calcs1.o calcs2.o calcs3.o calcs4.o chidden.o cphidden.o cutil.o dist.o eigen.o gaps.o gar.o gausscop.o hidden.o kcountb.o kserieb.o logitord.o repeated_init.o romberg_sexp.o -lgfortran -lm -lquadmath
installing to /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/00LOCK-repeated/00new/repeated/libs
** R
** byte-compile and prepare package for lazy loading
** help
*** installing help indices
** building package indices
** testing if installed package can be loaded from temporary location
** checking absolute paths in shared objects and dynamic libraries
** testing if installed package can be loaded from final location
** testing if installed package keeps a record of temporary installation path
* DONE (repeated)

The downloaded source packages are in
	‘/tmp/RtmpYHiv1F/downloaded_packages’
> library(repeated)
Loading required package: rmutil

Attaching package: ‘rmutil’

The following object is masked from ‘package:stats’:

    nobs

The following objects are masked from ‘package:base’:

    as.data.frame, units

> dose <- c(9,12,4,9,11,10,2,11,12,9,9,9,4,9,11,9,14,7,9,8)
> y <- c(8.674419, 11.506066, 11.386742, 27.414532, 12.135699,  4.359469,
+        1.900681, 17.425948,  4.503345,  2.691792,  5.731100, 10.534971,
+        11.220260,  6.968932,  4.094357, 16.393806, 14.656584,  8.786133,
+        20.972267, 17.178012)
> id <- rep(1:4, each=5)
> beg.jim <- Sys.time()
> jim <- gnlmix(y, mu=~a+b*dose+rand, random="rand", nest=id, pmu=c(a=8.7,b=0.25),
+               pshape=3.44, pmix=2.3)

````

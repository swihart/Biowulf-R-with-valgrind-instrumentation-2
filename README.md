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
 * should see something like this header (valgrind R session):

````
[swihartbj@cn0876 /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin 3]./R -d "valgrind --tool=memcheck --leak-check=full --track-origins=yes" --vanilla
==45366== Memcheck, a memory error detector
==45366== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==45366== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==45366== Command: /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin/exec/R --vanilla
==45366==

R version 4.1.2 (2021-11-01) -- "Bird Hippie"
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

gnlmix successfully ran, so I typed q("no") and a bunch of messages from valgrind flooded the screen and made me worried.  so I reran below -- but take heart -- if the errors were within gnlmix they would have appeared during its execution.  I think we have the valgrind issue solved for repeated!

---

## 2nd go around 

````
[swihartbj@cn0876 /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin 4]./R -d "valgrind --tool=memcheck --leak-check=full --track-origins=yes --show-leak-kinds=all -v" --vanilla
==57107== Memcheck, a memory error detector
==57107== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==57107== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==57107== Command: /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin/exec/R --vanilla
==57107==
--57107-- Valgrind options:
--57107--    --tool=memcheck
--57107--    --leak-check=full
--57107--    --track-origins=yes
--57107--    --show-leak-kinds=all
--57107--    -v
--57107-- Contents of /proc/version:
--57107--   Linux version 3.10.0-862.14.4.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) ) #1 SMP Wed Sep 26 15:12:11 UTC 2018
--57107--
--57107-- Arch and hwcaps: AMD64, LittleEndian, amd64-cx16-lzcnt-rdtscp-sse3-avx-avx2-bmi
--57107-- Page sizes: currently 4096, max supported 4096
--57107-- Valgrind library directory: /usr/lib64/valgrind
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/bin/exec/R
--57107-- Reading syms from /usr/lib64/ld-2.17.so
--57107-- Reading syms from /usr/lib64/valgrind/memcheck-amd64-linux
--57107--    object doesn't have a symbol table
--57107--    object doesn't have a dynamic symbol table
--57107-- Scheduler: using generic scheduler lock implementation.
--57107-- Reading suppressions file: /usr/lib64/valgrind/default.supp
==57107== embedded gdbserver: reading from /tmp/vgdb-pipe-from-vgdb-to-57107-by-swihartbj-on-biowulf.nih.gov
==57107== embedded gdbserver: writing to   /tmp/vgdb-pipe-to-vgdb-from-57107-by-swihartbj-on-biowulf.nih.gov
==57107== embedded gdbserver: shared mem   /tmp/vgdb-pipe-shared-mem-vgdb-57107-by-swihartbj-on-biowulf.nih.gov
==57107==
==57107== TO CONTROL THIS PROCESS USING vgdb (which you probably
==57107== don't want to do, unless you know exactly what you're doing,
==57107== or are doing some strange experiment):
==57107==   /usr/lib64/valgrind/../../bin/vgdb --pid=57107 ...command...
==57107==
==57107== TO DEBUG THIS PROCESS USING GDB: start GDB like this
==57107==   /path/to/gdb /data/swihartbj/R_valgrind_level_2/R-4.1.2/bin/exec/R
==57107== and then give GDB the following command
==57107==   target remote | /usr/lib64/valgrind/../../bin/vgdb --pid=57107
==57107== --pid is optional if only one valgrind process is running
==57107==
--57107-- REDIR: 0x4019f70 (ld-linux-x86-64.so.2:strlen) redirected to 0x58059d91 (???)
--57107-- REDIR: 0x4019d40 (ld-linux-x86-64.so.2:index) redirected to 0x58059dab (???)
--57107-- Reading syms from /usr/lib64/valgrind/vgpreload_core-amd64-linux.so
--57107-- Reading syms from /usr/lib64/valgrind/vgpreload_memcheck-amd64-linux.so
==57107== WARNING: new redirection conflicts with existing -- ignoring it
--57107--     old: 0x04019f70 (strlen              ) R-> (0000.0) 0x58059d91 ???
--57107--     new: 0x04019f70 (strlen              ) R-> (2007.0) 0x04c2cad0 strlen
--57107-- REDIR: 0x4019ef0 (ld-linux-x86-64.so.2:strcmp) redirected to 0x4c2dc20 (strcmp)
--57107-- REDIR: 0x401aae0 (ld-linux-x86-64.so.2:mempcpy) redirected to 0x4c30ca0 (mempcpy)
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/lib/libRblas.so
--57107-- Reading syms from /usr/lib64/libgfortran.so.3.0.0
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libm-2.17.so
--57107-- Reading syms from /usr/lib64/libquadmath.so.0.0.0
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libreadline.so.6.2
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libpcre.so.1.2.0
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/liblzma.so.5.2.2
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libbz2.so.1.0.6
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libz.so.1.2.7
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/librt-2.17.so
--57107-- Reading syms from /usr/lib64/libdl-2.17.so
--57107-- Reading syms from /usr/lib64/libicuuc.so.50.1.2
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libicui18n.so.50.1.2
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libgomp.so.1.0.0
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libpthread-2.17.so
--57107-- Reading syms from /usr/lib64/libc-2.17.so
--57107-- Reading syms from /usr/lib64/libgcc_s-4.8.5-20150702.so.1
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libtinfo.so.5.9
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libicudata.so.50.1.2
--57107--    object doesn't have a symbol table
--57107-- Reading syms from /usr/lib64/libstdc++.so.6.0.19
--57107--    object doesn't have a symbol table
--57107-- REDIR: 0x740ce40 (libc.so.6:strcasecmp) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x7409bc0 (libc.so.6:strnlen) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x740f110 (libc.so.6:strncasecmp) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x740c620 (libc.so.6:memset) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x740c5d0 (libc.so.6:memcpy@GLIBC_2.2.5) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x7409ae0 (libc.so.6:__GI_strlen) redirected to 0x4c2ca30 (__GI_strlen)
--57107-- REDIR: 0x7409ce0 (libc.so.6:__GI_strncmp) redirected to 0x4c2d260 (__GI_strncmp)
--57107-- REDIR: 0x740b5b0 (libc.so.6:__GI_strrchr) redirected to 0x4c2c490 (__GI_strrchr)
--57107-- REDIR: 0x7402800 (libc.so.6:malloc) redirected to 0x4c29b9c (malloc)
--57107-- REDIR: 0x740c440 (libc.so.6:__GI_memmove) redirected to 0x4c30210 (__GI_memmove)
--57107-- REDIR: 0x7402c20 (libc.so.6:free) redirected to 0x4c2ac96 (free)
--57107-- REDIR: 0x740c680 (libc.so.6:__GI_memset) redirected to 0x4c2fec0 (memset)
--57107-- REDIR: 0x7403210 (libc.so.6:calloc) redirected to 0x4c2b91f (calloc)
--57107-- REDIR: 0x7411850 (libc.so.6:__GI_memcpy) redirected to 0x4c2e5c0 (__GI_memcpy)
--57107-- REDIR: 0x7409a90 (libc.so.6:strlen) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74c3670 (libc.so.6:__strlen_sse42) redirected to 0x4c2ca90 (__strlen_sse42)
--57107-- REDIR: 0x74117e0 (libc.so.6:memcpy@@GLIBC_2.14) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74c9480 (libc.so.6:__memcpy_ssse3) redirected to 0x4c2dfe0 (memcpy@@GLIBC_2.14)
--57107-- REDIR: 0x7422630 (libc.so.6:__GI_strstr) redirected to 0x4c30f30 (__strstr_sse2)
--57107-- REDIR: 0x7408080 (libc.so.6:__GI_strcmp) redirected to 0x4c2db30 (__GI_strcmp)
--57107-- REDIR: 0x740bcb0 (libc.so.6:memchr) redirected to 0x4c2dcc0 (memchr)
--57107-- REDIR: 0x7407fc0 (libc.so.6:__GI_strchr) redirected to 0x4c2c5c0 (__GI_strchr)
--57107-- REDIR: 0x74130d0 (libc.so.6:strchrnul) redirected to 0x4c307c0 (strchrnul)
--57107-- REDIR: 0x740c7f0 (libc.so.6:__GI_mempcpy) redirected to 0x4c309d0 (__GI_mempcpy)
--57107-- REDIR: 0x740cce0 (libc.so.6:__GI_stpcpy) redirected to 0x4c2f910 (__GI_stpcpy)
--57107-- REDIR: 0x7407f80 (libc.so.6:index) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74bb640 (libc.so.6:__strchr_sse42) redirected to 0x4c2c680 (index)
--57107-- REDIR: 0x7409c60 (libc.so.6:strncat) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x741d6a0 (libc.so.6:__strncat_ssse3) redirected to 0x4c2c8a0 (strncat)
--57107-- REDIR: 0x740b530 (libc.so.6:strncpy) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74df2b0 (libc.so.6:__strncpy_ssse3) redirected to 0x4c2ccb0 (strncpy)
--57107-- REDIR: 0x73b67e0 (libc.so.6:setenv) redirected to 0x4c315d0 (setenv)
--57107-- REDIR: 0x7402d00 (libc.so.6:realloc) redirected to 0x4c2baee (realloc)
--57107-- REDIR: 0x74094d0 (libc.so.6:strcpy) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74ddb00 (libc.so.6:__strcpy_ssse3) redirected to 0x4c2caf0 (strcpy)
--57107-- REDIR: 0x740c040 (libc.so.6:__GI_memcmp) redirected to 0x4c2f510 (__GI_memcmp)
--57107-- REDIR: 0xffffffffff600000 (???:???) redirected to 0x58059d73 (???)
--57107-- REDIR: 0x7408040 (libc.so.6:strcmp) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74bb6f0 (libc.so.6:__strcmp_sse42) redirected to 0x4c2dbd0 (__strcmp_sse42)
--57107-- REDIR: 0x740c000 (libc.so.6:bcmp) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74e6910 (libc.so.6:__memcmp_sse4_1) redirected to 0x4c2f650 (__memcmp_sse4_1)
--57107-- REDIR: 0x74c5100 (libc.so.6:__strncasecmp_avx) redirected to 0x4c2d490 (strncasecmp)
--57107-- REDIR: 0x7409ca0 (libc.so.6:strncmp) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74bc4a0 (libc.so.6:__strncmp_sse42) redirected to 0x4c2d340 (__strncmp_sse42)
--57107-- REDIR: 0x7422bf0 (libc.so.6:strstr) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74bd620 (libc.so.6:__strstr_sse42) redirected to 0x4c30fc0 (__strstr_sse42)
--57107-- REDIR: 0x740b570 (libc.so.6:rindex) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74bd480 (libc.so.6:__strrchr_sse42) redirected to 0x4c2c520 (__strrchr_sse42)
--57107-- REDIR: 0x7407d80 (libc.so.6:strcat) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x741ba90 (libc.so.6:__strcat_ssse3) redirected to 0x4c2c6c0 (strcat)
--57107-- REDIR: 0x74095f0 (libc.so.6:strcspn) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74c3710 (libc.so.6:__strcspn_sse42) redirected to 0x4c310b0 (strcspn)
--57107-- REDIR: 0x74ce820 (libc.so.6:__memmove_ssse3) redirected to 0x4c2dd80 (memcpy@GLIBC_2.2.5)
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/methods/libs/methods.so
--57107-- REDIR: 0x73b6840 (libc.so.6:unsetenv) redirected to 0x4c31530 (unsetenv)

R version 4.1.2 (2021-11-01) -- "Bird Hippie"
Copyright (C) 2021 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

--57107-- REDIR: 0x7409bf0 (libc.so.6:__GI_strnlen) redirected to 0x4c2c9e0 (__GI_strnlen)
--57107-- REDIR: 0x7409510 (libc.so.6:__GI_strcpy) redirected to 0x4c2cbd0 (__GI_strcpy)
--57107-- REDIR: 0x740cea0 (libc.so.6:__GI___strcasecmp_l) redirected to 0x4c2d750 (__GI___strcasecmp_l)
--57107-- REDIR: 0x7412ec0 (libc.so.6:__GI___rawmemchr) redirected to 0x4c30820 (__GI___rawmemchr)
R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

--57107-- Reading syms from /usr/lib64/gconv/ISO8859-1.so
--57107-- REDIR: 0x401ac30 (ld-linux-x86-64.so.2:stpcpy) redirected to 0x4c2fc90 (stpcpy)
  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/utils/libs/utils.so
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/grDevices/libs/grDevices.so
--57107-- REDIR: 0x740cca0 (libc.so.6:stpcpy) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74e1e30 (libc.so.6:__stpcpy_ssse3) redirected to 0x4c2f830 (stpcpy)
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/graphics/libs/graphics.so
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/stats/libs/stats.so
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/lib/libRlapack.so
--57107-- REDIR: 0xffffffffff600400 (???:???) redirected to 0x58059d7d (???)
--57107-- REDIR: 0x74c3a90 (libc.so.6:__strcasecmp_avx) redirected to 0x4c2d3b0 (strcasecmp)
> --57107-- REDIR: 0x7492360 (libc.so.6:__memcpy_chk) redirected to 0x4a247b0 (_vgnU_ifunc_wrapper)
--57107-- REDIR: 0x74c9470 (libc.so.6:__memcpy_chk_ssse3) redirected to 0x4c30d90 (__memcpy_chk)
library(repeated)
Loading required package: rmutil
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/rmutil/libs/rmutil.so

Attaching package: ‘rmutil’

The following object is masked from ‘package:stats’:

    nobs

The following objects are masked from ‘package:base’:

    as.data.frame, units

--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/library/repeated/libs/repeated.so
> library(repeated)
>
> dose <- c(9,12,4,9,11,10,2,11,12,9,9,9,4,9,11,9,14,7,9,8)
> y <- c(8.674419, 11.506066, 11.386742, 27.414532, 12.135699,  4.359469,
+        1.900681, 17.425948,  4.503345,  2.691792,  5.731100, 10.534971,
+        11.220260,  6.968932,  4.094357, 16.393806, 14.656584,  8.786133,
+        20.972267, 17.178012)
> id <- rep(1:4, each=5)
>
>
> beg.jim <- Sys.time()
> jim <- gnlmix(y, mu=~a+b*dose+rand, random="rand", nest=id, pmu=c(a=8.7,b=0.25),
+               pshape=3.44, pmix=2.3)
--57107-- Reading syms from /gpfs/gsfs8/users/swihartbj/R_valgrind_level_2/R-4.1.2/modules/lapack.so
> end.jim <- Sys.time()
> time.jim <- end.jim - beg.jim
> time.jim
Time difference of 1.167166 mins
> resp <- restovec(matrix(y, nrow=4, byrow=TRUE), name="y")
> reps <- rmna(resp, tvcov=tvctomat(matrix(dose, nrow=4, byrow=TRUE), name="dose"))
> glmm(y~dose, nest=individuals, data=reps)

Call:  glmm(y ~ dose, nest = individuals, data = reps)

Coefficients:
(Intercept)         dose           sd
     8.7102       0.2491       3.0862

Degrees of Freedom: 18 Total (i.e. Null);  16 Residual
Null Deviance:	    131.3
Residual Deviance: 129.3 	AIC: 137.3
> gnlmm(reps, mu=~dose, pmu=c(8.7,0.25), psh=3.5, psd=3)

Call:
gnlmm(reps, mu = ~dose, pmu = c(8.7, 0.25), psh = 3.5, psd = 3)

normal distribution

 with normal mixing distribution on identity scale
 (10 point Gauss-Hermite integration)

Response: y

Log likelihood function:
{
    t <- sh1(p)
    wt * (t + (y - mu4(p))^2/exp(t))/2
}

Location function:
~dose

Log shape function:
structure(p[1] * rep(1, n)), srcfile = <environment>, wholeSrcref = structure(c(1L,
0L, 2L, 0L, 0L, 0L, 1L, 2L), srcfile = <environment>, class = "srcref")

-Log likelihood    64.65019
Degrees of freedom 16
AIC                68.65019
Iterations         18

Location parameters:
             estimate      se
(Intercept)    8.7106  4.4459
dose           0.2491  0.4467

Mixing standard deviation:
   estimate     se
      3.088  1.838

Shape parameters:
      estimate      se
p[1]     3.442  0.3532

Correlations:
         1        2        3        4
1  1.00000 -0.89424  0.08947 -0.04067
2 -0.89424  1.00000 -0.09830  0.04489
3  0.08947 -0.09830  1.00000 -0.18945
4 -0.04067  0.04489 -0.18945  1.00000
> gnlmix(reps, mu=~a+b*dose+rand, random="rand", pmu=c(8.7,0.25),pshape=3.44, pmix=2.3)

Call:
gnlmix(reps, mu = ~a + b * dose + rand, random = "rand", pmu = c(8.7,
    0.25), pshape = 3.44, pmix = 2.3)

normal distribution

normal mixing distribution

Response:

Log likelihood function:
{
    fn <- function(r) mix(p[np], r) * capply(fcn(p, r[nest]) *
        delta^cc, nest, prod)
    -sum(log(inta(fn)))
}

Location function:
~a + b * dose + rand

-Log likelihood    64.64958
Degrees of freedom 16
AIC                68.64958
Iterations         10

Location parameters:
   estimate      se
a    8.7119  4.4449
b    0.2489  0.4466

Shape parameters:
      estimate      se
p[1]     3.441  0.3542

Mixing dispersion parameter:
   estimate     se
      2.259  1.197

Correlations:
         1        2        3        4
1  1.00000 -0.89432 -0.04119  0.08966
2 -0.89432  1.00000  0.04605 -0.09992
3 -0.04119  0.04605  1.00000 -0.19689
4  0.08966 -0.09992 -0.19689  1.00000
> q("no")

````
Note -- there were no valgrind messages after gnlmix.  Looks like we cured it, because when CRAN posted the valgrind output it happened right after gnlmix example in v1.1.3.

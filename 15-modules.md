---
title: Acceder a software a través de Módulos
teaching: 30
exercises: 15
---



::::::::::::::::::::::::::::::::::::::: objectives

- Cargar y usar un paquete de software.
- Explicar cómo cambia el entorno de la shell cuando el mecanismo de módulos carga o descarga paquetes.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo cargamos y descargamos paquetes de software?

::::::::::::::::::::::::::::::::::::::::::::::::::

En un sistema de computación de alto rendimiento, raramente el software que
queremos usar está disponible cuando iniciamos sesión. Está instalado, pero
necesitaremos "cargarlo" antes de que pueda ejecutarse.

Antes de empezar a usar paquetes de software individuales, sin embargo, debemos
entender el razonamiento detrás de este enfoque. Los tres factores más grandes
son:

- incompatibilidades de software
- versiones
- dependencias

La incompatibilidad de software es un dolor de cabeza importante para los
programadores. A veces, la presencia (o ausencia) de un paquete de software
romperá otros que dependen de él. Dos ejemplos bien conocidos son las versiones
de Python y compilador C. Python 3 proporciona famosamente un comando `python`
que entra en conflicto con el proporcionado por Python 2. El software compilado
contra una versión más nueva de las bibliotecas C y luego ejecutado en una
máquina que tiene versiones más antiguas de bibliotecas C instaladas resultará
en un error opaco `'GLIBCXX_3.4.20' not found`.

El versionado del software es otro problema común. Un equipo podría depender de
una cierta versión del paquete para su proyecto de investigación; si la versión
del software cambiara (por ejemplo, si se actualizara un paquete), podría afectar
sus resultados. Tener acceso a múltiples versiones de software permite a un
conjunto de investigadores evitar que los problemas de versiones de software
afecten sus resultados.

Las dependencias son donde un paquete de software particular (o incluso una
versión particular) depende de tener acceso a otro paquete de software (o incluso
una versión particular de otro paquete de software). Por ejemplo, el software de
ciencia de materiales VASP puede requerir una versión particular de la biblioteca
de software FFTW (Transformada rápida de Fourier en el Oeste) para
que funcione.

## Módulos de entorno

Los módulos de entorno son la solución a estos problemas. Un *módulo* es una
descripción autónoma de un paquete de software: contiene la configuración
requerida para ejecutar un paquete de software y, generalmente, codifica las
dependencias requeridas en otros paquetes de software.

Hay varias implementaciones diferentes de módulos de entorno comúnmente usadas en
sistemas HPC: las dos más comunes son *módulos TCL* y *Lmod*. Ambos usan
sintaxis similar y los conceptos son los mismos, por lo que aprender a usar uno
te permitirá usar el que esté instalado en el sistema que estés usando. En ambas
implementaciones, el comando `module` se usa para interactuar con módulos de
entorno. Generalmente se agrega un subcomando adicional al comando para
especificar qué quieres hacer. Para una lista de subcomandos puedes usar
`module -h` o `module help`. Como con todos los comandos, puedes acceder a la
ayuda completa en las páginas *man* con `man module`.

Al iniciar sesión, puede que comiences con un conjunto predeterminado de
módulos cargados o puede que comiences con un entorno vacío; esto depende de la
configura del sistema que estés usando.

### Listar módulos disponibles

Para ver módulos de software disponibles, usa `module avail`:


```bash
[yourUsername@login1 ~]$ module avail | less
```

```output
~~~ /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/modules/all ~~~
  Bazel/3.6.0-GCCcore-x.y.z              NSS/3.51-GCCcore-x.y.z
  Bison/3.5.3-GCCcore-x.y.z              Ninja/1.10.0-GCCcore-x.y.z
  Boost/1.72.0-gompi-2020a               OSU-Micro-Benchmarks/5.6.3-gompi-2020a
  CGAL/4.14.3-gompi-2020a-Python-3.x.y   OpenBLAS/0.3.9-GCC-x.y.z
  CMake/3.16.4-GCCcore-x.y.z             OpenFOAM/v2006-foss-2020a

[removed most of the output here for clarity]

  Where:
   L:        Module is loaded
   D:        Default Module
   Aliases exist: foo/1.2.3 (1.2) means that
             "module load foo/1.2" will load foo/1.2.3

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching
any of the "keys".
```

Ten en cuenta que enviar la salida a través de `less` nos permite buscar dentro de la salida usando la tecla <kbd>/</kbd>.

### Listar módulos cargados actualmente

Puedes usar el comando `module list` para ver qué módulos tienes actualmente
cargados en tu entorno. Si no tienes módulos cargados, verás un mensaje que te
lo dice.

```bash
[yourUsername@login1 ~]$ module list
```

```output
Currently Loaded Modules:
  1) autotools   3) gnu14/14.2.0   5) ucx/1.18.0         7) openmpi5/5.0.7   9) ohpc
  2) prun/2.2    4) hwloc/2.12.0   6) libfabric/1.18.0   8) nvtop/3.2.0

``` 

## Cargar y descargar software

Para cargar un módulo de software, usa `module load`.

En este ejemplo usaremos Python 3. Inicialmente, no está cargado. Podemos
probar esto usando el comando `which`. `which` busca programas de la misma manera
que Bash, así que podemos usarlo para decirnos dónde se almacena un software
particular.

```bash
[yourUsername@login1 ~]$ which python3
```


If the `python3` command was unavailable, we would see output like

```output
/usr/bin/which: no python3 in (/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin:/opt/software/slurm/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/yourUsername/.local/bin:/home/yourUsername/bin)
```

Note that this wall of text is really a list, with values separated
by the `:` character. The output is telling us that the `which` command
searched the following directories for `python3`, without success:

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin
/opt/software/slurm/bin
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
/opt/puppetlabs/bin
/home/yourUsername/.local/bin
/home/yourUsername/bin
```

However, in our case we do have an existing `python3` available so we see

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin/python3
```

We need a different Python than the system provided one though, so let us load
a module to access it.

Podemos cargar el comando `python3` con `module load`:


```bash
[yourUsername@login1 ~]$ module load Python
[yourUsername@login1 ~]$ which python3
```

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin/python3
```

Entonces, ¿qué acaba de suceder?

Para entender la salida, primero necesitamos entender la naturaleza de la
variable de entorno `$PATH`. `$PATH` es una variable de entorno especial que
controla dónde busca software un sistema UNIX. Específicamente, `$PATH` es una
lista de directorios (separados por `:`) que el SO busca a través de un comando
antes de rendirse y decirnos que no puede encontrarlo. Como con todas las
variables de entorno, podemos imprimirlo usando `echo`.

```bash
[yourUsername@login1 ~]$ echo $PATH
```

```output
/opt/ohpc/pub/apps/spack/local/linux-x86_64/python-3.13.5-aloly4qhu2ecv2x3urosyr44oq4hmbn6/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/gettext-0.23.1-qde2xzyizazzp5s47ejgaxyzwc4nfnfd/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/libxml2-2.13.5-z4a4hmomjj5ro423f22s2t32zvlciycd/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/tar-1.35-qe5dfed6ps3v2caxwfdwwqyah7z6pbu4/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/pigz-2.8-2smh4ufgfif5rb3j6m2zkhrca3seaiqg/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/bzip2-1.0.8-wucwh5ce7m7vilxnz56gwqxqzx7nnwzb/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/zstd-1.5.7-shcfzlty5p73tnmstuv2rw5uil6y6qfh/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/libiconv-1.18-446fnouxgsaesaumijbliacukhy3ixde/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/sqlite-3.46.0-ugjl73lgudrl2prwuxinbqtx7j4lzrne/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/openssl-3.4.1-zsrh72hcb4nifdxsddfhbylgofl2dp7x/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/gdbm-1.23-hzp3jssy2pkph7kmjtqyonoqfpd6fhzu/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/expat-2.7.1-zfeeeecjghhwg3sud4lnwk354hyyxava/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/readline-8.2-2sa76zwjjun2hdt7cn75hmt6cutq3egw/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/ncurses-6.5-24imdjdnnqyszfgfquu23k2jcumxzfv2/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/xz-5.6.3-dfp5nxkcjqw55iyzxcg5vcvdud3z2jmw/bin:/opt/ohpc/pub/apps/spack/local/linux-x86_64/util-linux-uuid-2.41-mtu6n2jns45cggjd7x3y7c2bax7oaes5/bin:/usr/local/sbin:/opt/ohpc/pub/apps/nvtop/bin:/opt/ohpc/pub/mpi/libfabric/1.18.0/bin:/opt/ohpc/pub/mpi/ucx-ohpc/1.18.0/bin:/opt/ohpc/pub/libs/hwloc/bin:/opt/ohpc/pub/mpi/openmpi5-gnu14/5.0.7/bin:/opt/ohpc/pub/compiler/gcc/14.2.0/bin:/opt/ohpc/pub/utils/prun/2.2:/opt/ohpc/pub/utils/autotools/bin:/opt/ohpc/pub/bin:/usr/local/bin:/usr/bin:/usr/sbin
```

Notarás una similitud con la salida del comando `which`. En este caso, solo hay
una diferencia: el directorio diferente está al principio. Cuando ejecutamos el
comando `module load`, se agregó un directorio al principio de nuestra `$PATH`
o se "antepuso al `$PATH`". Veamos qué hay allí:


```bash
[yourUsername@login1 ~]$ ls /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin
```

```output
2to3              nosetests-3.8  python                 rst2s5.py
2to3-3.8          pasteurize     python3                rst2xetex.py
chardetect        pbr            python3.8              rst2xml.py
cygdb             pip            python3.8-config       rstpep2html.py
cython            pip3           python3-config         runxlrd.py
cythonize         pip3.8         rst2html4.py           sphinx-apidoc
easy_install      pybabel        rst2html5.py           sphinx-autogen
easy_install-3.8  __pycache__    rst2html.py            sphinx-build
futurize          pydoc3         rst2latex.py           sphinx-quickstart
idle3             pydoc3.8       rst2man.py             tabulate
idle3.8           pygmentize     rst2odt_prepstyles.py  virtualenv
netaddr           pytest         rst2odt.py             wheel
nosetests         py.test        rst2pseudoxml.py
```

En resumen, `module load` agregará software a tu `$PATH`. "Carga"
software. Una nota importante al respecto: dependiendo de qué versión del programa
`module` esté instalada en tu sistema, `module load` también cargará las
dependencias de software necesarias.


To demonstrate, let's use `module list`. `module list` shows all loaded
software modules.

```bash
[yourUsername@login1 ~]$ module list
```

```output
Currently Loaded Modules:
  1) GCCcore/x.y.z                 4) GMP/6.2.0-GCCcore-x.y.z
  2) Tcl/8.6.10-GCCcore-x.y.z      5) libffi/3.3-GCCcore-x.y.z
  3) SQLite/3.31.1-GCCcore-x.y.z   6) Python/3.x.y-GCCcore-x.y.z
```

```bash
[yourUsername@login1 ~]$ module load GROMACS
[yourUsername@login1 ~]$ module list
```

```output
Currently Loaded Modules:
  1) GCCcore/x.y.z                    14) libfabric/1.11.0-GCCcore-x.y.z
  2) Tcl/8.6.10-GCCcore-x.y.z         15) PMIx/3.1.5-GCCcore-x.y.z
  3) SQLite/3.31.1-GCCcore-x.y.z      16) OpenMPI/4.0.3-GCC-x.y.z
  4) GMP/6.2.0-GCCcore-x.y.z          17) OpenBLAS/0.3.9-GCC-x.y.z
  5) libffi/3.3-GCCcore-x.y.z         18) gompi/2020a
  6) Python/3.x.y-GCCcore-x.y.z       19) FFTW/3.3.8-gompi-2020a
  7) GCC/x.y.z                        20) ScaLAPACK/2.1.0-gompi-2020a
  8) numactl/2.0.13-GCCcore-x.y.z     21) foss/2020a
  9) libxml2/2.9.10-GCCcore-x.y.z     22) pybind11/2.4.3-GCCcore-x.y.z-Pytho...
 10) libpciaccess/0.16-GCCcore-x.y.z  23) SciPy-bundle/2020.03-foss-2020a-Py...
 11) hwloc/2.2.0-GCCcore-x.y.z        24) networkx/2.4-foss-2020a-Python-3.8...
 12) libevent/2.1.11-GCCcore-x.y.z    25) GROMACS/2020.1-foss-2020a-Python-3...
 13) UCX/1.8.0-GCCcore-x.y.z
```

So in this case, loading the `GROMACS` module (a bioinformatics software
package), also loaded `GMP/6.2.0-GCCcore-x.y.z` and
`SciPy-bundle/2020.03-foss-2020a-Python-3.x.y` as well. Let's try unloading the
`GROMACS` package.

```bash
[yourUsername@login1 ~]$ module unload GROMACS
[yourUsername@login1 ~]$ module list
```

```output
Currently Loaded Modules:
  1) GCCcore/x.y.z                    13) UCX/1.8.0-GCCcore-x.y.z
  2) Tcl/8.6.10-GCCcore-x.y.z         14) libfabric/1.11.0-GCCcore-x.y.z
  3) SQLite/3.31.1-GCCcore-x.y.z      15) PMIx/3.1.5-GCCcore-x.y.z
  4) GMP/6.2.0-GCCcore-x.y.z          16) OpenMPI/4.0.3-GCC-x.y.z
  5) libffi/3.3-GCCcore-x.y.z         17) OpenBLAS/0.3.9-GCC-x.y.z
  6) Python/3.x.y-GCCcore-x.y.z       18) gompi/2020a
  7) GCC/x.y.z                        19) FFTW/3.3.8-gompi-2020a
  8) numactl/2.0.13-GCCcore-x.y.z     20) ScaLAPACK/2.1.0-gompi-2020a
  9) libxml2/2.9.10-GCCcore-x.y.z     21) foss/2020a
 10) libpciaccess/0.16-GCCcore-x.y.z  22) pybind11/2.4.3-GCCcore-x.y.z-Pytho...
 11) hwloc/2.2.0-GCCcore-x.y.z        23) SciPy-bundle/2020.03-foss-2020a-Py...
 12) libevent/2.1.11-GCCcore-x.y.z    24) networkx/2.4-foss-2020a-Python-3.x.y
```

So using `module unload` "un-loads" a module, and depending on how a site is
configured it may also unload all of the dependencies (in our case it does
not). If we wanted to unload everything at once, we could run `module purge`
(unloads everything).

```bash
[yourUsername@login1 ~]$ module purge
[yourUsername@login1 ~]$ module list
```

```output
No modules loaded
```

Note that `module purge` is informative. It will also let us know if a default
set of "sticky" packages cannot be unloaded (and how to actually unload these
if we truly so desired).

Ten en cuenta que este proceso de carga de módulos sucede principalmente a través
de la manipulación de variables de entorno como `$PATH`. Generalmente hay poco o
ningún movimiento de datos involucrado.

El proceso de carga de módulos también manipula otras variables de entorno
especiales, incluidas variables que influyen en dónde busca el sistema
bibliotecas de software, y a veces variables que le dicen a paquetes de software
comercial dónde encontrar servidores de licencia.

El comando module también restaura estas variables de entorno de la shell a su
estado anterior cuando un módulo se descarga.

## Versioning del software

Hasta ahora, hemos aprendido cómo cargar y descargar paquetes de software. Esto
es muy útil. Sin embargo, aún no hemos abordado el problema del versionado del
software. En algún momento u otro, te encontrarás con problemas donde solo una
versión particular de algún software será adecuada. Quizá la corrección de un error
clave solo sucedió en una versión determinada, o la versión X rompió la
compatibilidad con un formato de archivo que usas. En cualquiera de estos casos
de ejemplo, es útil ser muy específico sobre qué versión de software estas cargado.

Examinemos la salida de `module avail` más de cerca, usando el paginador ya que
puede haber montones de salida:


```bash
[yourUsername@login1 ~]$ module avail | less
```

```output
~~~ /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/modules/all ~~~
  Bazel/3.6.0-GCCcore-x.y.z              NSS/3.51-GCCcore-x.y.z
  Bison/3.5.3-GCCcore-x.y.z              Ninja/1.10.0-GCCcore-x.y.z
  Boost/1.72.0-gompi-2020a               OSU-Micro-Benchmarks/5.6.3-gompi-2020a
  CGAL/4.14.3-gompi-2020a-Python-3.x.y   OpenBLAS/0.3.9-GCC-x.y.z
  CMake/3.16.4-GCCcore-x.y.z             OpenFOAM/v2006-foss-2020a

[removed most of the output here for clarity]

  Where:
   L:        Module is loaded
   D:        Default Module
   Aliases exist: foo/1.2.3 (1.2) means that
             "module load foo/1.2" will load foo/1.2.3

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching
any of the "keys".
```

Si el software que ejecuta tu script de Slurm requiere una versión específica de 
una dependencia, asegúrate de utilizar el nombre completo del módulo, en lugar de 
la versión predeterminada (_default_) que se carga cuando se indica solo su nombre 
(hasta la primera barra `/`)

:::::::::::::::::::::::::::::::::::::::  challenge

## Usar módulos de software en scripts

Crea un trabajo que pueda ejecutar `python3 --version`. Recuerda, ¡ningún
software está cargado por defecto! Ejecutar un trabajo es como iniciar sesión en
el sistema (no debes asumir que un módulo cargado en el nodo de acceso está
cargado en un nodo de cómputo).

:::::::::::::::  solution

## Solución

```bash
[yourUsername@login1 ~]$ nano python-module.slurm
[yourUsername@login1 ~]$ cat python-module.slurm
```

```output
#!/bin/bash
#SBATCH 

#SBATCH -t 00:00:30

module load Python

python3 --version
```

```bash
[yourUsername@login1 ~]$ sbatch  python-module.slurm
```

Es recomendable guardar tus scripts de trabajo con la extensión `.slurm`, ya que así podrás diferenciarlos fácilmente de los scripts de shell tradicionales.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Carga software con `module load nombreDelSoftware`.
- Descarga  software con `module unload`.
- El sistema de módulos gestiona automáticamente el versionado del software y los conflictos entre paquetes.

::::::::::::::::::::::::::::::::::::::::::::::::::

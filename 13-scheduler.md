---
title: "Fundamentos del Planificador de Trabajos"
teaching: 45
exercises: 30
---



::::::::::::::::::::::::::::::::::::::: objectives

- Enviar un script simple al clúster.
- Monitorear la ejecución de trabajos usando herramientas de línea de comandos.
- Inspeccionar los archivos de salida y error de tus trabajos.
- Encontrar el lugar adecuado para poner conjuntos de datos grandes en el clúster.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Qué es un planificador de trabajos y por qué lo necesita un clúster?
- ¿Cómo lanzo un programa para que se ejecute en un nodo de cómputo del clúster?
- ¿Cómo capturo la salida de un programa que se ejecuta en un nodo del clúster?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Planificador de trabajos

Un sistema HPC puede tener miles de nodos y miles de usuarios. ¿Cómo decidimos
quién obtiene qué y cuándo? ¿Cómo aseguramos que una tarea se ejecute con los
recursos que necesita? Este trabajo lo realiza un software especial llamado
*gestor de colas* o *planificador*. En un sistema HPC, el planificador gestiona qué trabajos se
ejecutan, dónde y cuándo.

La siguiente ilustración compara estas tareas de un planificador de trabajos
con un mesero en un restaurante. Si puedes relacionarlo con alguna ocasión en
la que tuviste que esperar en una fila para entrar a un restaurante popular,
entonces puedes entender por qué a veces tu trabajo no inicia de inmediato
como en tu laptop.

![](fig/restaurant_queue_manager.svg){alt="Comparar un planificador de trabajos con un mesero en un restaurante" max-width="75%"}

El planificador usado en esta lección es Slurm. Aunque
Slurm no se usa en todos lados, ejecutar trabajos es bastante
similar sin importar el software que se utilice. La sintaxis exacta puede
cambiar, pero los conceptos permanecen iguales.

## Ejecutar un trabajo por lotes (batch)

El uso más básico del planificador es ejecutar un comando de forma no
interactiva. Cualquier comando (o serie de comandos) que quieras ejecutar en el
clúster se llama *trabajo*, y el proceso de usar un planificador para ejecutar
el trabajo se llama *envío de trabajos por lotes* o *batch job submission*.

En este caso, el trabajo que queremos ejecutar es un script de shell: en
esencia, un archivo de texto que contiene una lista de comandos UNIX que se
ejecutan de manera secuencial. Nuestro script tendrá tres partes:

- En la primera línea, agrega `#!/bin/bash`. El `#!`
  (pronunciado "hash-bang" o "shebang") le dice a la computadora qué programa
  debe procesar el contenido de este archivo. En este caso, le indicamos que
  los comandos que siguen están escritos para la shell de línea de comandos (la
  que hemos estado usando hasta ahora).
- En cualquier lugar debajo de la primera línea, agregaremos un comando `echo`
  con un saludo amistoso. Al ejecutarse, el script imprimirá en la terminal
  todo lo que venga después de `echo`.
  - `echo -n` imprimirá todo lo que sigue, *sin* terminar la línea con el
    carácter de nueva línea.
- En la última línea, invocaremos el comando `hostname`, que imprimirá el
  nombre de la máquina donde se ejecuta el script.

```bash
[yourUsername@login1 ~]$ nano example-job.sh
```

```bash
#!/bin/bash

echo -n "This script is running on "
hostname
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Creando nuestro trabajo de prueba

Ejecuta el script. ¿Se ejecuta en el clúster o solo en nuestro nodo de acceso?

:::::::::::::::  solution

## Solución

```bash
[yourUsername@login1 ~]$ bash example-job.sh
```

```output
This script is running on login1
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Este script se ejecutó en el nodo de acceso, pero queremos aprovechar los
nodos de cómputo: necesitamos que el planificador ponga `example-job.sh` en
cola para ejecutarlo en un nodo de cómputo.

Para enviar esta tarea al planificador, usamos el comando
`sbatch`.
Esto crea un *trabajo* que ejecutará el *script* cuando sea *despachado* a un
nodo de cómputo que el sistema de colas ha identificado como disponible para
realizar el trabajo.

```bash
[yourUsername@login1 ~]$ sbatch  example-job.sh
```


```output
Submitted batch job 7
```

Y eso es todo lo que necesitamos para enviar un trabajo. Nuestro trabajo está
hecho: ahora el planificador se encarga y trata de ejecutarlo por nosotros.
Mientras el trabajo espera para ejecutarse, entra en una lista de trabajos
llamada *cola*. Para revisar el estado del trabajo, consultamos la cola con el
comando `squeue -u yourUsername`.

```bash
[yourUsername@login1 ~]$ squeue -u yourUsername
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
    9 cpubase_b example-   user01  R       0:05      1 node1
```

Podemos ver todos los detalles de nuestro trabajo, y lo más importante es que
está en estado `R` o `RUNNING`. A veces nuestros trabajos deben esperar en la
cola (`PENDING`) o presentar un error (`E`).

::::::::::::::::::::::::::::::::::::::  discussion

## ¿Dónde está la salida?

En el nodo de acceso, este script imprimió la salida en la terminal, pero
ahora, cuando `squeue` muestra que el trabajo terminó,
no se imprimió nada en la terminal.

La salida de un trabajo en el clúster normalmente se redirige a un archivo en
el directorio desde el que lo lanzaste. Usa `ls` para encontrarlo y `cat` para
leerlo.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Personalizar un trabajo

El trabajo que acabamos de ejecutar usó todas las opciones predeterminadas del
planificador. En un escenario real, probablemente no es lo que queremos. Las
opciones predeterminadas representan un mínimo razonable. Es probable que
necesitemos más núcleos, más memoria, más tiempo, entre otras consideraciones
especiales. Para acceder a esos recursos debemos personalizar nuestro script.

Los comentarios en scripts de shell UNIX (marcados con `#`) normalmente se
ignoran, pero hay excepciones. Por ejemplo, el comentario especial `#!` al
inicio de los scripts especifica qué programa debe usarse para ejecutarlo
(normalmente verás `#!/bin/bash`). Planificadores como
Slurm también tienen un comentario especial que se usa para
indicar opciones específicas del planificador. Aunque estos comentarios difieren
entre planificadores, el comentario especial de Slurm es
`#SBATCH`. Cualquier cosa que siga al comentario
`#SBATCH` se interpreta como una instrucción al planificador.

Ilustremos esto con un ejemplo. Por defecto, el nombre de un trabajo es el
nombre del script, pero la opción `-J` se puede usar
para cambiar el nombre de un trabajo. Agrega una opción al script:

```bash
[yourUsername@login1 ~]$ cat example-job.sh
```

```bash
#!/bin/bash
#SBATCH -J hello-world

echo -n "This script is running on "
hostname
```

Envía el trabajo y monitorea su estado:

```bash
[yourUsername@login1 ~]$ sbatch  example-job.sh
[yourUsername@login1 ~]$ squeue -u yourUsername
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   10 cpubase_b hello-wo   user01  R       0:02      1 node1
```

¡Excelente, cambiamos con éxito el nombre de nuestro trabajo!

### Solicitudes de recursos

¿Qué hay de cambios más importantes, como el número de núcleos y la memoria
para nuestros trabajos? Algo absolutamente crítico cuando trabajamos en un
sistema HPC es especificar los recursos necesarios para ejecutar un trabajo.
Esto permite que el planificador encuentre el momento y el lugar adecuados para
programar nuestro trabajo. Si no especificas requisitos (como el tiempo que
necesitas), probablemente quedarás limitado a los recursos predeterminados de
tu sitio, lo cual no es lo que deseas.

Estas son algunas solicitudes clave de recursos:

- `--ntasks=<ntasks>` o `-n <ntasks>`: ¿Cuántos núcleos de CPU necesita tu
  trabajo en total?

- `--time <days-hours:minutes:seconds>` o `-t <days-hours:minutes:seconds>`:
  ¿Cuánto tiempo real (walltime) tardará en ejecutarse tu trabajo? La parte
  `<days>` se puede omitir.

- `--mem=<megabytes>`: ¿Cuánta memoria en un nodo necesita tu trabajo en
  megabytes? También puedes especificar gigabytes añadiendo una "g"
  después (ejemplo: `--mem=5g`)

- `--nodes=<nnodes>` o `-N <nnodes>`: ¿En cuántas máquinas separadas necesita
  ejecutarse tu trabajo? Ten en cuenta que si estableces `ntasks` en un número
  mayor que lo que puede ofrecer una sola máquina, Slurm
  ajustará este valor automáticamente.

Ten en cuenta que solo *solicitar* estos recursos no hace que tu trabajo corra
más rápido, ni significa necesariamente que consumirás todos esos recursos. Solo
significa que se ponen a tu disposición. Tu trabajo puede terminar usando menos
memoria, menos tiempo o menos nodos de los que solicitaste, y aun así se
ejecutará.

Es mejor que tus solicitudes reflejen con precisión los requisitos de tu
trabajo. Hablaremos más sobre cómo asegurarte de usar los recursos de manera
efectiva en un episodio posterior de esta lección.

:::::::::::::::::::::::::::::::::::::::  challenge

## Enviar solicitudes de recursos

Modifica nuestro script `hostname` para que se ejecute durante un minuto, y
luego envía un trabajo para él en el clúster.

:::::::::::::::  solution

## Solución

```bash
[yourUsername@login1 ~]$ cat example-job.sh
```

```bash
#!/bin/bash
#SBATCH -t 00:01 # timeout in HH:MM

echo -n "This script is running on "
sleep 20 # time in seconds
hostname
```

```bash
[yourUsername@login1 ~]$ sbatch  example-job.sh
```

¿Por qué el tiempo de ejecución de Slurm y el tiempo de `sleep`
no son idénticos?



:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

Las solicitudes de recursos suelen ser obligatorias. Si las excedes, tu trabajo
será terminado. Usemos el tiempo de ejecución real (*wall time*) como ejemplo. Solicitaremos 1 minuto
de tiempo real e intentaremos ejecutar un trabajo durante dos minutos.

```bash
[yourUsername@login1 ~]$ cat example-job.sh
```

```bash
#!/bin/bash
#SBATCH -J long_job
#SBATCH -t 00:01 # timeout in HH:MM

echo "This script is running on ... "
sleep 240 # time in seconds
hostname
```

Envía el trabajo y espera a que termine. Una vez que haya terminado, revisa el
archivo de registro.

```bash
[yourUsername@login1 ~]$ sbatch  example-job.sh
[yourUsername@login1 ~]$ squeue -u yourUsername
```

```bash
[yourUsername@login1 ~]$ cat slurm-12.out
```

```output
This script is running on ...
slurmstepd: error: *** JOB 12 ON node1 CANCELLED AT 2021-02-19T13:55:57
DUE TO TIME LIMIT ***
```

Nuestro trabajo fue terminado por exceder la cantidad de recursos solicitados.
Aunque esto parezca duro, en realidad es una característica. El cumplimiento
estricto de las solicitudes de recursos permite que el planificador encuentre
el mejor lugar posible para tus trabajos. Aún más importante, asegura que otra
persona usuaria no pueda usar más recursos de los que se le asignaron. Si otra
persona se equivoca y accidentalmente intenta usar todos los núcleos o la
memoria de un nodo, Slurm restringirá su trabajo a los recursos
solicitados o lo terminará de inmediato. Los otros trabajos en el nodo no se
verán afectados. Esto significa que una persona no puede arruinar la experiencia
de las demás; los únicos trabajos afectados por un error en la planificación
serán los suyos.

## Cancelar un trabajo

A veces cometeremos un error y necesitaremos cancelar un trabajo. Esto se puede
hacer con el comando `scancel`. Enviemos un trabajo y luego
cancelémoslo usando su número de trabajo (recuerda cambiar el tiempo de ejecución real (*wall time*)
para que se ejecute lo suficiente como para cancelarlo antes de que sea
terminado).

```bash
[yourUsername@login1 ~]$ sbatch  example-job.sh
[yourUsername@login1 ~]$ squeue -u yourUsername
```

```output
Submitted batch job 13

JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   13 cpubase_b long_job   user01  R       0:02      1 node1
```

Ahora cancela el trabajo con su número (impreso en tu terminal). El retorno
limpio del prompt indica que la solicitud de cancelación fue exitosa.

```bash
[yourUsername@login1 ~]$ scancel 38759
# It might take a minute for the job to disappear from the queue...
[yourUsername@login1 ~]$ squeue -u yourUsername
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Cancelar múltiples trabajos

También podemos cancelar todos nuestros trabajos a la vez usando la opción
`-u`. Esto eliminará todos los trabajos de una persona usuaria específica (en
este caso, tú). Ten en cuenta que solo puedes eliminar tus propios trabajos.

Intenta enviar varios trabajos y luego cancelarlos todos.

:::::::::::::::  solution

## Solución

Primero, envía un trío de trabajos:

```bash
[yourUsername@login1 ~]$ sbatch  example-job.sh
[yourUsername@login1 ~]$ sbatch  example-job.sh
[yourUsername@login1 ~]$ sbatch  example-job.sh
```

Luego, cancélalos todos:

```bash
[yourUsername@login1 ~]$ scancel -u yourUsername
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Otros tipos de trabajos

Hasta este punto, nos hemos enfocado en ejecutar trabajos en modo por lotes (*batch mode*).
`Slurm` también ofrece la posibilidad de iniciar una sesión
interactiva.

Con mucha frecuencia hay tareas que deben hacerse de forma interactiva. Crear
un script de trabajo completo puede ser excesivo, pero la cantidad de recursos
requerida es demasiada para un nodo de acceso. Un buen ejemplo sería construir
un índice genómico para alineamiento con una herramienta como [HISAT2][hisat].
Por suerte, podemos ejecutar este tipo de tareas de una sola vez con
`srun`.

`srun` ejecuta un solo comando en el clúster y luego
sale. Demostremos esto ejecutando el comando `hostname` con
`srun`. (Podemos cancelar un trabajo de
`srun` con `Ctrl-c`.)

```bash
[yourUsername@login1 ~]$ srun hostname
```

```output
smnode1
```

`srun` acepta todas las mismas opciones que
`sbatch`. Sin embargo, en lugar de especificarlas en un
script, estas opciones se indican en la línea de comandos al iniciar un trabajo.
Por ejemplo, para enviar un trabajo que use 2 CPU, podríamos usar el siguiente
comando:

```bash
[yourUsername@login1 ~]$ srun -n 2 echo "This job will use 2 CPUs."
```

```output
This job will use 2 CPUs.
This job will use 2 CPUs.
```

Por lo general, el entorno de la shell resultante será el mismo que el de
`sbatch`.

### Trabajos interactivos

A veces necesitarás muchos recursos para uso interactivo. Tal vez sea nuestra
primera vez ejecutando un análisis o estemos intentando depurar algo que salió
mal en un trabajo anterior. Afortunadamente, Slurm facilita
iniciar un trabajo interactivo con `srun`:

```bash
[yourUsername@login1 ~]$ srun --pty bash
```

Deberías ver un prompt de bash. Observa que el prompt probablemente cambiará
para reflejar tu nueva ubicación, en este caso el nodo de cómputo en el que
estamos conectados. También puedes verificarlo con `hostname`.

:::::::::::::::::::::::::::::::::::::::::  callout

## Crear gráficos remotos

Para ver salida gráfica dentro de tus trabajos, necesitas usar reenvío X11.
Para conectar con esta función habilitada, usa la opción `-X` cuando inicies
sesión con el comando `ssh`, por ejemplo, `ssh -X yourUsername@cluster.hpc-carpentry.org`.

Para demostrar lo que sucede cuando creas una ventana gráfica en el nodo
remoto, usa el comando `xeyes`. Debería aparecer un par de ojos bastante
adorables (presiona `Ctrl-C` para detener). Si usas Mac, debes haber instalado
XQuartz (y reiniciado tu computadora) para que esto funcione.

Si tu clúster tiene instalado el plugin
[slurm-spank-x11](https://github.com/hautreux/slurm-spank-x11), puedes asegurar
el reenvío X11 dentro de trabajos interactivos usando la opción `--x11` para
`srun` con el comando
`srun --x11 --pty bash`.

::::::::::::::::::::::::::::::::::::::::::::::::::

Cuando termines con el trabajo interactivo, escribe `exit` para salir de la
sesión.

[hisat]: https://daehwankimlab.github.io/hisat2/

:::::::::::::::::::::::::::::::::::::::: keypoints

- El planificador gestiona cómo se comparten los recursos de cómputo entre
  usuarios.
- Un trabajo es solo un script de shell.
- Solicita *ligeramente* más recursos de los que necesitarás.

::::::::::::::::::::::::::::::::::::::::::::::::::



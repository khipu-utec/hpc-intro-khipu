---
title: Usar recursos de forma eficaz
teaching: 10
exercises: 20
---



::::::::::::::::::::::::::::::::::::::: objectives

- Consultar estadísticas de trabajos.
- Hacer solicitudes de recursos más precisas en los scripts de trabajo basadas en datos
  que describen el rendimiento pasado.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- ¿Cómo puedo revisar trabajos pasados?
- ¿Cómo puedo usar este conocimiento para crear un script de envío más preciso?

::::::::::::::::::::::::::::::::::::::::::::::::::

Hemos tocado todas las habilidades que necesita para interactuar con un clúster HPC:
iniciar sesión por SSH, cargar módulos de software, enviar trabajos paralelos y
encontrar la salida. Aprendamos sobre la estimación del uso de recursos y por qué
podría importar.

## Estimar Recursos Requeridos Usando el Planificador

Aunque ya hablamos sobre cómo solicitar recursos al planificador (*scheduler*) anteriormente,
¿cómo sabemos qué tipo de recursos necesitará el software en
primer lugar, y su demanda de cada uno? En general, a menos que la
documentación del software o los testimonios de usuarios proporcionen alguna idea, no sabremos cuánta
memoria o tiempo de cómputo necesitará un programa.

:::::::::::::::::::::::::::::::::::::::::  callout

## Leer la Documentación

La mayoría de los centros HPC mantienen documentación como un wiki, un sitio web o un
documento enviado cuando se registra para una cuenta. Eche un vistazo a estos
recursos y busque el software que planea usar: alguien podría haber
redactado orientación para sacarle el máximo provecho.

::::::::::::::::::::::::::::::::::::::::::::::::::

Una forma conveniente de averiguar los recursos necesarios para que un trabajo se ejecute
con éxito es enviar un trabajo de prueba, y luego preguntar al planificador sobre su
impacto usando `sacct -u yourUsername`. Puede usar este conocimiento para configurar el
siguiente trabajo con una estimación más cercana de su carga en el sistema. Una buena regla general
es solicitar al planificador entre 20% y 30% más tiempo y memoria de lo que espera que el
trabajo necesite. Esto asegura que pequeñas fluctuaciones en el tiempo de ejecución o el uso de memoria
no resulten en la cancelación de su trabajo por parte del planificador. Tenga en cuenta que
si pide demasiado, su trabajo podría no ejecutarse aunque haya suficientes recursos
disponibles, porque el planificador estará esperando que los trabajos de otras personas terminen y
liberen los recursos necesarios para igualar lo que usted pidió.

## Estadísticas

Dado que ya enviamos `amdahl` para ejecutarse en el clúster, podemos consultar al
planificador para ver cuánto tiempo tomó nuestro trabajo y qué recursos se usaron. Usaremos
`sacct -u yourUsername` para obtener estadísticas sobre `parallel-job.sh`.

```bash
[yourUsername@login1 ~]$ sacct -u yourUsername
```


```output
       JobID    JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
7               file.sh cpubase_b+ def-spons+          1  COMPLETED      0:0
7.batch           batch            def-spons+          1  COMPLETED      0:0
7.extern         extern            def-spons+          1  COMPLETED      0:0
8               file.sh cpubase_b+ def-spons+          1  COMPLETED      0:0
8.batch           batch            def-spons+          1  COMPLETED      0:0
8.extern         extern            def-spons+          1  COMPLETED      0:0
9            example-j+ cpubase_b+ def-spons+          1  COMPLETED      0:0
9.batch           batch            def-spons+          1  COMPLETED      0:0
9.extern         extern            def-spons+          1  COMPLETED      0:0
```

Esto muestra todos los trabajos que ejecutamos hoy (note que hay múltiples entradas por
trabajo).
Para obtener información sobre un trabajo específico (por ejemplo, 347087), cambiamos el comando
ligeramente.

```bash
[yourUsername@login1 ~]$ sacct -u yourUsername -l -j 347087
```

Mostrará mucha información; de hecho, cada pieza de información recopilada sobre
su trabajo por el planificador aparecerá aquí. Puede ser útil redirigir esta
información a `less` para facilitar su visualización (use las teclas de flecha izquierda y derecha
para desplazarse por los campos).

```bash
[yourUsername@login1 ~]$ sacct -u yourUsername -l -j 347087 | less -S
```

::::::::::::::::::::::::::::::::::::::  discussion

## Discusión

Esta vista puede ayudar a comparar la cantidad de tiempo solicitada y la realmente
utilizada, la duración de permanencia en la cola antes del lanzamiento y la huella de memoria
en el(los) nodo(s) de cálculo.

¿Qué tan precisas fueron nuestras estimaciones?


::::::::::::::::::::::::::::::::::::::::::::::::::

## Mejorar Solicitudes de Recursos

Del historial de trabajos, vemos que los trabajos `amdahl` terminaron de ejecutarse en
como mucho unos minutos, una vez despachados. ¡La estimación de tiempo que proporcionamos
en el script de trabajo fue demasiado larga! Esto hace más difícil para el
sistema de colas estimar con precisión cuándo los recursos quedarán libres
para otros trabajos. En la práctica, esto significa que el sistema de colas espera
para despachar nuestro trabajo `amdahl` hasta que se abra el intervalo completo solicitado,
en lugar de "colarlo" en una ventana mucho más corta donde el trabajo podría
realmente terminar. Especificar el tiempo de ejecución esperado en el script de envío
con mayor precisión ayudará a aliviar la congestión del clúster y puede
hacer que su trabajo se despache antes.

:::::::::::::::::::::::::::::::::::::::  challenge

## Ajustar la Estimación de Tiempo

Edite `parallel_job.sh` para establecer una mejor estimación de tiempo. ¿Qué tan cerca puede
estar?

Pista: use `-t`.

:::::::::::::::  solution

## Solución

La siguiente línea le indica a Slurm que nuestro trabajo debería
terminar en 2 minutos:

```bash
#SBATCH -t 00:02:00
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Los scripts de trabajo precisos ayudan al sistema de colas a asignar eficientemente los recursos compartidos.

::::::::::::::::::::::::::::::::::::::::::::::::::


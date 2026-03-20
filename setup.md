---
title: Setup
---

Hay varios programas que deberá instalar antes del taller. Aunque se proporcionará ayuda para la instalación durante el taller, recomendamos que estas herramientas se instalen (o al menos se descarguen) previamente.

1. [Una aplicación de terminal o interfaz de línea de comandos](#dónde-escribir-comandos-cómo-abrir-una-nueva-terminal)
2. [Una aplicación de Secure Shell](#ssh-para-conexiones-seguras)

::::::::::::::::::::::::::::::::::::::::::  prereq

## Bash y SSH

Esta lección requiere una aplicación de terminal (`bash`, `zsh` u otras) con
la capacidad de conectarse de manera segura a una máquina remota (`ssh`).


::::::::::::::::::::::::::::::::::::::::::::::::::

## Dónde escribir comandos: Cómo abrir una nueva terminal

La terminal es un programa que nos permite enviar comandos a la computadora y
recibir resultados. También se le conoce como línea de comandos o shell.

Algunas computadoras incluyen un programa de shell Unix por defecto. Los pasos a continuación describen algunos métodos para identificar y abrir una shell Unix si ya tiene una instalada. También hay opciones para identificar y descargar un programa de shell Unix, un emulador de Linux/UNIX o un programa para acceder a un shell Unix en un servidor.

### Unix Shells en Windows

Las computadoras con sistemas operativos Windows no tienen automáticamente un programa de shell Unix instalado. En esta lección, recomendamos utilizar un emulador incluido en Git para Windows, que le da acceso tanto a los comandos de la shell Bash como a Git. Si ha asistido a una sesión de taller de Software Carpentry, es probable que ya haya recibido instrucciones sobre cómo instalar Git para Windows.

Una vez instalado, puede abrir una terminal ejecutando el programa Git Bash desde el menú de inicio de Windows.

#### Programas de Shell para Windows

- [Git for Windows][git4win] -- *Recomendado*
- [Windows Subsystem for Linux][wsl] -- opción avanzada para Windows 10

::::::::::::::::::::::::::::::::::::::  discussion

## Alternativas a Git para Windows

Otras soluciones están disponibles para ejecutar comandos Bash en Windows. Ahora hay una herramienta de línea de comandos de Bash disponible para Windows 10. Además, puede ejecutar comandos Bash en una computadora o servidor remoto que ya tenga un shell Unix, desde su máquina con Windows. Esto generalmente se puede hacer a través de un cliente de Secure Shell (SSH). Un cliente disponible de forma gratuita para computadoras con Windows es [PuTTY][putty]. Consulte la referencia a continuación para obtener información sobre cómo instalar y usar PuTTY, usar la herramienta de línea de comandos de Windows 10 o instalar y usar un emulador de Unix/Linux.

Para usuarios avanzados, puede elegir una de las siguientes alternativas:

- Instalar el [Windows Subsystem for Linux][wsl]
- Usar el [PowerShell][ms-shell] de Windows
- Leer sobre [Uso de un emulador de Unix/Linux][unix-emulator] (Cygwin) o cliente de Secure Shell (SSH) (PuTTY)

:::::::::::::::::::::::::::::::::::::::  challenge

## Advertencia

Los comandos en el Windows Subsystem for Linux (WSL), PowerShell o Cygwin
pueden diferir ligeramente de los mostrados en la lección o presentados en el
taller. Por favor, pregunte si encuentra tal discrepancia: probablemente no esté solo.



::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

### Unix Shell en macOS

En macOS, el shell Unix predeterminado es accesible ejecutando el programa Terminal desde la carpeta `/Application/Utilities` en Finder.

Para abrir Terminal, intente uno o ambos de los siguientes métodos:

- En Finder, seleccione el menú Ir y luego seleccione Utilidades. Localice Terminal en la carpeta de Utilidades y ábralo.
- Use la función de búsqueda de la computadora 'Spotlight' de Mac. Busque: `Terminal` y presione <kbd>Return</kbd>.

Para una introducción, consulte [Cómo usar Terminal en una Mac][mac-terminal].

### Unix Shell en Linux

En la mayoría de las versiones de Linux, el shell Unix predeterminado es accesible ejecutando el
[(Gnome) Terminal](https://help.gnome.org/users/gnome-terminal/stable/) o
[(KDE) Konsole](https://konsole.kde.org/) o
[xterm](https://en.wikipedia.org/wiki/Xterm), que se pueden encontrar a través del
menú de aplicaciones o la barra de búsqueda.

### Casos Especiales

Si ninguna de las opciones anteriores aborda su situación, intente una búsqueda en línea
con: `Unix shell [su sistema operativo]`.

## SSH para Conexiones Seguras

Todos los estudiantes deben tener un cliente SSH instalado. SSH es una herramienta que nos permite
conectarnos y usar una computadora remota como si fuera nuestra.

### SSH para Windows

Git para Windows viene con SSH preinstalado: no tiene que hacer nada.

::::::::::::::::::::::::::::::::::::::  discussion

## Soporte gráfico (GUI) para Windows

Si sabe que el software que ejecutará en el clúster requiere una
interfaz gráfica de usuario (una ventana GUI necesita abrirse para que la aplicación se
ejecute correctamente), instale [MobaXterm](https://mobaxterm.mobatek.net) Home
Edition.


::::::::::::::::::::::::::::::::::::::::::::::::::

### SSH para macOS

macOS viene con SSH preinstalado: no tiene que hacer nada.

::::::::::::::::::::::::::::::::::::::  discussion

## Soporte gráfico (GUI) para macOS

Si sabe que el software que ejecutará requiere una interfaz gráfica de usuario, instale [XQuartz](https://www.xquartz.org).


::::::::::::::::::::::::::::::::::::::::::::::::::

### SSH para Linux

Linux viene con SSH y soporte para ventanas X preinstalado: no tiene que hacer
nada.

<!-- links -->

[git4win]: https://gitforwindows.org/
[wsl]: https://docs.microsoft.com/en-us/windows/wsl/install-win10
[putty]: https://www.chiark.greenend.org.uk/~sgtatham/putty/
[ms-shell]: https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core?view=powershell-7
[unix-emulator]: https://www.cygwin.com/
[mac-terminal]: https://www.macworld.co.uk/feature/mac-software/how-use-terminal-on-mac-3608274/




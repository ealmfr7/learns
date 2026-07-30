# 01 — Del código fuente al proceso

## Pregunta central

Cuando ejecutas:

```bash
gcc programa.c -o programa
./programa
```

¿qué entidades distintas participan y qué transforma cada una?

La respuesta completa no es simplemente “GCC compila y la CPU lo ejecuta”.
Durante este módulo construiremos una explicación precisa, desde el texto en
`programa.c` hasta la primera instrucción de `main`.

## Modelo inicial

Antes de entrar en detalles, conserva este mapa:

```text
programa.c
    │ preprocesador
    ▼
programa.i
    │ compilador
    ▼
programa.s
    │ ensamblador
    ▼
programa.o
    │ linker
    ▼
programa (ELF)
    │ kernel + loader dinámico + runtime de C
    ▼
proceso en ejecución
```

Cada flecha representa una transformación diferente. Un archivo ejecutable y un
proceso tampoco son la misma cosa: uno son bytes persistentes en almacenamiento;
el otro es un estado vivo administrado por el kernel.

## Objetivos de comprensión

Al cerrar el módulo podrás:

- Diferenciar preprocesamiento, compilación, ensamblado y enlazado.
- Explicar por qué un archivo objeto todavía no es normalmente un ejecutable.
- Distinguir código fuente, ensamblador, código máquina, ELF y proceso.
- Inspeccionar las etapas sin aceptar `gcc` como una caja negra.
- Explicar por qué el kernel no comienza directamente en `main`.
- Separar las responsabilidades del kernel, loader dinámico, runtime de C y CPU.

## Laboratorio

Trabajaremos dentro de:

```bash
cd c-low-level/01-del-codigo-al-proceso/laboratorio
```

Comprueba primero las herramientas disponibles:

```bash
gcc --version
file --version
readelf --version
objdump --version
nm --version
```

En Ubuntu, si falta alguna herramienta:

```bash
sudo apt update
sudo apt install build-essential binutils file gdb
```

### Experimento 1: hacer visible el pipeline

No ejecutes todos los comandos de golpe. Antes de cada uno, predice qué clase de
archivo aparecerá y si la CPU podría ejecutarlo directamente.

```bash
# 1. Preprocesamiento
gcc -E programa.c -o programa.i

# 2. Traducción a ensamblador
gcc -S -O0 -masm=intel programa.i -o programa.s

# 3. Ensamblado a código máquina relocatable
gcc -c programa.s -o programa.o

# 4. Enlazado a un ejecutable ELF
gcc programa.o -o programa
```

Inspecciona la naturaleza de cada resultado:

```bash
file programa.c programa.i programa.s programa.o programa
wc -c programa.c programa.i programa.s programa.o programa
```

Preguntas de control:

1. ¿Por qué el archivo preprocesado puede ser mucho mayor que el original?
2. ¿`programa.o` contiene instrucciones de máquina? Si las contiene, ¿por qué no
   funciona como un ejecutable normal?
3. ¿Por qué el ejecutable puede ser mayor que el objeto si nuestro código no
   creció?

### Experimento 2: observar símbolos y punto de entrada

```bash
nm programa.o
nm programa
readelf -h programa
readelf -S programa
readelf -l programa
```

Busca `main`, `sumar` y el valor de `Entry point address`.

Preguntas de control:

1. ¿La dirección de entrada indicada por ELF coincide necesariamente con
   `main`?
2. ¿Qué diferencia conceptual hay entre una sección de ELF y un segmento
   cargado en memoria?
3. ¿Qué símbolos existen en el ejecutable que no escribimos en `programa.c`?

### Experimento 3: código máquina frente a ensamblador

```bash
objdump -d -M intel programa.o
objdump -d -M intel programa
```

El ensamblador mostrado es una representación textual de instrucciones ya
codificadas como bytes. No confundas `programa.s`, que es entrada textual para
el assembler, con el desensamblado que `objdump` reconstruye desde código
máquina.

Preguntas de control:

1. ¿Qué información perdió el binario respecto al código C?
2. ¿Qué información conserva todavía porque compilamos sin eliminar símbolos?
3. ¿Por qué las direcciones del objeto y del ejecutable no tienen que coincidir?

### Experimento 4: archivo frente a proceso

```bash
./programa
echo $?
```

El programa no imprime texto. El `42` observado pertenece al estado de salida
que el shell recibió cuando terminó el proceso.

Para observar el inicio con GDB:

```bash
gdb ./programa
```

Dentro de GDB:

```text
starti
info registers
x/10i $pc
break main
continue
disassemble main
quit
```

Preguntas de control:

1. Antes de alcanzar `main`, ¿ya existían stack, registros y memoria virtual?
2. ¿Quién creó el proceso y quién cargó o mapeó las partes del ELF?
3. ¿Qué trabajo debe ocurrir entre el punto de entrada y la llamada a `main`?

## Capas que no debemos mezclar

| Capa | Responsabilidad principal |
|---|---|
| Preprocesador | Expande directivas como `#include`, `#define` y compilación condicional. |
| Compilador | Analiza C y genera una representación de más bajo nivel. |
| Assembler | Convierte instrucciones de ensamblador en código máquina y metadatos relocatables. |
| Linker | Resuelve símbolos y relocations para producir el ELF final. |
| Kernel | Crea el proceso y prepara su espacio de ejecución. |
| Loader dinámico | Carga dependencias compartidas y resuelve linking dinámico. |
| Runtime de C | Prepara el entorno del lenguaje y termina llamando a `main`. |
| CPU | Ejecuta instrucciones; no conoce conceptos como “archivo C” o “función main”. |

Esta tabla es un mapa, no una explicación final. Durante la conversación iremos
defendiendo cada afirmación con observaciones.

## Puerta de dominio

El módulo queda superado cuando puedas explicar, sin consultar el documento:

1. Las transformaciones desde `programa.c` hasta ELF.
2. Qué información añade y pierde cada etapa.
3. Por qué el punto de entrada no tiene que ser `main`.
4. Qué convierte un archivo ejecutable en un proceso.
5. Qué responsabilidades pertenecen al kernel, al loader, al runtime y a la CPU.

No hace falta memorizar comandos. Sí hace falta poder elegir qué herramienta
usar para responder una pregunta concreta.

# C y sistemas de bajo nivel

## Propósito

Esta ruta usa C como una ventana hacia la máquina. Su meta no es aprender
recetas de sintaxis, sino poder razonar sobre lo que ocurre desde que existe un
archivo de código fuente hasta que una CPU ejecuta sus instrucciones.

Entorno de referencia:

- Linux
- Arquitectura x86-64
- GCC o Clang
- Binarios ELF
- GDB, binutils y, más adelante, Ghidra

## Cómo se avanza

No hay una duración fija por módulo. Cada uno tiene una **puerta de dominio**:
una pregunta, predicción o explicación que debe resolverse sin depender de que
la IA escriba la respuesta. La experiencia previa en Rust, Go, Python y
JavaScript permite avanzar deprisa por conceptos ya dominados.

La sintaxis de C se introduce cuando sirve para observar un fundamento. El
código es material experimental, no el objetivo final.

## Mapa

### 1. Del código fuente al proceso

Preprocesador, compilador, ensamblador, linker, ELF, loader, `_start`, runtime de
C y `main`.

**Puerta de dominio:** explicar qué transforma cada etapa, qué archivos produce
y quién pone realmente en marcha `main`.

[Comenzar el módulo](01-del-codigo-al-proceso/TEMARIO.md)

### 2. Bits, bytes y representación

Binario, hexadecimal, complemento a dos, endianness, tamaños, alineación y
representación de enteros y flotantes.

**Puerta de dominio:** reconstruir el significado de una región de memoria a
partir de sus bytes y del tipo con el que se interpreta.

### 3. Objetos, direcciones y punteros

Objetos de C, direcciones virtuales, tipos de puntero, desreferenciación,
aritmética de punteros, aliasing y `const`.

**Puerta de dominio:** predecir qué dirección calcula una expresión y distinguir
una dirección válida de un acceso válido.

### 4. Duración y regiones de memoria

Stack, heap, datos globales, almacenamiento estático, vida de los objetos,
`malloc`, `free` y ownership manual.

**Puerta de dominio:** detectar referencias colgantes, fugas y dobles
liberaciones razonando sobre la vida de cada objeto.

### 5. Arrays, strings y estructuras

Decaimiento de arrays, strings terminadas en cero, layouts de `struct`, padding,
unions y representación de datos.

**Puerta de dominio:** calcular manualmente offsets y tamaños, y explicar por qué
un array no es un puntero aunque a menudo se comporte como uno.

### 6. Funciones y ABI de System V AMD64

Stack frames, registros, convención de llamadas, paso de argumentos, valores de
retorno, prólogo y epílogo.

**Puerta de dominio:** seguir una llamada en ensamblador y reconstruir su firma
probable en C.

### 7. Compilación, linking y ELF

Símbolos, secciones, relocations, linking estático y dinámico, bibliotecas,
PLT/GOT y carga dinámica.

**Puerta de dominio:** diagnosticar un error de linking y localizar en un ELF
dónde se define o resuelve un símbolo.

### 8. C frente al sistema operativo

Procesos, memoria virtual, descriptores de archivo, syscalls, señales, `/proc`,
`mmap` y límites entre libc, kernel y hardware.

**Puerta de dominio:** describir el recorrido completo de una operación como
escribir bytes en un archivo.

### 9. Comportamiento indefinido y seguridad

Undefined behavior, overflows, out-of-bounds, use-after-free, stack smashing,
ASLR, NX, canarios y sanitizers.

**Puerta de dominio:** diferenciar un fallo observado de la regla del lenguaje
que fue violada y explicar por qué “funcionó en mi máquina” no es una garantía.

### 10. De binario a intención

GDB, `objdump`, `strace`, Ghidra, símbolos eliminados, reconocimiento de
patrones y reconstrucción gradual de lógica.

**Puerta de dominio:** analizar un binario pequeño sin código fuente y justificar
con evidencia qué hace.

## Resultado esperado

Al terminar, deberías poder:

- Explicar el recorrido de código fuente a instrucciones ejecutadas.
- Razonar sobre memoria y vida de objetos sin depender de ensayo y error.
- Leer ensamblador x86-64 básico generado desde C.
- Inspeccionar y depurar ejecutables ELF.
- Reconocer las causas fundamentales de errores de memoria.
- Empezar una sesión de ingeniería inversa con una metodología basada en
  hipótesis y evidencia.

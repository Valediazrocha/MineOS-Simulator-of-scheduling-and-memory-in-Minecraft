# MineOS · Misión Kernel

**Una página web para aprender los conceptos básicos de sistemas operativos, pensada para estudiantes de colegios STEAM de Bogotá.**

El sitio usa la idea de un mundo de bloques, inspirado en un servidor de Minecraft, para explicar cómo un computador decide quién usa la CPU, cómo guarda los programas en memoria y qué pasa cuando los procesos se bloquean entre sí. Tiene lecciones cortas con retos, laboratorios interactivos y una guía para docentes.

Proyecto final del curso de Sistemas Operativos, Universidad del Rosario (2026-2).

---

## Cómo abrirla

No necesita instalación ni conexión a internet.

1. Descarga `MineOS_estudiantes.html` (y `MineOS_docentes.html`, si eres docente).
2. Guárdalos en la **misma carpeta**: los enlaces entre las dos páginas dependen de eso.
3. Ábrelos con doble clic en un navegador actualizado (Chrome, Edge, Firefox o Safari).

El avance de cada estudiante se guarda solo en su navegador. No se pide nombre ni correo.

## Archivos

| Archivo | Para quién | Qué es |
|---|---|---|
| `MineOS_estudiantes.html` | Estudiantes | La página: mundo 3D, ruta de lecciones, laboratorios, manual y glosario |
| `MineOS_docentes.html` | Docentes | Guía para docentes: talleres de 90 minutos, conexiones STEAM, rúbrica y respuestas de los retos. La página de estudiantes no la enlaza |
| `Sugerencias de profesor` | Equipo | Indicaciones del profesor para esta entrega |

## Contenido

La ruta tiene **31 lecciones** en cinco mundos. Cada lección incluye teoría, un ejemplo resuelto y un reto.

| Mundo | Temas |
|---|---|
| 1. Aldea de Procesos | Qué es un SO, procesos y estados, llamadas al sistema e interrupciones, crear y terminar procesos, comunicación entre procesos, hilos |
| 2. Estación del Planificador | Métricas, FCFS, SJF y SRTF, Round Robin, prioridades y aging, colas multinivel, MFQ y CFS |
| 3. Cruce de Vagonetas | Interbloqueos: condiciones, grafo de asignación, prevención y evasión, algoritmo del banquero, detección y recuperación |
| 4. Mina de Memoria | Direcciones lógicas y físicas, asignación contigua, First/Best/Worst/Segregated Fit, paginación, TLB, tabla de páginas y segmentación |
| 5. Torre de Control | Resumen de las entregas del curso sobre el simulador UR-OS y el Mini UR-OS |

**Laboratorios:** planificadores de CPU, asignación contigua, paginación, segmentación, Mini UR-OS (CPU y memoria juntas) y «Servidor: antes y después».

**Fuera del alcance:** la memoria virtual (reemplazo de páginas y fallos de página) no se incluye, por indicación del profesor.

## Créditos y licencia

- Integrantes: *[completar]*
- La Torre de Control resume las entregas hechas sobre **UR-OS**, el simulador académico del profesor **Pedro Wightman**. El Mini UR-OS de la página es una versión simplificada escrita desde cero; no incluye código de UR-OS.
- El código, los textos y las texturas originales se dedican al dominio público con **CC0 1.0**. Componentes de terceros con su propia licencia: **three.js** (MIT) y la fuente **VT323** (SIL OFL 1.1).
- Referencias: Silberschatz, Galvin y Gagne, *Operating System Concepts*; Arpaci-Dusseau, *Operating Systems: Three Easy Pieces*.

> Proyecto educativo independiente. NO ES UN PRODUCTO OFICIAL DE MINECRAFT. NO ESTÁ APROBADO NI ASOCIADO CON MOJANG NI MICROSOFT.

---

### English

MineOS is a bilingual (Spanish/English) educational website that teaches operating-system basics to secondary-school students through a block-world theme. Open `MineOS_estudiantes.html` in a browser; teachers can open `MineOS_docentes.html` from the same folder. No installation or internet connection is needed. Virtual memory is out of scope. Licence: CC0 1.0 (three.js: MIT; VT323 font: SIL OFL 1.1).

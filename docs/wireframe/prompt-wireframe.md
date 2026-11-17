# Prompt para crear wireframes de otros software

Prompt reutilizable para obtener el mismo tipo de wireframe que el de TheirView: un tablero estático de pantallas, en escala de grises, sin diseño visual ni código. Este prompt incluye las pantallas genéricas (carga, selección de perfil, menú en fila de botones cuadrados y ajustes) y la división entre el diseñador y el equipo de programación, que se define junto con el usuario.

**Uso:** copiar todo el bloque y pegar el contexto del software en la sección final, CONTEXTO DEL SOFTWARE.

```text
OBJETIVO
Crear el wireframe del software descrito al final, en la seccion CONTEXTO DEL SOFTWARE. Debe ser un tablero estatico de pantallas, en un unico archivo HTML: docs/wireframe/wireframe-<nombre-del-software>.html. Es solo la distribucion de la interfaz: no es un prototipo, no es diseno visual y no es codigo.

Si en el proyecto existe un wireframe de referencia (por ejemplo docs/wireframe/wireframe-tracepath-example.html o docs/wireframe/wireframe-theirview.html), leelo antes de empezar y sigue su disposicion. No lo modifiques.

==================================================
FASE 1 · ENTENDER Y PREGUNTAR (antes de escribir el archivo)
==================================================
Lee el contexto completo y responde en el chat, sin crear todavia el archivo:

1. Tu resumen en 3 a 5 lineas de que hace el software y quien lo usa.
2. Los bloques y las pantallas que propones, en orden de flujo. Incluye las pantallas genericas de la seccion PANTALLAS GENERICAS y las propias del software.
3. Las acciones del menu principal: entre 3 y 5 botones, y si corresponde un boton Continuar.
4. Los ajustes adicionales propios de este software, si hacen falta, con la razon de cada uno. Si no hace falta ninguno, dilo y explica por que. No son obligatorios.
5. La division entre el diseñador y el equipo de programacion. Este punto se razona con el usuario:
   - Por defecto, el diseñador hace toda la interfaz y el logo.
   - El equipo de programacion construye todo lo que no es interfaz: modelos 3D, escenas, tableros de juego, lienzos de dibujo, contenido generado, sonidos generados, y ademas hace que todo funcione.
   - Propone que le toca a cada uno en ESTE software, explica tu razonamiento y señala los casos dudosos. Por ejemplo: quien define los colores de las piezas de un juego, o quien dibuja el contenido de los niveles.
   - Haz las preguntas concretas que necesites.
6. Cualquier duda sobre el contexto.

Espera la respuesta del usuario. Ajusta la propuesta con lo que diga y recien entonces pasa a la Fase 2.

==================================================
FASE 2 · CONSTRUIR EL WIREFRAME
==================================================

ESTRUCTURA DEL TABLERO (de arriba a abajo)
1. Encabezado:
   - Una etiqueta pequeña con el area o tema del software.
   - El nombre del software.
   - Una linea que diga de que trata.
   - Si corresponde, una caja de aviso de alcance (por ejemplo: no es una prueba clinica, los datos quedan en este equipo).
2. Bloque sin numero, Antes de empezar · Quién diseña qué, con tres tarjetas:
   - El diseñador, con el subtitulo «toda la interfaz y el logo» y una lista numerada.
   - Equipo de programación, con dos grupos titulados: uno con lo que construye y no es interfaz (por ejemplo «Lo 3D (la escena)») y otro titulado «Hacer que todo funcione». El segundo grupo incluye: armar en la app las pantallas del diseñador y conectar la navegacion; la logica propia del software; el contenido (niveles, rondas, ejercicios); los perfiles y el guardado en la PC, incluido retomar donde se quedo; calcular y mostrar resultados; que los ajustes funcionen; los efectos de sonido; los textos de la interfaz; probar todo y preparar la app para instalarla en Windows y usarla sin internet.
   - Punto de contacto: como se juntan las dos partes (por ejemplo, mostrarle al diseñador una captura de la escena antes de que cierre los colores).
   - Una muestra de las dos marcas: una caja con borde normal rotulada INTERFAZ · DISEÑADOR y una caja rayada con borde discontinuo rotulada ZONA <nombre> · EQUIPO DE PROGRAMACIÓN.
   Si las tarjetas tienen alturas muy distintas, pon las dos primeras lado a lado y el punto de contacto debajo, a todo el ancho.
3. Bloques numerados (1 · ACCESO Y PERFIL, 2 · CONFIGURACIÓN Y AVANCE, etc.). Cada bloque tiene:
   - Un titulo de seccion con una linea gruesa debajo.
   - Una tira de flujo: cajas con el nombre de cada paso, unidas con flechas.
   - Una grilla de tarjetas de pantalla. Cada tarjeta tiene:
     a. Cabecera con el nombre de la pantalla en mayusculas pequeñas y tres circulos a la derecha.
     b. Lista breve de elementos y comportamientos.
     c. Esquema de la pantalla con sus componentes.
     d. Nota final con Propósito: y, segun aplique, Regla:, Registra: o Muestra:.
   Los estados importantes van en tarjetas propias: dialogos, primera vez, sesion a medias, casos de resultado.
4. Al final, una leyenda Cómo funciona (resumen del flujo completo): un parrafo que recorre todo el flujo y explica las marcas (que significa la zona rayada).

LENGUAJE VISUAL (no negociable)
- Solo escala de grises. Todo color del archivo es un gris: en cada valor, los canales R, G y B difieren en 8 o menos.
- Tipografia del sistema: sans-serif para el contenido y monoespaciada para rotulos, etiquetas y tablas. Sin fuentes externas ni de marca.
- Fondo del lienzo gris muy claro (por ejemplo #F7F6F3).
- Componentes con relleno blanco y borde de 1.5px gris oscuro (por ejemplo #37352F).
- Regiones de layout con linea continua fina de 1px en gris medio.
- Todo borde de componente contrasta 3:1 o mas con su fondo, y el texto cumple contraste AA.
- Rotulo de componente: texto pequeño, monoespaciado y en mayusculas, apoyado sobre el borde superior izquierdo, que dice que es. Ejemplos: BOTON · PRIMARIO, BOTON · SECUNDARIO, CAMPO · NOMBRE DE PERFIL, LISTA · PERFILES LOCALES, ESCALA 1-5, TARJETA · METRICA, TABLA · DETALLE, DIALOGO · CONFIRMACION, REGION · ENCABEZADO.
- Los rotulos nunca se superponen ni se cortan. Si un componente es angosto, rotula la region que lo contiene en lugar de cada pieza.
- Botones:
  - Primario: relleno oscuro y texto blanco.
  - Secundario: relleno blanco y borde oscuro.
  - Altura minima 44px.
  - Las teclas de atajo se muestran como teclas pequeñas: Enter, Espacio, Escape.
- La linea discontinua y el relleno rayado se usan SOLO para la zona que construye el equipo de programacion (escena 3D, tablero, lienzo). Esa zona lleva el rotulo ZONA <nombre> · EQUIPO DE PROGRAMACIÓN. Ningun otro elemento usa linea discontinua, punteada ni rayado: avisos y marcadores van con relleno gris liso.
- Lo que va encima de esa zona (consignas, cronometro, botones, mensajes) es interfaz del diseñador y lleva borde normal.
- Los estados se distinguen sin color: con relleno oscuro, doble linea, fondo gris o blanco, y siempre con su texto.
- Si un color es parte del contenido (por ejemplo el color de una pieza), se indica con una etiqueta de texto: ROJO, AZUL.
- Iconos: cajas de marcador con la palabra ICONO. Logo: caja LOGO · PENDIENTE con el nombre en texto. Sin emojis.
- Los datos de ejemplo son ficticios y van marcados como simulados.
- Ancho de referencia: 1280 a 1920 px. Grilla de 2 o 3 tarjetas por fila, sin scroll horizontal.

PROHIBIDO EN EL WIREFRAME
- Decisiones de diseño visual: paleta, fuentes de marca, estilo final. Eso lo decide el diseñador despues.
- Logica de prototipo: JavaScript, etiqueta script, temporizadores, puntajes calculados. El cronometro se muestra como un valor fijo.
- Referencias a codigo en el texto visible: nombres de funciones o de campos, nombres de archivos, rutas de carpetas, formatos tecnicos, atributos, identificadores en mayusculas con guion bajo y codigos de color como #FFFFFF. Todo se escribe en lenguaje llano: se guarda en esta PC, por perfil, sin internet. Blanco y negro puros.
- CDN, recursos externos, almacenamiento del navegador, exportaciones que el contexto no pida.

PANTALLAS GENERICAS (van en todos los software; adapta los textos a cada uno)
1. Pantalla de carga:
   - El logo centrado y un texto breve de carga.
   - Duracion de unos 2.5 s; pasa sola a Selección de perfil.
   - Con reducir movimiento activado, aparece sin animacion.
2. Selección de perfil:
   - Lista de perfiles guardados en esta PC: nombre, ultima actividad y boton eliminar con icono.
   - Un unico campo para buscar o crear un perfil («Ingresar o crear perfil»): si el nombre coincide, ingresa; si no, crea uno nuevo.
   - Aqui SI se usa lista.
3. Diálogo eliminar perfil:
   - Pide confirmacion, con Cancelar como secundario y Eliminar perfil como primario.
   - Advierte que se borra todo su historial y se cierra con Escape.
4. Menú principal (muy importante, cuida que no sea una lista ni quede soso):
   - Arriba a la izquierda, un boton Cambiar usuario con el nombre del perfil activo.
   - Arriba al centro, el nombre o logo del software como referencia visual.
   - Debajo, UNA fila horizontal centrada de botones CUADRADOS compactos:
     - Todos del mismo tamaño, entre 72 y 104 px de lado.
     - Cada uno con un marcador ICONO dentro y el nombre corto debajo.
     - Sin descripciones largas; un solo clic, sin submenus.
   - Si hay algo para retomar, Continuar va primero, destacado con relleno oscuro, con una linea secundaria corta debajo (por ejemplo «Nivel 3 de 5»).
   - Sin nada para retomar, Continuar no aparece. Agrega una tarjeta o nota para esa variante.
   - Los botones siempre en una sola fila, aun en tarjetas angostas: achica el cuadrado antes que partir la fila.
   - Un unico rotulo de region: REGION · MENU EN FILA DE ICONOS · BOTONES CUADRADOS.
   - Acciones tipicas: Iniciar o Jugar, Ver avance o Récords, Ajustes, y las propias del software.
5. Guía rápida (solo la primera vez, o desde Ajustes):
   - De 2 a 4 pasos con marcador de ilustracion.
   - Botones Anterior, Siguiente y Omitir.
6. Ajustes (aqui SI se usa lista de filas). Ajustes clasicos:
   - Volumen, con deslizador.
   - Silenciar, con interruptor de sonido.
   - Alto contraste.
   - Tamaño de texto: Normal o Grande.
   - Reducir movimiento.
   - Ver guía de nuevo.
   Ademas:
   - Se guarda al instante, por perfil, sin boton guardar.
   - Boton Volver al menú.
   - Una tarjeta de vista previa con alto contraste y texto grande activados.
   - Si el software lo necesita, un grupo Ajustes propios de <nombre>, separado de los clasicos. Puede no existir.
7. Ver avance o Récords: historial del perfil activo con un resumen y el detalle. Lo que no se jugo se muestra con guion.
8. Retomar:
   - Si la app se cerro a mitad de algo, el avance se guardo.
   - Al reabrir, se pasa por Carga, Perfil y Menú, y se muestra Continuar.
   - Describe desde donde se retoma.
Despues vienen los bloques propios del software (el flujo central), en el orden de uso, con un bloque final de cierre o resultados si corresponde.

GUARDADO DE DATOS (contexto tecnico, no va al texto visible del wireframe)
- Los datos de cada usuario se guardan solo en la PC, en la carpeta %LOCALAPPDATA%\<NombreDelSoftware>, con una subcarpeta por perfil para sus ajustes y su historial.
- En el wireframe solo se dice en lenguaje llano: se guarda en esta PC, por perfil.
- Si el proyecto tiene una especificacion tecnica (por ejemplo docs/SPECIFICATION.md), el detalle tecnico va alli, nunca en el wireframe.

==================================================
FASE 3 · VERIFICAR (antes de entregar)
==================================================
- Busca en el archivo, sin distinguir mayusculas: script, http, cdn, @import, localStorage, sessionStorage, indexedDB y console. Debe dar 0 resultados.
- Extrae con un script externo todos los colores hex y rgb y confirma que todos son grises (diferencia entre canales de 8 o menos).
- Lista todas las apariciones de dashed, dotted y repeating-linear-gradient. Cada una debe pertenecer a la zona del equipo de programacion o a lineas dentro de su dibujo.
- Busca en el texto visible: rutas, extensiones de archivo, codigos de color e identificadores con guion bajo. Debe dar 0.
- Busca emojis (caracteres entre U+1F000 y U+1FFFF): 0.
- Abre el archivo en un navegador a 1280 y a 1920 px de ancho y revisa:
  - ningun rotulo se superpone ni se corta;
  - el menu queda en una sola fila;
  - los bordes de cada boton, campo y tarjeta se ven claramente.
- Si alguna verificacion falla, corrigela antes de entregar.

==================================================
FASE 4 · ENTREGA
==================================================
Responde con:
- Archivo creado.
- Bloques y pantallas incluidos.
- Resultado de cada verificacion.
- Supuestos tomados sin consultar.
- Preguntas abiertas para el usuario.
Despues espera sus comentarios para iterar. Mantén siempre estas reglas en cada cambio.

==================================================
CONTEXTO DEL SOFTWARE
==================================================
Nombre:
De qué trata:
Quién lo usa y en qué situación:
Acciones principales del usuario:
Contenido que no es interfaz (3D, tablero, lienzo, niveles, sonidos, etc.):
Qué datos se guardan de cada usuario:
Restricciones (sin internet, Windows, resolución, accesibilidad, etc.):
Indicaciones y todo lo conocido del software:
<pega aquí el contexto>
```

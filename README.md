# Tarea Semana 9 · Manipulación de archivos CSV en C++

**Curso:** Algoritmos · Universidad Mariano Gálvez
**Estudiante:** ______________________  **Carné:** ____________  **Sección:** ______

## 1. Descripción del problema
Una tienda guarda su catálogo en `datos/productos.csv`. El programa lee los registros, detecta datos inválidos sin detenerse, calcula el valor total del inventario (precio × existencia de los registros válidos), permite buscar un producto por código y genera `reportes/resumen.txt`. El CSV solo se abre para lectura (`ifstream`), por lo que nunca se sobrescribe.

## 2. Estructura del CSV
Encabezado: `codigo,nombre,precio,existencia`. Un producto por línea, 4 campos separados por coma.
Limitación asumida: los nombres no contienen comas.

| Campo | Tipo | Regla |
|---|---|---|
| codigo | texto | No vacío |
| nombre | texto | No vacío |
| precio | decimal | Numérico y > 0 |
| existencia | entero | Entero y >= 0 |

## 3. Estructura del proyecto
```
semana9/
├── main.cpp
├── main.txt              (copia del código en .txt para Canvas)
├── datos/productos.csv
├── reportes/resumen.txt  (generado por el programa)
├── evidencias/
└── README.md
```

## 4. Compilación y ejecución
Desde la carpeta `semana9/`:
```
g++ -std=c++17 -Wall -Wextra -o inventario main.cpp
./inventario                      # usa datos/productos.csv
./inventario datos/otra_ruta.csv  # ruta alterna (útil para probar error de archivo)
```
En Windows (MinGW): `inventario.exe`. El programa pide un código por teclado (ej. `P003`).

## 5. Diseño antes de programar

**Entradas:** archivo `datos/productos.csv` (ruta relativa, o argumento opcional) y un código escrito por el usuario.
**Proceso:** abrir archivo → omitir encabezado → leer línea por línea → separar 4 campos → validar → acumular válidos/inválidos y valor del inventario → buscar código → escribir reporte.
**Salidas:** mensajes en pantalla (conteos, filas inválidas con motivo, resultado de la búsqueda) y el archivo `reportes/resumen.txt`.

**Pseudocódigo**
```
INICIO
  ruta <- "datos/productos.csv"
  abrir archivo para lectura
  SI no se abrió: mostrar error; terminar con código 1
  leer y descartar encabezado
  valorTotal <- 0
  MIENTRAS haya líneas:
      SI la línea está en blanco: continuar
      separar en codigo, nombre, precioTxt, existenciaTxt
      SI faltan o sobran campos: registrar inválido; continuar
      SI codigo vacío o nombre vacío: registrar inválido; continuar
      intentar convertir precioTxt a número
          SI falla o precio <= 0: registrar inválido; continuar
      intentar convertir existenciaTxt a entero
          SI falla o existencia < 0: registrar inválido; continuar
      guardar producto en lista de válidos
      valorTotal <- valorTotal + precio * existencia
  cerrar archivo
  mostrar conteos y valorTotal
  pedir código al usuario
  buscar en válidos; si no, buscar en inválidos; si no, "no existe"
  abrir reportes/resumen.txt
  SI no se pudo: mostrar error; terminar con código 2
  escribir válidos, inválidos, valorTotal y resultado de búsqueda
  cerrar archivo
FIN
```

**Cómo distingo válidos de inválidos:** cada fila pasa por `validarFila()`. Se rechaza si no tiene exactamente 4 campos, si código o nombre están vacíos, si el precio no es un número completo o es <= 0, o si la existencia no es un entero completo o es < 0. Se usa `stod`/`stoi` con el parámetro `pos` para exigir que **todo** el texto se convierta (así `12abc` o `8.5` como existencia se rechazan) y `try/catch` para atrapar `invalid_argument` y `out_of_range`. Una fila inválida solo se registra con su motivo y el ciclo continúa.

**Si el archivo no existe o no se abre:** se muestra un mensaje claro con la ruta intentada y el programa termina de forma controlada con código 1 (sin cerrarse de forma inesperada y sin crear reporte).

## 6. Decisiones de validación
- Las líneas completamente en blanco se ignoran (no cuentan como inválidas).
- Se elimina el `\r` final de cada línea (por si el CSV se editó en Windows).
- La búsqueda es exacta y solo considera registros válidos (una fila inválida se reporta como "no existe").
- La carpeta `reportes/` debe existir; si no, el programa muestra un error y termina con código 2.
- Un registro inválido nunca entra al cálculo del inventario.

## 7. Casos de prueba (resultados reales)
| Caso | Entrada / situación | Esperado | Obtenido | ¿Cumple? | Evidencia |
|---|---|---|---|---|---|
| 1 | Archivo base completo | Procesa válidos e inválidos | 5 válidos, 3 inválidos, total Q7179.00 | Sí | `evidencias/caso1_2_ejecucion_correcta_P003.txt` |
| 2 | Buscar P003 | Muestra Monitor 24 | ENCONTRADO: Monitor 24, Q1150.00, existencia 3 | Sí | `evidencias/caso1_2_ejecucion_correcta_P003.txt` |
| 3 | Buscar P999 | Indica que no existe | "El codigo NO existe en el archivo." | Sí | `evidencias/caso3_codigo_inexistente.txt` |
| 4 | Precio = abc (fila P007) | Cuenta inválido y continúa | "Linea 8 invalida: precio o existencia no numericos"; sigue y encuentra P001 | Sí | `evidencias/caso4_precio_invalido.txt` |
| 5 | Ruta incorrecta | Error y terminación controlada | Mensaje de error, código de salida 1 | Sí | `evidencias/caso5_ruta_incorrecta.txt` |

**Cálculo manual del total:** 125.50×8 = 1004.00; 85.00×15 = 1275.00; 1150.00×3 = 3450.00; 72.50×20 = 1450.00; 55.00×0 = 0.00. Suma = **Q7179.00** (coincide con el programa).

> Las capturas de pantalla (imágenes) deben tomarse al ejecutar el programa en su equipo y guardarse en `evidencias/`. Los `.txt` incluidos muestran la salida esperada.

## 8. Uso responsable de IA
Consulté a Claude (Anthropic) para generar una primera versión del programa, el pseudocódigo y este README. Estudié el código y puedo explicar `ifstream`, `getline`, `stringstream`, `stod`/`stoi` y `try/catch`. Partes que revisé/modifiqué: ______________________________ (completar con sus cambios reales).

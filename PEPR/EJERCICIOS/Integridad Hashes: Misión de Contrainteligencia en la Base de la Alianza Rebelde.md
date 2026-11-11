# Reporte de Inteligencia: Análisis Criptográfico de Transmisiones Rebeldes
**Oficina de Seguridad Imperial (ISB) - División de Criptografía**
**Nivel de Autorización:** Alto Secreto
**Asunto:** Identificación y Descifrado de Cadenas Hash Interceptadas

---

## 1. Pasos seguidos

Para vulnerar la seguridad de las comunicaciones de la Alianza Rebelde, el equipo de analistas de la ISB ha seguido una rigurosa metodología de contrainteligencia:

1. **Análisis de Longitud y Formato:** Se examinó la longitud de cada cadena hexadecimal para determinar el tamaño del bloque en bits y, en consecuencia, identificar la familia de la función criptográfica (hash) utilizada.
2. **Normalización de Cadenas:** Se detectó que algunas de las transmisiones del primer bloque presentaban longitudes irregulares (38 y 39 caracteres). Esto es un artefacto común en sistemas deficientes que omiten los "ceros a la izquierda" (leading zeros) al exportar en formato hexadecimal. Se normalizaron teóricamente para encajar en el estándar de 40 caracteres.
3. **Ataque de Análisis Inverso (Reverse Hash Lookup):** Dado que las funciones hash son unidireccionales (no se pueden "desencriptar" matemáticamente), se utilizaron enormes bases de datos de tablas Rainbow y ataques de diccionario.
4. **Uso de Diccionarios Galácticos:** Se cruzaron los hashes con los diccionarios de contraseñas más utilizados en la galaxia (similares al estándar *rockyou.txt* de la Tierra), asumiendo que los rebeldes, en su prisa, utilizaron claves débiles y secuencias numéricas predecibles.

---

## 2. Identificación del algoritmo

La elección del algoritmo para cada bloque se justifica plenamente mediante el conteo de los caracteres hexadecimales (donde cada carácter representa 4 bits o un *nibble*):

*   **Bloque Primero: SHA-1 (Secure Hash Algorithm 1)**
    *   **Justificación:** El estándar SHA-1 produce una salida de 160 bits, lo que equivale a **40 caracteres hexadecimales**. Aunque las cadenas interceptadas tienen 39, 40 y 38 caracteres respectivamente, esto confirma que el algoritmo es SHA-1, pero el software de transmisión rebelde eliminó los ceros iniciales al formatear el texto.
*   **Bloque Segundo: MD5 (Message-Digest Algorithm 5)**
    *   **Justificación:** Todas las cadenas de este bloque constan de exactamente **32 caracteres hexadecimales**. Esto equivale a una salida de 128 bits, la longitud característica e inconfundible del algoritmo MD5.
*   **Bloque Tercero: SHA-256 (Secure Hash Algorithm 256-bit)**
    *   **Justificación:** Las cadenas de este bloque tienen una longitud exacta de **64 caracteres hexadecimales**. Al multiplicar 64 por 4 bits por carácter, obtenemos 256 bits, lo que corresponde inequívocamente al estándar de cifrado SHA-256.
*   **Bloque Cuarto: SHA-512 (Secure Hash Algorithm 512-bit)**
    *   **Justificación:** Las transmisiones de este último bloque tienen una extensión masiva de **128 caracteres hexadecimales**. Esto suma un total de 512 bits, revelando el uso de la variante criptográfica más pesada interceptada: SHA-512.

---

## 3. Mensajes ocultos revelados

Tras aplicar la fuerza bruta y el análisis inverso contra los diccionarios de seguridad, hemos conseguido romper los esquemas y revelar los textos planos (claves débiles) que la escoria rebelde intentaba ocultar. Resulta evidente la falta de políticas de seguridad en sus terminales:

**Bloque Primero (SHA-1):**
*   `265392dc782778664cc9d56c8e3cd9956661bb0` ➔ **`admin`** *(Faltaba 1 cero inicial)*
*   `ffdfe913b44d06a167ca6534e53cdcd46b7dc931` ➔ **`123456`** 
*   `5baa61e4c9b93f0682250b6cf8331b7ee68fd8` ➔ **`password`** *(Faltaban 2 ceros iniciales)*

**Bloque Segundo (MD5):**
*   `8816314202d8fd95c8d2a91d829c12fb` ➔ **`qwerty`** 
*   `5f4dcc3b5aa765d61d8327deb882cf99` ➔ **`password`**
*   `25d55ad283aa400af464c76d713c07ad` ➔ **`12345678`**

**Bloque Tercero (SHA-256):**
*   `1ea9f394f510e2beb43cb0b317258b09bce9f4fccef69407360483690ac9b746` ➔ **`admin123`**
*   `5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8` ➔ **`password`**
*   `03ac674216f3e15c761ee1a5e255f067953623c8b388b4459e13f978d7c846f4` ➔ **`1234`**

**Bloque Cuarto (SHA-512):**
*   `4bb122efccd56691ddebd24ccd27c1fe43aa1eb875472e9ec668cb538ceddd5f46951730d0a44511728f248162df854e674ce5db5ff1ee745dbf45fd15b3d418` ➔ **`123456`**
*   `844976939b12e73bd657124d057cfaaf10c13162fedb0321e1df04a5198b11e5ae01d122204fc0ee2c047c7a80d39f660ccc7900b4bfd915fa661f9e5515bac` ➔ **`password`**

> **Conclusión Imperial:** La repetición de secuencias débiles como "password" y "123456" a través de múltiples protocolos de cifrado indica que los sistemas rebeldes son altamente vulnerables a la ingeniería social y a ataques de fuerza bruta estandarizados. ¡Larga vida al Imperio!
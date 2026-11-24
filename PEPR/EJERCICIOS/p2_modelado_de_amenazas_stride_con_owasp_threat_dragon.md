# Módulo 03 · Construcción de software seguro · Sesión 1 - Ejercicio 2

**P2 · Modelado de amenazas STRIDE con OWASP Threat Dragon**

---

## 1. Puesta en marcha: BreakingLab, código fuente y Threat Dragon

### Enunciado y entorno de laboratorio

**BreakingLab** (`https://github.com/TiiZss/BreakingLab`) es un script Bash que despliega laboratorios vulnerables mediante contenedores Docker en Linux (Kali/Ubuntu) o Windows con WSL2. Para realizar el modelado de amenazas y contrastarlo posteriormente en la aplicación real:

1. **Despliegue de OWASP Juice Shop (`20.3.0`):**

```bash
git clone https://github.com/TiiZss/BreakingLab.git
cd BreakingLab
chmod +x breakinglab.sh
./breakinglab.sh start juiceshop      # Privado: http://127.10.0.1:3000
./breakinglab.sh status               # Comprueba que aparece en verde
# Al terminar la práctica: ./breakinglab.sh stop juiceshop
```

2. **Obtención del modelo oficial (`threat-model.json`):**

```bash
git clone --depth 1 --branch v20.3.0 https://github.com/juice-shop/juice-shop.git
```

3. **Carga en OWASP Threat Dragon 2.6.2:**
   * Abrimos la aplicación de escritorio **OWASP Threat Dragon v2.6.2** y seleccionamos *«Abrir un modelo de amenazas existente»* (*Open an existing threat model*) apuntando a `juice-shop/threat-model.json`.
   * Al tratarse de un modelo definido originalmente en el esquema v1 de Threat Dragon, la versión 2.6.2 muestra una notificación informativa de esquema y lo adapta al entorno v2 para poder editar el diagrama **«High Level Data Flow»**.

> **Aviso importante:** Juice Shop y el resto de máquinas de BreakingLab son vulnerables a propósito. No deben exponerse a Internet, no deben parchearse durante la práctica y nunca deben utilizarse estas técnicas contra sistemas ajenos sin autorización expresa.

---

## 2. Análisis del Diagrama de Flujo de Datos (DFD) y Fronteras de Confianza

### Enunciado (Pasos 1 a 4)

1. Abrir el diagrama **«High Level Data Flow»** en Threat Dragon 2.6.2.
2. Identificar todos sus elementos: actores externos, procesos, almacenes de datos y flujos de información.
3. Localizar las **fronteras de confianza** (*Trust Boundaries*) que atraviesan los flujos, ya que constituyen los puntos críticos donde cambian los privilegios y donde deben concentrarse las amenazas STRIDE.

[MODELO](./IMG/threadmodel.png)

### Explicación e inventario del DFD «High Level Data Flow»

Al inspeccionar el diagrama de arquitectura de OWASP Juice Shop, el sistema se divide en cuatro tipos de componentes gráficos:

| Categoría DFD | Elementos exactos en el diagrama | Descripción y función en el sistema |
| :--- | :--- | :--- |
| **Actores externos** *(Rectángulos)* | `B2C Customer (Browser)`, `B2B Customer (Browser)`, `Admin (Browser)`, `Accounting (Browser)`, `Google` | Entidades externas que interactúan con la aplicación: clientes particulares, clientes corporativos, administradores, personal de contabilidad y el proveedor externo de autenticación OAuth2 (`Google`). |
| **Procesos** *(Círculos)* | `Angular Frontend`, `Application Server`, `B2B API` | Unidades de procesamiento: la aplicación web SPA ejecutada en el navegador del cliente (`Angular Frontend`), el backend principal en Node.js/Express (`Application Server`) y la interfaz de integración empresarial (`B2B API`). |
| **Almacenes de datos** *(Líneas paralelas)* | `SQLite Database`, `MarsDB NoSQL DB`, `Local File System` | Repositorios persistentes: base de datos relacional principal (`SQLite Database` para usuarios, productos y cestas), base de datos documental (`MarsDB NoSQL DB` para pedidos y reseñas) y sistema de ficheros del servidor (`Local File System` para facturas, configuración, logs y `/ftp`). |
| **Flujos de datos** *(Flechas)* | `OAuth2`, `API Requests`, `API Responses`, `Invoices`, `Orders`, `Reviews`, `Configuration`, `Logging`, `all other data`, `User Management`, `Product Inventory` | Canales por los que viajan los datos entre actores, procesos y almacenes (peticiones REST/JSON, autenticación delegada, gestión administrativa y escritura en disco/BD). |

### Fronteras de confianza (*Trust Boundaries*) identificadas

En el diagrama se distinguen **5 fronteras de confianza** (representadas mediante líneas curvas discontinuas) que delimitan zonas con distinto nivel de control y privilegio:

1. **Frontera `Angular Frontend` $\leftrightarrow$ `Application Server`:** Separa el código ejecutado en el navegador del usuario (entorno no confiable donde un atacante puede interceptar o modificar peticiones HTTP) del servidor backend.
2. **Frontera `B2C / B2B Customer (Browser)` $\leftrightarrow$ `Angular Frontend` / `B2B API`:** Delimita la entrada de datos desde los equipos de los clientes hacia las interfaces públicas de la tienda y de la API B2B.
3. **Frontera `Admin / Accounting (Browser)` $\leftrightarrow$ `Angular Frontend`:** Separa el acceso de los perfiles con privilegios elevados (gestión de usuarios e inventario) frente al resto de usuarios.
4. **Frontera `Angular Frontend` $\leftrightarrow$ `Google`:** Cruza hacia un proveedor de identidad externo en Internet mediante el flujo `OAuth2`.
5. **Frontera `Application Server` $\leftrightarrow$ Almacenes (`Local File System`, `SQLite Database`):** Separa la lógica de negocio de la persistencia en disco y base de datos.

### Evidencia visual 1: Diagrama DFD con amenazas STRIDE en Threat Dragon

![Diagrama DFD de OWASP Juice Shop con amenazas STRIDE](IMG/threadmodelterminado.png)


---

## 3. Modelado de amenazas STRIDE y contramedidas

### Enunciado (Pasos 5 y 6)

Aplicar la metodología **STRIDE** sobre los elementos y flujos del DFD en Threat Dragon (*New threat*), documentando un **mínimo de 8 amenazas** (cubriendo obligatoriamente las 6 letras: **S, T, R, I, D, E**), su prioridad (`High`, `Medium`, `Low`), estado (`Open`) y su contramedida específica (*Mitigations*):

| Letra | Amenaza | Propiedad de seguridad vulnerada | Elementos típicos afectados |
| :---: | :--- | :--- | :--- |
| **S** | **Spoofing** *(Suplantación)* | Autenticación | Actores, procesos, flujos de sesión |
| **T** | **Tampering** *(Manipulación)* | Integridad | Flujos de datos, almacenes, procesos |
| **R** | **Repudiation** *(Repudio)* | No repudio / Trazabilidad | Procesos, flujos de registro (`Logging`) |
| **I** | **Information Disclosure** *(Revelación de info.)* | Confidencialidad | Almacenes, flujos, procesos |
| **D** | **Denial of Service** *(Denegación de servicio)* | Disponibilidad | Procesos, flujos, almacenes |
| **E** | **Elevation of Privilege** *(Elevación de privilegios)* | Autorización | Procesos |

### Matriz detallada de amenazas STRIDE modeladas (9 amenazas)

A continuación se detallan las **9 amenazas** incorporadas al modelo, explicando el escenario de ataque, el elemento exacto afectado en el DFD y la mitigación técnica propuesta:

| ID | STRIDE | Elemento del DFD | Título y explicación de la amenaza | Prioridad / Estado | Contramedida (*Mitigations*) |
| :-: | :---: | :--- | :--- | :---: | :--- |
| **01** | **S** *(Spoofing)* | **Proceso:** `Application Server` | **Suplantación de administrador mediante Inyección SQL en Login:** El servidor construye la consulta SQL de autenticación concatenando directamente el campo `email` recibido del cliente. Un atacante introduce `' OR 1=1--` y logra iniciar sesión como `admin@juice-sh.op` sin conocer la contraseña. | **High**<br>*Open* | Reemplazar la concatenación de cadenas SQL por **consultas parametrizadas** (sentencias preparadas) e implementar autenticación multifactor (MFA) para cuentas privilegiadas. |
| **02** | **T** *(Tampering)* | **Flujo:** `API Requests` *(Angular Frontend $\rightarrow$ Application Server)* | **Manipulación de cantidades y precios en la cesta de compra:** Al cruzar la frontera de confianza cliente-servidor, un atacante intercepta la petición `POST /api/BasketItems` y envía cantidades negativas (`"quantity": -5`), alterando la integridad del cálculo del pedido. | **High**<br>*Open* | Validar en el backend que las cantidades sean enteros positivos ($q \ge 1$) y calcular los precios unitarios y totales exclusivamente desde el catálogo del servidor. |
| **03** | **T** *(Tampering)* | **Almacén:** `MarsDB NoSQL DB` / **Flujo:** `Reviews` | **Manipulación de contenido mediante XSS almacenado en reseñas:** Un usuario malicioso publica una opinión de producto con código JavaScript incrustado que se guarda en `MarsDB NoSQL DB` y se ejecuta en el navegador de otros clientes al consultar el producto. | **Medium**<br>*Open* | Aplicar sanitización estricta de entradas en la API y codificación de salida (*output encoding*) contextual antes de renderizar reseñas en Angular. |
| **04** | **R** *(Repudio)* | **Flujo:** `Logging` *(Application Server $\rightarrow$ Local File System)* | **Repudio de operaciones críticas por registros sin integridad:** Un cliente o administrador puede negar haber realizado un pedido o modificado el inventario debido a que los registros en `Local File System` carecen de firma, usuario autenticado y protección frente a borrado. | **Medium**<br>*Open* | Registrar cada evento crítico con identificador de usuario, IP origen y marca de tiempo sincronizada (NTP), enviando los logs a un repositorio centralizado de solo escritura (*append-only*). |
| **05** | **I** *(Info. Disclosure)* | **Almacén:** `Local File System` | **Exposición pública de ficheros confidenciales en `/ftp` y logs:** El servidor web publica directorios internos del sistema de ficheros (`/ftp`, `/encryptionkeys`, `/support/logs`) con listado de archivos habilitado, revelando copias de seguridad, claves y registros internos. | **High**<br>*Open* | Deshabilitar el listado automático de directorios (*directory listing*), mover los archivos sensibles fuera de la raíz web pública y exigir autorización por recurso. |
| **06** | **I** *(Info. Disclosure)* | **Almacén:** `SQLite Database` / **Flujo:** `all other data` | **Volcado masivo de usuarios y contraseñas por SQLi en buscador:** Mediante una inyección `UNION SELECT` en el buscador de productos, un atacante extrae la tabla `Users` de `SQLite Database`, obteniendo correos y hashes MD5 fáciles de descifrar. | **High**<br>*Open* | Parametrizar todas las consultas de búsqueda y almacenar las contraseñas utilizando funciones de derivación lentas y con *salt* como **Argon2id** o **bcrypt**. |
| **07** | **D** *(Denial of Service)* | **Proceso:** `Application Server` | **Denegación de servicio por subida de archivos masivos o peticiones sin límite:** Un atacante envía ficheros de gran tamaño en el formulario de quejas (*Complaint*) o lanza ráfagas masivas de peticiones contra la API, saturando la CPU, memoria y disco del servidor. | **Medium**<br>*Open* | Limitar el tamaño máximo de subida de archivos (p. ej., 2 MB), validar extensiones/MIME types en el servidor y aplicar *rate limiting* por IP y usuario. |
| **08** | **E** *(Elevation of Privilege)* | **Proceso:** `Application Server` / `B2B API` | **Elevación de privilegios horizontal (IDOR) y vertical:** Un cliente autenticado accede a rutas de administración (`/#/administration`) o consulta cestas de otros clientes modificando el identificador numérico en `GET /rest/basket/1` sin validación de permisos. | **High**<br>*Open* | Implementar control de acceso basado en roles (RBAC) en cada endpoint y comprobar en el servidor que el recurso solicitado pertenece al usuario de la sesión. |

![AMENAZA1](./IMG/1amenazastride.png)
![AMENAZA2](./IMG/2amenazastride.png)
![AMENAZA3](./IMG/3amenazastride.png)
![AMENAZA4](./IMG/4amenazastride.png)
![AMENAZA5](./IMG/5amenazastride.png)
![AMENAZA6](./IMG/6amenazastride.png)
![AMENAZA7](./IMG/7amenazastride.png)
![AMENAZA8](./IMG/8amenazastride.png)

### Evidencia visual 2: Panel de amenazas y reporte generado en Threat Dragon

> **Instrucción:** Inserta aquí una captura del panel de amenazas dentro de Threat Dragon o del informe PDF exportado desde el botón **Reporte** (*Report*).

![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_1.png)
![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_2.png)
![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_3.png)
![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_4.png)
![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_5.png)
![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_6.png)
![Informe STRIDE generado en OWASP Threat Dragon](./IMG/page_7.png)


*Figura 2. Vista del informe generado desde OWASP Threat Dragon 2.6.2 con el resumen de amenazas STRIDE y sus respectivas mitigaciones.*

---

## 4. Comprobación práctica en OWASP Juice Shop y entregables

### Enunciado (Pasos 7 y 8)

* **Paso 7:** Verificar en la tienda desplegada (`http://127.10.0.1:3000`) al menos dos de las amenazas identificadas en el modelo STRIDE.
* **Paso 8:** Guardar el modelo modificado en una carpeta local sin alterar el repositorio original (`threat-model-juice-shop-stride.json`) y exportar el informe en PDF desde Threat Dragon.



### Desarrollo de las comprobaciones empíricas

#### Prueba 1: Suplantación de identidad (S) mediante Inyección SQL en el Login

* **Amenaza verificada:** ID 01 (`Application Server` — *Spoofing*).
* **Procedimiento:**
  1. Accedemos en el navegador a `http://127.10.0.1:3000/#/login`.
  2. En el campo **Email**, introducimos la carga útil SQL `' OR 1=1--` para forzar que la condición `WHERE` de la consulta evalúe siempre a verdadero y comente la comprobación de contraseña.
  3. En el campo **Password**, escribimos cualquier valor (por ejemplo, `1234`) y pulsamos **Log in**.
* **Resultado observado:** El servidor acepta la consulta manipulada e inicia sesión automáticamente como el primer usuario registrado en la base de datos (`admin@juice-sh.op`), confirmando la suplantación completa sin credenciales.

> **Instrucción:** Inserta aquí la captura de pantalla del navegador mostrando la sesión iniciada como `admin@juice-sh.op` tras introducir `' OR 1=1--`.

[PLANTILLAS/plantilla.md](PEPR/IMG/Captura de pantalla 2026-11-24 201201.png)
[PLANTILLAS/plantilla.md](PEPR/IMG/Captura de pantalla 2026-11-24 201216.png)

*Figura 3. Acceso no autorizado con privilegios de administrador en Juice Shop mediante inyección SQL en el formulario de inicio de sesión.*

---

#### Prueba 2: Revelación de información (I) por exposición del directorio `/ftp`

* **Amenaza verificada:** ID 06 (`Local File System` — *Information Disclosure*).
* **Procedimiento:**
  1. Sin necesidad de iniciar sesión, escribimos directamente en la barra de direcciones del navegador la ruta `http://127.10.0.1:3000/ftp`.
* **Resultado observado:** El servidor devuelve el índice completo de archivos internos almacenados en el sistema de ficheros (`acquisitions.md`, `coupons_2013.md.bak`, `package.json.bak`, etc.), permitiendo su visualización y descarga directa por cualquier usuario externo.

> **Instrucción:** Inserta aquí la captura de pantalla del listado de archivos expuesto en `http://127.10.0.1:3000/ftp`.

[PLANTILLAS/plantilla.md](PEPR/IMG/Captura de pantalla 2026-11-24 201244.png)

*Figura 4. Listado público de documentos confidenciales y copias de seguridad en la ruta `/ftp` de OWASP Juice Shop.*

---


---

## 5. Reflexión final

El ejercicio de modelado de amenazas con **STRIDE** sobre el DFD de OWASP Juice Shop pone de manifiesto que la mayoría de los fallos críticos de seguridad no surgen de errores aislados, sino de **asunciones de confianza incorrectas en las fronteras del sistema**. Cuando el `Application Server` confía en las entradas procedentes del `Angular Frontend` sin parametrizar consultas SQL ni validar cantidades de compra, o cuando expone directamente rutas del `Local File System` como `/ftp`, cualquier usuario externo puede comprometer la confidencialidad, integridad y autorización de toda la plataforma. Realizar este análisis en la **Fase II de Diseño** del ciclo de vida de desarrollo seguro (S-SDLC) permite detectar y mitigar estos riesgos estructurales antes de escribir el código.

---

## 6. Preguntas de autocomprobación

### 1. ¿Qué es una frontera de confianza? Señala dos en el DFD de Juice Shop.

* **Definición:** Una **frontera de confianza** (*trust boundary*) es el límite lógico o arquitectónico dentro de un sistema donde cambia el nivel de confianza, control o privilegio sobre los datos. Todo flujo de información que atraviesa una frontera de confianza debe tratarse como potencialmente malicioso y someterse a controles de validación, autenticación y autorización.
* **Dos ejemplos en el DFD de Juice Shop:**
  1. La frontera entre `Angular Frontend` (ejecutado en el navegador bajo control del usuario) y `Application Server` (el servidor backend), por donde circulan `API Requests` y `API Responses`.
  2. La frontera entre `Application Server` y el servicio externo `Google` para el intercambio de tokens de identidad en el flujo `OAuth2`.

### 2. ¿Por qué un actor externo no puede sufrir «Tampering» en STRIDE por elemento, pero un flujo sí?

* En la metodología **STRIDE por elemento**, los **actores externos** representan entidades humanas o sistemas fuera del alcance y control de nuestra aplicación; sobre ellos únicamente evaluamos si alguien puede hacerse pasar por el actor (**S** — *Spoofing*) o si el actor puede negar haber efectuado una acción (**R** — *Repudiation*).
* Por el contrario, un **flujo de datos** representa el transporte de información a través de un canal de red entre dos componentes, por lo que sí es susceptible de ser interceptado y alterado en tránsito (**T** — *Tampering*) si carece de cifrado (TLS) o validación de integridad.

### 3. El almacén «Local File System» guarda ficheros a los que se accede por la web. ¿Qué letra de STRIDE es la más preocupante y por qué?

* La letra más crítica es la **I (Revelación de información / *Information Disclosure*)**, porque en Juice Shop este almacén publica rutas accesibles vía HTTP (`/ftp`, `/encryptionkeys` y `/support/logs`) sin control de acceso ni restricción de listado de directorios, permitiendo que cualquier atacante descargue documentos internos, copias de seguridad y claves criptográficas.

### 4. ¿Cómo priorizarías dos amenazas con el mismo impacto pero distinta facilidad de explotación?

* Se prioriza siempre en primer lugar **la amenaza más fácil de explotar**. El nivel de riesgo se determina combinando probabilidad e impacto ($\text{Riesgo} = \text{Probabilidad} \times \text{Impacto}$); a igualdad de impacto, aquella vulnerabilidad que requiere menos conocimientos técnicos, herramientas estándar o ninguna autenticación previa presenta una probabilidad de explotación mucho mayor (criterio empleado también en estándares como **CVSS** o **DREAD**).

### 5. ¿En qué momento del ciclo de vida debe revisarse el modelo de amenazas, además del diseño inicial?

* El modelo de amenazas es un artefacto vivo que debe revisarse y actualizarse en tres momentos clave además del diseño inicial:
  1. **Ante cambios relevantes de arquitectura o funcionalidad** (nuevos endpoints, cambios en el esquema de base de datos o nuevos mecanismos de autenticación).
  2. **Al integrar servicios o APIs de terceros** que introduzcan nuevas fronteras de confianza.
  3. **Tras un incidente de seguridad o auditoría**, para incorporar los vectores descubiertos y verificar la eficacia de las contramedidas aplicadas.
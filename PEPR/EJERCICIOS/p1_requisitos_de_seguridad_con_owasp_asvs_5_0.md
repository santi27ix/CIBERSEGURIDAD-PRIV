# Módulo 03 · Construcción de software seguro · Sesión 1 - Ejercicio 1

**P1 · Requisitos de seguridad con OWASP ASVS 5.0**

* **Apertura:** martes, 24 de noviembre de 2026, 00:00
* **Cierre:** martes, 24 de noviembre de 2026, 21:00
* **Objetivo:** Convertir requisitos de OWASP ASVS 5.0 (nivel 1) en historias de usuario y casos de abuso para Juice Shop, y comprobar en la propia tienda si se cumplen.
* **Duración:** 40 min (5 de lectura, 25 de trabajo y 10 de reflexión final y autocomprobación).
* **Requisitos:** Navegador, Juice Shop levantado con BreakingLab y una hoja de cálculo o un documento.
* **Diapositivas de referencia:** 21 (*SDL, Phase One: Requirements*), 50 (*CLASP: Capture security requirements*) y 117 (*Fase I: requisitos*).

---

## 1. Puesta en marcha: BreakingLab con Juice Shop

**BreakingLab** (`https://github.com/TiiZss/BreakingLab`) es un script Bash que levanta laboratorios vulnerables con Docker. Funciona en Linux (Kali/Ubuntu) y en Windows con WSL2.

### Requisitos del entorno

* Linux o Windows 10/11 con WSL2 y Docker (Docker Engine con el plugin `docker compose`, o Docker Desktop con integración WSL).
* Herramientas que comprueba el instalador: `docker`, `git`, `curl` y `nc`. Para el panel web hace falta `python3`.
* Unos 8 GB de RAM en el equipo para ir cómodo con Docker y las herramientas a la vez.
* La imagen de Juice Shop (`tiizss/juice-shop:latest`, Juice Shop 20.3.0) ocupa unos 120 MB comprimida. Descárgala antes de empezar si tu conexión es lenta.

### Arranque y descarga del código fuente

```bash
git clone https://github.com/TiiZss/BreakingLab.git
cd BreakingLab
chmod +x breakinglab.sh
./breakinglab.sh start juiceshop      # Privado: http://127.10.0.1:3000
./breakinglab.sh status               # Comprueba que aparece en verde
# Alternativa gráfica: ./breakinglab.sh web  ->  http://127.10.0.1:8765
```

* Abre `http://127.10.0.1:3000` en el navegador. Debe aparecer la tienda **OWASP Juice Shop**.
* Si el script pide `sudo` es para añadir un alias en `/etc/hosts`; si no lo tienes, la dirección IP funciona igual.
* Al terminar: `./breakinglab.sh stop juiceshop`.
* Clonado del código fuente de la versión 20.3.0 para revisión estática:

```bash
git clone --depth 1 --branch v20.3.0 https://github.com/juice-shop/juice-shop.git
```

> **Aviso importante:** Juice Shop y el resto de máquinas de BreakingLab son vulnerables a propósito. No las expongas a Internet, no las parchees (el objetivo es encontrar los fallos, no arreglar el laboratorio) y no lances estas herramientas contra sistemas que no sean tuyos. Escanear o atacar sistemas ajenos sin autorización es delito.

---

## 2. Evaluación de requisitos OWASP ASVS 5.0 (Nivel 1)

### Enunciado (Pasos 1 a 4)

1. Abre el ASVS 5.0 en `https://github.com/OWASP/ASVS` (carpeta `5.0/en`) o en la web del proyecto OWASP ASVS. Fíjate en la estructura: 17 capítulos (*V1 Encoding and Sanitization* … *V17 WebRTC*) y requisitos con número `capítulo.sección.requisito` y nivel (1, 2 o 3).
2. Recorre Juice Shop como cliente: registro, inicio de sesión, buscador, cesta, opiniones de productos y subida de ficheros en «Complaint».
3. Elige 6 requisitos de nivel 1 que apliquen a esas funciones. Pistas de capítulos: V1 (inyección), V2 (validación y lógica de negocio), V5 (ficheros), V6 (autenticación), V8 (autorización), V9 (tokens como JWT) y V11 (criptografía).
4. Para cada requisito, rellena una fila de la tabla con su historia de usuario positiva, caso de abuso, forma de comprobación y estado de cumplimiento.

### Resultados: Matriz de requisitos OWASP ASVS 5.0 (Nivel 1)

Tras levantar el contenedor de Juice Shop con BreakingLab (`./breakinglab.sh start juiceshop` en `http://127.10.0.1:3000`) y contrastar las pruebas en el navegador con el código fuente de la rama `v20.3.0`, se han seleccionado y ordenado 6 requisitos de OWASP ASVS 5.0 (Nivel 1) priorizando las vulnerabilidades de mayor impacto y facilidad de verificación:

| ID ASVS | Historia de usuario (positiva) | Caso de abuso | Cómo lo compruebo en Juice Shop | ¿Se cumple? |
| :--- | :--- | :--- | :--- | :--- |
| **1.2.4** *(V1 - Inyección)* | Como cliente quiero que mis búsquedas y el inicio de sesión usen consultas parametrizadas para que nadie pueda inyectar SQL y robar mis datos o credenciales. | Como atacante quiero inyectar comandos SQL en el buscador o en el login para volcar toda la tabla de usuarios (correos y hashes de contraseña) y entrar como administrador sin conocer su clave. | En el buscador introduzco `')) UNION SELECT id, email, password, '4', '5', '6', '7', '8', '9' FROM Users--` (o `' OR 1=1--` en el login) y me devuelve los registros de la base de datos; en `routes/search.ts` y `routes/login.ts` se ve que concatenan variables directamente en SQLite. | **No** *(Crítica)* |
| **2.2.1** *(V2 - Lógica de negocio)* | Como cliente y responsable de la tienda quiero que el servidor valide las cantidades y calcule los precios reales de la cesta para que nadie altere el importe desde el navegador. | Como atacante quiero interceptar las peticiones al añadir productos a la cesta o aplicar descuentos para modificar el precio o enviar cantidades negativas y llevarme productos gratis o con saldo a favor. | Capturo la petición `POST /api/BasketItems` con las DevTools o un proxy y cambio `"quantity": 1` por `"quantity": -10` (o manipulo los parámetros del pedido); el backend acepta el valor negativo y resta dinero del total a pagar. | **No** *(Crítica)* |
| **11.4.2** *(V11 - Criptografía)* | Como cliente quiero que mi contraseña se almacene usando una función de derivación lenta y con *salt* (como Argon2id o bcrypt) para que no puedan obtener mi clave original si filtran la BD. | Como atacante quiero obtener los hashes de contraseñas filtrados de la tabla `Users` para crackearlos en segundos por diccionario o fuerza bruta al estar guardados con un algoritmo obsoleto sin *salt*. | Tras sacar los hashes por SQLi, compruebo que son cadenas hexadecimales de 32 caracteres (MD5) que se rompen al instante en CrackStation/Hashcat; en `lib/insecurity.ts` se confirma el uso de `crypto.createHash('md5')`. | **No** *(Alta)* |
| **8.2.2** *(V8 - Autorización)* | Como cliente quiero que el servidor verifique que mi usuario es el dueño de la cesta o perfil consultado para que ningún otro cliente vea ni modifique mis datos o pedidos. | Como atacante quiero alterar el identificador numérico de la cesta o del usuario en las llamadas a la API (IDOR/BOLA) para espiar y modificar las cestas y datos personales de otros clientes. | Inicio sesión con mi cuenta, intercepto la llamada `GET /rest/basket/2` y la repito cambiando el ID a `/rest/basket/1`; el servidor me devuelve la cesta del administrador sin devolver un error `403 Forbidden`. | **No** *(Alta)* |
| **6.3.1** *(V6 - Autenticación)* | Como cliente quiero que el sistema bloquee o ralentice los intentos repetidos de inicio de sesión fallidos para proteger mi cuenta frente a ataques de fuerza bruta. | Como atacante quiero probar cientos de contraseñas seguidas contra el formulario de inicio de sesión sin sufrir bloqueos ni captchas para adivinar la clave de cualquier cuenta (como la del administrador). | Accedo al formulario de login (`/#/login`), introduzco el correo `admin@juice-sh.op` y pruebo repetidamente contraseñas incorrectas (más de 15 veces seguidas); la tienda no bloquea los intentos, no aplica retardos ni muestra un CAPTCHA. | **No** *(Alta)* |
| **6.2.1** *(V6 - Autenticación)* | Como cliente quiero que el sistema exija contraseñas de al menos 8 caracteres (15 recomendados) al registrarme para evitar que comprometan mi cuenta fácilmente. | Como atacante quiero registrar cuentas con contraseñas triviales de menos de 8 caracteres (como `12345`) o aprovechar políticas débiles para comprometer cuentas mediante ataques de diccionario básicos. | Accedo al formulario de registro (`/#/register`) y creo una cuenta usando la contraseña `12345` (5 caracteres); la tienda acepta el registro y permite iniciar sesión sin bloquearlo en el backend. | **No** *(Media)* |

---

## 3. Caso de abuso y criterio de aceptación verificable

### Enunciado (Paso 5)

Escribe un caso de abuso en formato «Como atacante quiero… para…», con un criterio de aceptación verificable («Dado… cuando… entonces…»).

### Resultado

Para blindar el punto más crítico de negocio (la manipulación de precios y cantidades en la cesta bajo el requisito **ASVS 2.2.1**), el caso de abuso y su prueba de aceptación quedan definidos así:

* **Caso de abuso:** Como atacante quiero interceptar y modificar los parámetros de precio o cantidad en las peticiones HTTP al gestionar mi cesta de la compra para reducir el importe total a cero o generar saldo negativo a mi favor y obtener productos sin pagar.
* **Criterio de aceptación verificable:**
  * **Dado** que un usuario autenticado dispone de una cesta activa en la tienda y selecciona un producto cuyo precio unitario en el catálogo del servidor es mayor que 0 €,
  * **cuando** envía una petición `POST` o `PUT` al endpoint `/api/BasketItems` manipulando el cuerpo JSON con una cantidad negativa (por ejemplo, `"quantity": -5`) o añadiendo un campo de precio alterado (`"price": 0.01`),
  * **entonces** el servidor debe ignorar cualquier precio enviado por el cliente, rechazar cantidades menores o iguales a 0 devolviendo un código de estado `HTTP 400 Bad Request` y no modificar ni añadir el artículo a la cesta.

---

## 4. Reflexión final

### Enunciado (Paso 6)

Elige de tu tabla el requisito que no se cumple en Juice Shop y que te parezca más grave, y explica en una frase por qué.

### Resultado

El requisito incumplido más grave es el **1.2.4** (Inyección SQL en buscador y login) porque permite a cualquier atacante sin autenticar extraer de un solo golpe toda la base de datos de usuarios y contraseñas —que al estar en MD5 quedan expuestas en texto claro— además de tomar el control total de la cuenta de administrador y de la tienda.

---

## 5. Preguntas de autocomprobación

### 1. ¿Qué diferencia hay entre un requisito funcional y uno de seguridad? Pon un ejemplo de Juice Shop.

* **Requisito funcional:** Define qué debe hacer el sistema para aportar valor al usuario o al negocio. *Ejemplo en Juice Shop:* «Como cliente quiero poder buscar productos por nombre o descripción en la barra de búsqueda».
* **Requisito de seguridad:** Define cómo debe protegerse esa función frente a usos maliciosos o abusos para garantizar la confidencialidad, integridad y disponibilidad. *Ejemplo en Juice Shop (ASVS 1.2.4):* «Todas las consultas del buscador a la base de datos deben ejecutarse mediante consultas parametrizadas u ORM para impedir inyecciones SQL».

### 2. ¿Por qué conviene que los requisitos de seguridad sean verificables? ¿Qué pasa si escribes «la aplicación debe ser segura»?

* Los requisitos tienen que ser verificables para poder traducirlos a pruebas técnicas objetivas (como un test automatizado o un criterio *Dado / Cuando / Entonces* en QA o auditoría) y saber con certeza si pasan o fallan.
* Si escribes «la aplicación debe ser segura», es una frase ambigua que no se puede medir: ningún desarrollador sabe qué implementar exactamente y ningún auditor puede demostrar objetivamente si se cumple o no.

### 3. ¿Qué nivel de ASVS pedirías para una tienda online con pagos? ¿Y para la web de un banco?

* **Tienda online con pagos:** **Nivel 2** como mínimo, ya que gestiona datos personales (PII), sesiones de clientes y transacciones económicas donde un fallo de lógica de negocio o robo de sesión tiene impacto financiero directo.
* **Web de un banco:** **Nivel 3** (al menos en toda la operativa crítica de transferencias, autenticación y gestión de cuentas), porque el propio estándar ASVS reserva el Nivel 3 para aplicaciones de muy alto valor o riesgo crítico donde las consecuencias de una brecha son severas.

### 4. El buscador de Juice Shop construye la consulta SQL concatenando texto. ¿Qué requisito del capítulo V1 incumple?

* Incumple el requisito **1.2.4**, que exige usar consultas parametrizadas, sentencias preparadas u ORM seguros en lugar de concatenar cadenas de texto en las consultas SQL/NoSQL.

### 5. ¿En qué fase del S-SDLC se deberían definir estos requisitos y quién debería firmarlos?

* Se definen desde la **Fase I de Requisitos** (tal como marcan *SDL en Phase One: Requirements*, diapositiva 21, y *CLASP: Capture security requirements*, diapositiva 50), antes de escribir una sola línea de código.
* Deben aprobarlos y firmarlos conjuntamente el **Product Owner / responsable de producto** (por el impacto en negocio) y el **responsable de seguridad / AppSec** (por el riesgo técnico y el cumplimiento normativo, como el **RGPD** y la **LOPDGDD** en España), revisándolos además en cada iteración o *sprint* cuando cambie el alcance.
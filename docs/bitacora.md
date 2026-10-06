# Bitácora — Práctica 10: Avisos con Imágenes

---

## 1. Experimentos Realizados

### Experimento A3: Coil con su propio cliente HTTP
* **¿Qué se hizo?**: Se removió la configuración personalizada de `.components { add(OkHttpNetworkFetcherFactory(callFactory = { cliente })) }` en `di/ImagenesModule.kt`, haciendo que Coil utilizara su `ImageLoader` e `OkHttpClient` por defecto.
* **Resultado observado**:
  * Las imágenes almacenadas en el caché local en disco continuaban viéndose, pero al borrar el caché (`adb shell run-as mx.tec.avisos rm -rf cache/image_cache`) o intentar cargar imágenes nuevas, las tarjetas del tablón mostraban un rectángulo gris.
  * En Logcat no aparecía ningún registro del interceptor de logs de la app.
  * En los logs del servidor Docker (`docker compose logs api`) aparecieron peticiones `GET /api/imagenes/... HTTP/1.1` retornando **`401 Unauthorized`**.
* **Conclusión**: Si Coil no utiliza el mismo `OkHttpClient` inyectado por Hilt (que posee el interceptor con `Authorization: Bearer <token>` y el `Authenticator` para renovar tokens), las peticiones de imágenes se realizan sin credenciales y el servidor las rechazaría con error 401.

---

### Experimento D2: Carga de imágenes sin reducir
* **¿Qué se hizo?**: En la rama de experimento se modificó `CompresorDeImagen.kt` cambiando sus constantes a `LADO_MAX = 8000` y `CALIDAD = 100`.
* **Resultado observado**:
  * Al intentar subir una foto original de cámara/galería (ej. 4032x3024 de 2.0 MB), el archivo JPEG resultante **aumentó** de tamaño a **3.6 MB** (3,765,422 bytes). Recomprimir a calidad 100 un JPEG previamente comprimido agrega peso guardando con detalle el ruido sin aportar información.
  * Al enviar la petición `POST /api/imagenes`, el servidor respondió **`413 Request Entity Too Large`**.
  * Inicialmente, la app mostraba un mensaje genérico *"El servidor respondió 413."* en lugar de explicar claramente el error al usuario (lo cual se solucionó posteriormente en el punto D3 al usar la respuesta del servidor `mensaje ?: "El servidor respondió ${e.code()}."`).
* **Conclusión**: Reducir y comprimir la imagen en el cliente antes de transmitirla es fundamental para no sobrepasar los límites de los servidores, ahorrar datos móviles y evitar fallos por memoria o entidades demasiado grandes.

---

## 2. Ejercicio D4 — Las imágenes de tu reto

### 1. Tamaño y `LADO_MAX`
* **Ancho visual en pantalla**: En la pantalla más grande que mostrará la imagen (ej. teléfono o tablet Full HD / 1080p en una tarjeta a ancho completo), el espacio útil de la imagen es de aproximadamente ~1080px.
* **Elección de `LADO_MAX`**: **`1280 px`**.
* **Justificación**: Proporciona nitidez óptima en pantallas de alta densidad (hdpi/xhdpi/xxhdpi) y reduce de manera drástica el tamaño del archivo (de ~4000x3000px y ~3.5 MB a ~960x1280px y ~35 KB) sin desperdiciar almacenamiento ni datos móviles del usuario.

### 2. Visibilidad y Permisos (`GET /imagenes/{clave}`)
* **Visibilidad**: Para este reto/aplicación de avisos, cualquier usuario autenticado con sesión activa tiene permiso para ver las imágenes publicadas en el tablón.
* **Cambios en el servidor (`GET /imagenes/{clave}`)**:
  * Si la imagen perteneciera a un recurso privado (por ejemplo, identificación personal o comprobante de pago), el endpoint requeriría autenticación mediante el token JWT en el encabezado `Authorization`.
  * El servidor comprobaría en la base de datos si el usuario autenticado tiene relación de propiedad o permiso explícito sobre el recurso antes de devolver el archivo; en caso contrario, respondería con HTTP `403 Forbidden`.

### 3. Gestión de imágenes huérfanas (subidas sin publicar)
* **¿Quién la borra?**: El **servidor**.
* **Mecanismo de limpieza**:
  * Al subir una imagen mediante `POST /imagenes`, la API registra el archivo con un estado temporal o marca de tiempo (*timestamp*).
  * Si el usuario se arrepiente y no completa la publicación del aviso (`POST /avisos`), la imagen queda desvinculada (huérfana).
  * Una tarea programada (*cron job* o proceso en segundo plano) en el servidor analiza periódicamente las imágenes en el almacenamiento (S3/RustFS) que tengan más de cierto tiempo (ej. 24 horas) sin estar referenciadas en ninguna tabla de la base de datos y las elimina definitivamente.

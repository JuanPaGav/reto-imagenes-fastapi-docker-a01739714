# Bitácora — Práctica 10: Avisos con Imágenes

## Ejercicio D4 — Decisiones sobre las imágenes del reto

### 1. Tamaño y `LADO_MAX`
* **Tamaño visual**: En la pantalla más grande que la muestra (ej. un teléfono con resolución Full HD / 1080p en tarjeta de ancho completo), la imagen se renderiza a un ancho efectivo de hasta ~1080px.
* **Elección de `LADO_MAX`**: **1280 px**. Este valor mantiene la nitidez visual adecuada en dispositivos de alta densidad sin desperdiciar ancho de banda (reduce archivos de ~4000x3000 y 3.5 MB a ~960x1280 y ~35 KB).

### 2. Visibilidad y Permisos (`GET /imagenes/{clave}`)
* **Visibilidad**: Cualquier usuario con sesión activa puede ver las imágenes asociadas a los avisos del tablón.
* **Cambios en `GET /imagenes/{clave}`**:
  * Si la imagen fuera privada o restringida solo a su creador, el endpoint requeriría autenticación mediante el encabezado `Authorization: Bearer <token>`.
  * El servidor verificaría si la clave pertenece a un aviso al que el usuario tiene acceso; de lo contrario, respondería con `403 Forbidden`.

### 3. Gestión de imágenes huérfanas (subidas sin publicar)
* **¿Quién la borra?**: El **servidor**.
* **Mecanismo**: Cuando se sube una imagen (`POST /imagenes`), se registra con un estado temporal o marca de tiempo. Si la imagen no es vinculada a un aviso publicado en la base de datos dentro de un periodo determinado (ej. 24 horas), una tarea programada (*cron job* / proceso de limpieza en el servidor) la elimina automáticamente del almacenamiento (S3 / sistema de archivos) para no acumular basura.

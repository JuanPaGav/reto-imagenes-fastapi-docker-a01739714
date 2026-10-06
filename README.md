# Práctica 10 — Avisos con imágenes

Código de arranque de la Práctica 10 de TC2007B.

Es la app de Avisos de la Práctica 8 (sesión, stream y notificaciones) con tres
cambios que no son el tema de la práctica:

- **Hilt** en vez del `AppContainer` (lo de la Práctica 9).
- **La dirección del servidor** sale de `local.properties` (`avisos.api`). Por omisión,
  `http://10.0.2.2:8000/api/`: tu computadora vista desde el emulador.
- **HTTP en claro**, permitido solo hacia `10.0.2.2` y solo en debug.

La app no habla con `startdroid.com`: habla con **tu** servidor, el del anexo de imágenes.

## Cómo empezar

1. Levanta el servidor (https://github.com/rpversontec/reto-imagenes-fastapi-docker):
   `cp .env.example .env ; docker compose up -d ; bash smoke.sh`
2. Crea una cuenta de profesor en tu servidor:
   https://startdroid.com/consola?api=http://localhost:8000/api
3. Clona este repositorio, ábrelo en Android Studio y corre la app en el emulador.
   Entra con tu cuenta: debes ver el tablón.
4. Sigue la guía: https://startdroid.com/practicas/avisos-con-imagenes.html

Para un teléfono de verdad, usa el túnel del servidor y pon su dirección en
`local.properties`: `avisos.api=https://…trycloudflare.com/api/`



## Cómo trabajar

Haz un commit en cada checkpoint de la guía:

    git add -A ; git commit -m "checkpoint a2"

Si algo se rompe sin remedio, `git restore .` te regresa al último checkpoint bueno.
Los experimentos van en su rama, con commit en la rama antes de volver a `main`.

## Uso de IA

Todo commit con código generado por IA debe declararlo con un trailer
`Co-Authored-By`. Ver la política completa en la guía.

## Entrega

Ver la rúbrica en la guía. Sube todas las ramas: `git push origin --all`.

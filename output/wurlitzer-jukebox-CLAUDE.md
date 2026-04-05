# Wurlitzer Jukebox

Aplicación web local que convierte una colección de MP3s en una rockola Wurlitzer interactiva.
Servidor Express en Node.js + frontend React/Vite. Accesible en LAN.

## Comandos

- `npm run dev` — Servidor Express (puerto 3000) + Vite dev server (puerto 5173) en paralelo
- `npm run build` — Compila el frontend React a `dist/`
- `npm start` — Corre Express en producción sirviendo `dist/` (después del build)
- `npm test` — Vitest para tests del scanner y artwork

## Tech Stack

Node.js + Express (servidor) + React 18 + Vite (frontend) + CSS puro (estilos) + iTunes API (carátulas) + HTML5 Audio

## Arquitectura

### Estructura de archivos de música esperada
```
config.musicFolder/
  Artista/
    Album/
      01 - Titulo.mp3
      02 - Titulo.mp3
```

### Flujo de datos
1. Express escanea `config.musicFolder` al arrancar → construye árbol en memoria
2. `GET /api/library` devuelve el árbol completo al frontend
3. `GET /api/artwork?artist=X&album=Y` → busca en caché local → si no hay, fetcha iTunes API → cachea → devuelve imagen
4. `GET /stream/:artist/:album/:filename` → stream del MP3 con soporte Range headers (para seek)
5. Frontend React gestiona todo el estado en `App.jsx`, sin librería de estado externa

### Patrones clave
- **Estado global en App.jsx** — library, selectedAlbum, currentSong, isPlaying fluyen hacia abajo por props
- **useAudio hook** — encapsula TODO lo relacionado con HTMLAudioElement; ningún otro componente lo toca directamente
- **CSS puro siempre** — no introducir Tailwind ni ningún framework de UI; el diseño del jukebox requiere CSS custom
- **Sin base de datos** — la biblioteca se escanea en memoria al arrancar; no persistir nada

## Reglas No Negociables

1. **El servidor debe escuchar en `0.0.0.0`**, no en `localhost` — requerido para acceso LAN.
2. **El streaming de MP3 debe soportar Range headers** — sin esto el seek de HTML5 Audio no funciona.
3. **CSS puro únicamente** — no instalar Tailwind, shadcn, ni ningún framework de estilos.
4. **Un componente por archivo**, máximo 200 líneas. Si el CSS de un componente crece, moverlo a su propio archivo en `styles/`.
5. **Las carátulas siempre tienen fallback** — si iTunes falla, mostrar `no-cover.png`; nunca mostrar un img roto.

## Reference Document

Ver `blueprint.md` en este directorio para el diseño completo:
modelo de datos, detalle de todos los endpoints, jerarquía completa de componentes y sistema de diseño.

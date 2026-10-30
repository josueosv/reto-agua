# 💧 RETO AGUA — Sistema de Resultados en Vivo

Segunda instalación del sistema de Reto Speed, con un cambio clave: **el cronómetro se opera directamente en la pantalla principal**, para eliminar el retraso al iniciar y detener.

## ¿Qué cambió respecto a Reto Speed?

| | Reto Speed | Reto Agua |
|---|---|---|
| Dónde se inicia/detiene el cronómetro | Panel `/control` (otro dispositivo) | **Pantalla principal `/`** |
| Cómo corre el cronómetro | Sincronizado vía base de datos (cada toque viaja al servidor y vuelve) | **Local en el navegador**: el toque se registra al instante, sin red |
| Registro del tiempo | Copiar el tiempo y guardarlo en `/control` | **Botón GUARDAR** en la pantalla: se asigna al equipo en turno |
| Avance de equipo | Manual | Automático al siguiente equipo sin tiempo |

Se mantiene igual: 16 equipos en el mismo orden, formato `M:SS:CC`, ranking automático (menor tiempo arriba, sin tiempo en orden de carrera), top 4 visible con scroll automático, PIN de operador y reinicio con doble confirmación.

## Operación el día del evento (pantalla principal)

1. Abre `/` en la computadora conectada a la pantalla y ponla en pantalla completa (F11).
2. El panel de la derecha muestra el **equipo en turno** (se elige solo; se puede cambiar con el selector).
3. **ESPACIO** o botón **▶ INICIAR** → arranca. **ESPACIO** o **■ DETENER** → se congela (en amarillo).
4. **ENTER** o **✓ GUARDAR** → guarda el tiempo para el equipo en turno, reordena el ranking, pone el cronómetro en cero y pasa al siguiente equipo.
   - La primera vez pide el PIN de operador; queda recordado en esa computadora.
   - Si el equipo ya tenía tiempo, pide confirmación antes de reemplazarlo.
5. **↺ REINICIAR** → descarta el tiempo sin guardar. **▶ CONTINUAR** reanuda si se detuvo por error.
6. **H** oculta/muestra el panel de botones, para dejar la vista limpia al público.

Si la página se recarga por accidente, el cronómetro conserva su estado (incluso si estaba corriendo).

## Panel `/control` (opcional, otro dispositivo)

Con PIN. Sirve para correcciones: ingreso manual de tiempos, borrar el tiempo de un equipo y **"Borrar todo y empezar de cero"**. Los cambios aparecen en la pantalla principal en 1–2 segundos.

---

## Instalación como proyecto independiente (Railway)

Reto Agua es **una instalación separada**: su propio repositorio, su propio servicio y su **propia base de datos**. No toca nada de Reto Speed.

### 1. Crear el repositorio en GitHub

Crea un repo vacío `reto-agua` en GitHub y luego, desde esta carpeta:

```bash
cd reto-agua
git init
git add .
git commit -m "Reto Agua - sistema de resultados"
git branch -M main
git remote add origin https://josueosv:TU_TOKEN@github.com/josueosv/reto-agua.git
git push -u origin main
```

### 2. Nuevo proyecto en Railway

1. [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo** → `reto-agua`.
2. En el mismo proyecto: **New** → **Database** → **Add PostgreSQL**.
3. En el servicio de la app → **Variables**:
   - `DATABASE_URL` = `${{Postgres.DATABASE_URL}}`
   - `ADMIN_PIN` = tu PIN (si no se define, es `1234`)
4. **Settings → Networking → Generate Domain**.

Al primer arranque la app crea la tabla y siembra los 16 equipos.

## Correr localmente

```bash
npm install
DATABASE_URL="postgresql://usuario:pass@localhost:5432/retoagua" ADMIN_PIN="1234" npm start
```

Pantalla principal: http://localhost:3000 · Control: http://localhost:3000/control

## Estructura

```
reto-agua/
├── src/
│   ├── server.js     # Express + API (ranking, tiempos, reset)
│   └── db.js         # PostgreSQL: esquema, siembra, ranking
├── public/
│   ├── index.html    # Pantalla principal con cronómetro y botones
│   ├── control.html  # Panel de correcciones
│   └── img/reto_agua.png
├── package.json
└── railway.json
```

## Notas técnicas

- **Por qué ya no hay delay:** el instante de inicio/parada se toma en el mismo navegador donde se presiona el botón, antes de cualquier comunicación con el servidor. Solo el resultado final viaja a la base de datos.
- El botón responde al **presionar** (no al soltar), y la barra espaciadora ignora la autorepetición si se mantiene presionada.
- Si falla el guardado (por ejemplo, sin internet), el tiempo queda congelado en pantalla y se puede reintentar sin perderlo. El indicador "EN VIVO" se pone rojo si no hay conexión.

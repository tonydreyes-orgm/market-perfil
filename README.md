# Despliegue de sitio HTML estático con URL fija y gratuita

Documentación de cómo se publicó la propuesta ejecutiva ("Penetración de Mercado Cárnico RD")
para que **cualquier agente** pueda repetirlo como modelo con otros sitios HTML estáticos.

> Contenido del proyecto: `index.html` (presentación tipo deck) + `logo_cliente.jpg`.
> Carpeta del proyecto: `C:\Users\ORGM\Downloads\market perfil\server`

---

## 1. Estado actual (resumen)

El sitio está **publicado y vivo** en dos hostings gratuitos, con URL fija (no aleatoria):

| Plataforma | URL fija | Comando para compartir |
|---|---|---|
| **GitHub Pages** | https://tonydreyes-orgm.github.io/market-perfil/ | abrir en navegador |
| **Surge.sh** | https://tonydreyes-market-perfil.surge.sh | abrir en navegador |

Ambas sirven el mismo `index.html`. Verificado: HTTP 200, `<title>` correcto.

**Cuentas ya configuradas (reutilizables):**
- GitHub: usuario `tonydreyes-orgm`, repo `market-perfil` (rama `master`, público). `gh` autenticado vía keyring de Windows.
- Surge: email `tonydreyes@or-gm.com`, dominio `tonydreyes-market-perfil.surge.sh`. Sesión guardada en `C:\Users\ORGM\.netrc` (token). **No se necesita la contraseña para re-publicar** en esta máquina.

---

## 2. Playbook reutilizable (pasos exactos)

### 2.1 Servir en local (preview rápido)
El `docker-compose.yml` original usa nginx, pero **el daemon de Docker no estaba corriendo** en esta máquina.
Para contenido estático basta Python:

```bash
cd "C:/Users/ORGM/Downloads/market perfil/server"
python -m http.server 8080 --bind 0.0.0.0
# abrir http://localhost:8080/index.html
```

### 2.2 Publicar en GitHub Pages (URL fija gratis)
Requisito: cuenta GitHub + autenticación del agente (`gh auth login -w` o PAT).
GitHub **desactivó la autenticación por contraseña** en 2021; usa flujo de dispositivo o token.

```bash
export PATH="$PATH:/c/Users/ORGM/Downloads/gh/bin"   # gh instalado localmente
cd "C:/Users/ORGM/Downloads/market perfil/server"

# 1) crear repo y subir (solo la primera vez)
gh repo create market-perfil --public --source . --description "..." --push

# 2) activar Pages (solo la primera vez)
gh api repos/tonydreyes-orgm/market-perfil/pages -X POST \
  -f "source[branch]=master" -f "source[path]=/"

# 3) en adelante, para actualizar:
git add -A && git commit -m "update" && git push
# Pages se recompila solo (~1 min). El primer build da 404 temporalmente.
```

### 2.3 Publicar en Surge (URL fija gratis)
Requisito: cuenta Surge (gratis). El CLI **no lee credenciales por stdin redirigido**;
la forma fiable es obtener el token por API y guardarlo en `~/.netrc`.

```bash
# Obtener token (crea la cuenta si no existe): basic auth email:password
TOKEN=$(curl -sS -X POST "https://surge.surge.sh/token" \
  -u "tonydreyes@or-gm.com:<<password>>" | grep -o '"token": *"[^"]*"' | grep -o '[a-f0-9]*$')

# Guardar en ~/.netrc  (formato que espera surge)
printf 'machine surge.surge.sh\n  login tonydreyes@or-gm.com\n  password %s\n' "$TOKEN" > "$USERPROFILE/.netrc"

# Publicar (ya autenticado, sin prompts)
cd "C:/Users/ORGM/Downloads/market perfil/server"
surge --project . --domain tonydreyes-market-perfil.surge.sh
```

> ⚠️ Surge escribe un archivo `CNAME` en la carpeta al publicar. **Borrarlo**
> si también vas a usar GitHub Pages, porque un `CNAME` en la raíz del repo
> fuerza un dominio personalizado y rompe Pages.

### 2.4 Túnel público temporal (NO fijo)
Para compartir algo ya mismo sin cuenta, `cloudflared tunnel --url http://localhost:8080`
da una URL `*.trycloudflare.com`, pero **es aleatoria y efímera** (cambia al reiniciar).
No sirve si se necesita ruta fija.

---

## 3. Comando simple para reactivar el HTML

El sitio ya está publicado de forma permanente, así que "activarlo" es abrir la URL.
Para levantarlo de nuevo en local o re-publicar tras un cambio:

**Preview local (1 línea en el prompt):**
```bash
python -m http.server 8080 --directory "C:/Users/ORGM/Downloads/market perfil/server"
```

**Re-publicar en Surge (ya autenticado vía ~/.netrc):**
```bash
surge --project "C:/Users/ORGM/Downloads/market perfil/server" --domain tonydreyes-market-perfil.surge.sh
```

**Re-publicar en GitHub Pages:**
```bash
cd "C:/Users/ORGM/Downloads/market perfil/server" && git add -A && git commit -m "update" && git push
```

---

## 4. PROMPT / SKILL: "Sitios HTML con ruta fija"

Plantilla lista para pegar en un nuevo chat y que otro agente haga lo mismo.
Explica **por qué importa** y **cómo**, aprovechando que ya existen las cuentas de GitHub y Surge.

```markdown
# Skill: Publicar sitio HTML estático con URL fija y gratuita

## Por qué importa
Un archivo HTML solo es útil si otras personas pueden verlo. Abrirlo del disco
(localhost) no sirve para compartir. Los túneles rápidos (ngrok/cloudflared
quick tunnel) dan URL, pero SON ALEATORIAS y se caen al reiniciar: no son
"ruta fija". Para entregar una propuesta, portafolio o demo a un cliente se
necesita una URL ESTABLE, legible y gratuita. Eso se consigue publicando el
HTML en un hosting estático free (GitHub Pages o Surge), no con un túnel.

## Cuándo usar
El entregable es HTML/CSS/JS estático (sin backend) y el usuario quiere una
URL para compartir con el mundo de forma permanente y sin pagar.

## Cómo hacerlo (infra ya disponible en esta máquina)
Cuentas configuradas; reutilízalas, no crees otras:

- GitHub Pages: usuario `tonydreyes-orgm`, repo `market-perfil` (rama master).
  `gh` ya autenticado. Para actualizar: editar archivos y `git push`; Pages
  recompila solo.
- Surge: email `tonydreyes@or-gm.com`, dominio `tonydreyes-market-perfil.surge.sh`.
  El token está en `C:\Users\ORGM\.netrc`, así que `surge --project . --domain
  <dominio>` publica sin pedir login.

Pasos para un sitio NUEVO:
1. Deja los archivos estáticos en una carpeta.
2. (GitHub) `gh repo create <repo> --public --source . --push`, luego
   `gh api repos/<user>/<repo>/pages -X POST -f source[branch]=master -f source[path]=/`.
   URL: https://<user>.github.io/<repo>/.
3. (Surge) obtén token con `POST https://surge.surge.sh/token` (basic auth
   email:password) y guárdalo en ~/.netrc; luego
   `surge --project . --domain <sub>.surge.sh`.
4. Verifica con `curl -I <url>` (HTTP 200).
5. Documenta las URLs y las cuentas en un README del proyecto.

Reglas de oro:
- NUNCA metas contraseñas planas en repos públicos; usa tokens en ~/.netrc
  o credential manager.
- Borra el CNAME que Surge crea antes de commitear a GitHub Pages.
- GitHub desactivó login por contraseña: usa `gh auth login -w` (flujo
  dispositivo) o un PAT, nunca el password de la cuenta.
- El primer build de GitHub Pages tarda ~1 min y da 404 hasta terminar.
```

---

## 5. Lecciones aprendidas / troubleshooting

- **Docker daemon no disponible** en este equipo → servir con `python -m http.server`.
- **GitHub ya no acepta la contraseña de la cuenta** (ni aunque entre por Google).
  Usar `gh auth login -w` (código de dispositivo que el usuario aprueba en el
  navegador) o un PAT con scope `repo`.
- **Surge CLI ignora stdin redirigido** en el login interactivo. Solución:
  `POST /token` con basic auth → token → `~/.netrc`.
- **CNAME de Surge** contamina el repo de Pages → borrarlo.
- **GitHub Pages 404 al inicio** = build en curso, no error.
- **Túneles quick (cloudflared/ngrok)** = URL efímera; úsalos solo para preview,
  no para entregables fijos.

---
*Generado por el agente de despliegue. Modelo reutilizable para sitios HTML
estáticos con URL fija gratuita (GitHub Pages + Surge).*

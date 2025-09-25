- Dominio/Hosting: `https://teamskillevolution.com` sirviendo **200**; `www` → **301** a raíz vía Cloudflare (Proxied).  
  _Verificación:_ `curl -I https://teamskillevolution.com` y `curl -I https://www.teamskillevolution.com`
- Firebase Hosting en prod (`tsevolution-prod`) con **CI/CD** por GitHub Actions (FIREBASE_TOKEN) — **OK**.
- DNS (Cloudflare):  
  - A `@` → `199.36.158.100` (Firebase).  
  - CNAME `www` → `tsevolution-prod.web.app` (**Proxied**).  
  _Verificación:_ `dig +short A teamskillevolution.com` · `dig +short www.teamskillevolution.com`
- Correo: SPF/DKIM/DMARC activos; **DMARC = quarantine** (sp=quarantine).  
  _Verificación:_ `dig txt google._domainkey.teamskillevolution.com +short` · `dig txt _dmarc.teamskillevolution.com +short`
- SEO base: `robots.txt`, `sitemap.xml`, canonical, OG/Twitter, JSON-LD **Organization/WebSite/FAQPage/Service** — **OK**.
- GA4: ID **G-938KWH1VZX**, script con `debug_mode` vía `?debug=1`, eventos `click` con `event_category=CTA` y `event_label` (`agenda-20`, `whatsapp`) — **OK**.
- Assets: `img/hero-b.webp` y `img/og-b.jpg` (provisorias), favicons (32/192/512) y `site.webmanifest` con headers correctos — **OK**.
# Pendientes críticos (prioridad 1)
1) **DMARC → `p=reject` (tras 5–7 días sin fallos)**  
   - Acción: en Cloudflare **DNS**, editar `_dmarc` y cambiar `p=quarantine` → `p=reject` (mantén `sp=quarantine` o `sp=reject` según política).  
   - Verificación:  
     ```bash
     dig txt _dmarc.teamskillevolution.com +short
     ```
   - **DONE** si muestra `p=reject`.

2) **Conversiones GA4 (completarlas)**  
   - `cta_agenda` (hecha) y **crear** `cta_whatsapp` (**evento** con condiciones: `event_name=click` + `event_label=whatsapp`) y **marcar como evento clave**.  
   - Verificación: **Vista de depuración** con `?debug=1` y **Administrador → Conversiones** activo.

3) **CI/CD más seguro (recomendado)** — _migrar de token a Service Account_  
   - Crear **Service Account** con rol **Firebase Hosting Admin**, subir JSON a GitHub como secreto `FIREBASE_SERVICE_ACCOUNT` y cambiar workflow a `google-github-actions/auth@v2`.  
   - Snippet de workflow (referencia para cuando ejecutemos la tarea):  
     ```yaml
     - uses: google-github-actions/auth@v2
       with:
         credentials_json: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
     - run: npm i -g firebase-tools
     - run: firebase deploy --only hosting --project tsevolution-prod
     ```

4) **Cabeceras de seguridad (Firebase Hosting)**  
   - Añadir en `firebase.json` (headers globales):  
     - `Referrer-Policy: strict-origin-when-cross-origin`  
     - `X-Content-Type-Options: nosniff`  
     - `Permissions-Policy: interest-cohort=()`  
     - (Opcional) `X-Frame-Options: SAMEORIGIN`  
     - (CSP opcional, con cuidado por GA4)
   - Verificación:  
     ```bash
     curl -I https://teamskillevolution.com | grep -E 'Referrer-Policy|X-Content-Type-Options|Permissions-Policy|X-Frame-Options'
     ```

---

## Pendientes operativos (prioridad 2)
5) **Imágenes definitivas**  
   - Reemplazar `img/hero-b.webp` (≤300 KB) y `img/og-b.jpg` (1200×630) por las versiones de marca.  
   - Verificación:  
     ```bash
     curl -I https://teamskillevolution.com/img/hero-b.webp
     curl -I https://teamskillevolution.com/img/og-b.jpg
     ```

6) **Página 404 y futuras redirecciones**  
   - Agregar `404.html` en raíz (estilo simple).  
   - (Opcional) Tabla de redirecciones futuras en `firebase.json` si cambian URLs.  
   - Verificación: probar una ruta inexistente y comprobar 404.

7) **GitHub: protección básica**  
   - Activar **branch protection** en `main` (1 aprobación opcional, requerir checks verdes del workflow).  
   - Verificación: Settings → Branches → Rules → aplicado a `main`.

8) **Monitoreo**  
   - Uptime: activar alerta (StatusCake/BetterStack/Cloudflare Health).  
   - Verificación: check verde y alerta de email configurada.

---

## Notas
- El **certificado en Firebase para `www`** puede quedar “pendiente” sin impacto porque Cloudflare (Proxied) hace el TLS al borde y redirige a raíz — no bloquea operaciones.
- Search Console ya está “Correcto”; cada cambio mayor de contenido → re-enviar sitemap.


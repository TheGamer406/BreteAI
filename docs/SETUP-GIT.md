# Setup de Git — repo padre + submódulos

Estructura objetivo: un repo **padre** (`BreteAI`) que referencia los repos **hijos** como **submódulos**.

Ahora mismo los hijos son repos git locales independientes y están **ignorados** en el `.gitignore` del padre (para evitar el warning de "repos embebidos" mientras no existan los remotos). Cuando crees los repos en GitHub, conviértelos en submódulos con estos pasos.

## 1. Crear los repos en GitHub

Crea 4 repos vacíos (sin README) en tu cuenta:

- `BreteAI` (padre)
- `BreteAI-Backend`
- `BreteAI-Frontend`
- `BreteAI-Infra`

## 2. Subir cada repo hijo

Desde cada carpeta hija (ejemplo backend):

```bash
cd BreteAI-Backend
git remote add origin git@github.com:TheGamer406/BreteAI-Backend.git
git branch -M main
git push -u origin main
cd ..
```

Repite para `BreteAI-Frontend` y `BreteAI-Infra`.

## 3. Convertir los hijos en submódulos del padre

Desde la raíz del repo padre:

```bash
# Quitar las 3 líneas de /BreteAI-*/ del .gitignore primero (ver nota abajo)

# Mover temporalmente las carpetas locales y agregarlas como submódulos
for r in Backend Frontend Infra; do
  rm -rf "BreteAI-$r"
  git submodule add git@github.com:TheGamer406/BreteAI-$r.git "BreteAI-$r"
done

git commit -m "chore: agrega submódulos backend, frontend e infra"
git push
```

> **Nota:** antes del paso 3, borra del `.gitignore` del padre las líneas:
> ```
> /BreteAI-Backend/
> /BreteAI-Frontend/
> /BreteAI-Infra/
> ```

## 4. Clonar el proyecto completo (referencia futura)

```bash
git clone --recurse-submodules git@github.com:TheGamer406/BreteAI.git
# o si ya lo clonaste:
git submodule update --init --recursive
```

## Recordatorio de privacidad

- `resources/` (CVs, `perfil.toon` real, notas) **nunca** se sube: está en `.gitignore`.
- Copia `config/perfil.example.toon` → `resources/perfil.toon` y edítalo con tus datos.
- Los `.env` están ignorados; usa `.env.example` en cada repo como plantilla.

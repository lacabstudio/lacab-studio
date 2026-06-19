# LaCab Studio — Suite du déploiement (à reprendre après validation client)

> État au 19/06/2026 : le site est en ligne en **preview** sur GitHub Pages, en attente de
> validation client. Le nom de domaine n'a **pas encore** été basculé : `www.lacabstudio.fr`
> pointe toujours vers l'ancien site Webflow.

---

## 📍 Où en est-on

| Élément | Valeur |
|---|---|
| Dépôt GitHub | `lacabstudio/lacab-studio` (public) |
| Compte gh authentifié | `lacabstudio` |
| **URL de preview (à partager au client)** | **https://lacabstudio.github.io/lacab-studio/** |
| Domaine final visé | `www.lacabstudio.fr` (racine `lacabstudio.fr` → redirige vers www) |
| Registrar / gestion DNS | **Infomaniak** (manager.infomaniak.com) |
| Site Webflow | **toujours en ligne** sur le domaine, intact |

Le `CNAME` (custom domain) est **temporairement désactivé** pour que l'URL github.io reste
consultable. Concrètement, dans `.github/workflows/deploy.yml`, la ligne qui copie le `CNAME`
est commentée. Le fichier `CNAME` lui-même est conservé dans le dépôt.

---

## ✅ Étapes restantes (dans l'ordre)

### Étape 1 — Le client a validé → réactiver le domaine côté GitHub
*(Claude peut faire cette étape : dire « le client a validé, réactive le domaine ».)*

Dans `.github/workflows/deploy.yml`, remplacer le bloc :
```yaml
          # NOTE: CNAME temporairement desactive pour exposer l'URL github.io
          # (preview client). Reactiver en decommentant la ligne ci-dessous.
          # cp CNAME public/
          cp .nojekyll public/
```
par :
```yaml
          cp CNAME .nojekyll public/
```
Puis :
```bash
git add .github/workflows/deploy.yml
git commit -m "Reactiver le custom domain www.lacabstudio.fr"
git push origin main
gh api -X PUT repos/lacabstudio/lacab-studio/pages -f cname=www.lacabstudio.fr
```

### Étape 2 — Configurer les DNS chez Infomaniak
*(À faire par toi sur manager.infomaniak.com — Claude n'y a pas accès.)*

1. **manager.infomaniak.com** → **Domaines** → **lacabstudio.fr** → onglet **Zone DNS**.
2. **Supprimer** les anciens enregistrements pointant vers Webflow :
   - le(s) `A` sur `@` vers une IP Webflow,
   - le `CNAME` sur `www` vers `proxy-ssl.webflow.com` (ou similaire).
   - ⚠️ **Ne pas toucher** aux `MX` (emails) ni aux `TXT`.
3. **Ajouter** le CNAME pour `www` :
   | Type | Nom | Cible |
   |---|---|---|
   | CNAME | `www` | `lacabstudio.github.io.` |
4. **Ajouter** les 4 enregistrements A sur `@` (racine) :
   | Type | Nom | Cible |
   |---|---|---|
   | A | `@` | `185.199.108.153` |
   | A | `@` | `185.199.109.153` |
   | A | `@` | `185.199.110.153` |
   | A | `@` | `185.199.111.153` |
5. **Enregistrer**. Propagation DNS : de quelques minutes à 24-48 h.

### Étape 3 — Vérifier la propagation + activer HTTPS
*(Claude peut faire cette étape une fois les DNS posés : dire « les DNS sont en place ».)*

Vérifs (manuelles ou par Claude) :
```bash
dig +short www.lacabstudio.fr        # doit montrer lacabstudio.github.io / IPs GitHub
dig +short lacabstudio.fr            # doit montrer les IP 185.199.108-111.153
curl -sI https://www.lacabstudio.fr  # doit servir le nouveau site (200)
```
Quand GitHub a validé le domaine, forcer le HTTPS :
```bash
gh api -X PUT repos/lacabstudio/lacab-studio/pages -f https_enforced=true
```
(ou Settings → Pages → cocher « Enforce HTTPS »).

### Étape 4 — Nettoyer côté Webflow
*(À faire par toi.)*

Une fois `www.lacabstudio.fr` servi par GitHub Pages :
- Retirer le domaine personnalisé du projet Webflow (éviter conflit / facturation du plan hébergé).
- Garder le projet Webflow en mode édition si besoin de réexporter plus tard.

---

## 🔁 Comment mettre à jour le site plus tard

Le site se redéploie **automatiquement** à chaque `git push` sur `main` (via GitHub Actions).
Pour modifier le contenu : éditer les fichiers HTML/CSS/images, puis :
```bash
git add -A && git commit -m "..." && git push origin main
```

**Rappels techniques utiles :**
- Les **pages brouillons** (`style-guide.html`, `nos-services.html`, `projets/renewal.html`,
  `401.html`) restent dans le dépôt mais ne sont **jamais publiées** (liste blanche dans le workflow).
- **Noms de fichiers images** : rester en **ASCII** (pas d'accents). Les accents cassent sur
  GitHub Pages (bug NFC/NFD déjà corrigé pour les fichiers « bulletin »).
- **Lightbox** : les `url` des images plein écran doivent être des **chemins relatifs**
  (`../images/...`), pas des URL absolues — sinon elles chargent à l'infini tant que le
  domaine n'est pas en place.
- Les balises `og:image` / `twitter:image` / `JSON-LD` utilisent volontairement des **URL
  absolues** `https://www.lacabstudio.fr/...` (correct pour le partage social / SEO).

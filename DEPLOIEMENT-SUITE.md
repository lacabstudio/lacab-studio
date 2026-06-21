# LaCab Studio — État du déploiement & maintenance

> **Bascule terminée le 21/06/2026.** Le site est en ligne en production sur
> **https://www.lacabstudio.fr**, servi par **GitHub Pages** (export statique Webflow).
> Le domaine ne pointe plus vers Webflow.

---

## 📍 État actuel

| Élément | Valeur |
|---|---|
| Dépôt GitHub | `lacabstudio/lacab-studio` (public) |
| Compte gh authentifié | `lacabstudio` |
| URL de production | **https://www.lacabstudio.fr** |
| Apex `lacabstudio.fr` | redirige (301) vers `www` |
| Registrar / gestion DNS | **Infomaniak** (manager.infomaniak.com) |
| Site Webflow | domaine personnalisé retiré du projet ✔ |

**Zone DNS Infomaniak (état final) :**

```
NS     lacabstudio.fr   ns41.infomaniak.com
NS     lacabstudio.fr   ns42.infomaniak.com
A      @                185.199.108.153
A      @                185.199.109.153
A      @                185.199.110.153
A      @                185.199.111.153
CNAME  www              lacabstudio.github.io.
```

> Pas d'enregistrements `AAAA` (GitHub Pages = IPv4 uniquement, c'est voulu).
> Les anciens enregistrements Webflow (A `@` → 198.202.211.1, CNAME `www` → cdn.webflow.com,
> TXT `_webflow`) ont été supprimés.

---

## ✅ Étapes réalisées

1. **Custom domain réactivé côté GitHub** — workflow `deploy.yml` copie de nouveau le `CNAME`,
   et `gh api ... pages -f cname=www.lacabstudio.fr` posé.
2. **DNS configurés chez Infomaniak** — 4 A + CNAME www (voir tableau ci-dessus).
3. **Propagation vérifiée** — `www` et apex renvoient les IP GitHub sur les resolvers publics ;
   site servi en HTTPS 200.
4. **Domaine retiré du projet Webflow** — plus de conflit / facturation hébergement Webflow.

### ⏳ Dernier point en cours : Enforce HTTPS

Le « Enforce HTTPS » nécessite que GitHub ait émis le certificat TLS du domaine (peut prendre
jusqu'à ~1 h après la bascule DNS). Tant que le certificat n'existe pas, l'API renvoie
`certificate does not exist yet`. Une fois prêt :

```bash
gh api -X PUT repos/lacabstudio/lacab-studio/pages -F https_enforced=true
```
⚠️ Utiliser `-F` (booléen typé), **pas** `-f` (qui envoie une chaîne → erreur 422).
(ou Settings → Pages → cocher « Enforce HTTPS »).

---

## 🔁 Mettre à jour le site

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
  (`../images/...`), pas des URL absolues.
- Les balises `og:image` / `twitter:image` / `JSON-LD` utilisent volontairement des **URL
  absolues** `https://www.lacabstudio.fr/...` (correct pour le partage social / SEO).

**Piège cache DNS local :** après la bascule, ton réseau local (box / partage de connexion)
peut continuer à afficher l'ancien site Webflow en cache (TTL 12 h) alors que tout le reste
d'internet voit déjà GitHub. Pour vérifier sans le cache : tester en 4G/5G, ou
`dig +short www.lacabstudio.fr @8.8.8.8`. Flush macOS :
`sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`.

# Guide de Déploiement Frontend InsForge sur Vercel

Ce guide explique comment déployer le frontend InsForge (dashboard React) sur Vercel.

## ✅ Prérequis

- Compte Vercel (créez-en un gratuitement sur [vercel.com](https://vercel.com))
- Repository Git (GitHub, GitLab, ou Bitbucket)
- Ce repository poussé sur votre plateforme Git
- Node.js 18+ installé localement (pour les tests)

## 📋 Ce qui a été préparé

Les éléments suivants ont déjà été configurés dans le projet :

### 1. **vercel.json** ✅
Fichier de configuration Vercel qui définit :
- **buildCommand**: `npm run build:shared-schemas && npm run build:ui && npm run build:frontend`
- **outputDirectory**: `dist/frontend` (répertoire de sortie Vite)
- **rewrites SPA**: Toutes les routes React Router sont redirigées vers `/index.html`
- **variables d'environnement**: Configuration pour `VITE_PUBLIC_POSTHOG_KEY`

### 2. **.env.local** ✅
Fichier d'environnement local avec :
- `VITE_API_BASE_URL`: URL du backend (par défaut: `http://localhost:7130`)
- `VITE_PUBLIC_POSTHOG_KEY`: Clé PostHog pour l'analytics (optionnel)
- `VITE_DEBUG_MODE`: Mode debug (optionnel)

### 3. **Build Configuration** ✅
- `frontend/vite.config.ts`: Configuration Vite optimisée pour la production
- Output: `dist/frontend/` (conforme à la structure monorepo)
- Plugins: React, Tailwind CSS, SVGR

## 🚀 Étapes de Déploiement

### Étape 1: Préparer le Repository Git

```bash
# Vous êtes déjà dans le repository
# Assurez-vous que tous les changements sont commités
git status
git add .
git commit -m "chore: prepare for Vercel deployment"
git push origin main  # ou votre branche principale
```

### Étape 2: Connecter à Vercel

#### Option A: Via Interface Web (Recommandé)

1. Allez sur https://vercel.com/dashboard
2. Cliquez **"Add New... > Project"**
3. Sélectionnez votre plateforme Git (GitHub, GitLab, Bitbucket)
4. Connectez votre compte Git et autorisez Vercel
5. Sélectionnez votre repository `insforge` (ou le nom approprié)
6. Vercel détectera automatiquement que c'est un monorepo Vite

#### Option B: Via CLI

```bash
npm install -g vercel
vercel login
vercel link  # Liez votre projet local à Vercel
```

### Étape 3: Configurer les Paramètres de Build

Dans le dashboard Vercel :

1. **Build Command**:
   ```
   npm run build:shared-schemas && npm run build:ui && npm run build:frontend
   ```

2. **Output Directory**:
   ```
   dist/frontend
   ```

3. **Root Directory** (si demandé):
   ```
   ./
   ```

> **Note**: Vercel peut auto-détecter ces paramètres depuis `vercel.json`. Vérifiez simplement qu'ils sont corrects.

### Étape 4: Configurer les Variables d'Environnement

Dans le dashboard Vercel → **Settings > Environment Variables**:

#### ✅ Variables Essentielles:

| Variable | Valeur | Environnement |
|----------|--------|---------------|
| `VITE_API_BASE_URL` | URL de votre backend | Production (et autres) |
| `VITE_PUBLIC_POSTHOG_KEY` | Votre clé PostHog | Production (optionnel) |

#### Exemple de Configuration:

**Pour Production**:
- `VITE_API_BASE_URL`: `https://api.votre-insforge.com`
- `VITE_PUBLIC_POSTHOG_KEY`: `phc_votre_clé_posthog`

**Pour Preview** (branches de développement):
- `VITE_API_BASE_URL`: `http://localhost:7130` (ou URL de développement)
- `VITE_PUBLIC_POSTHOG_KEY`: Laissez vide ou valeur dev

> **Important**: Ne commitez PAS les secrets dans le repository. Utilisez uniquement le dashboard Vercel.

### Étape 5: Déployer

#### Option A: Déploiement Automatique (Recommandé)

Le déploiement se fait automatiquement à chaque push sur votre branche principale :

```bash
git push origin main
# Vercel détecte le changement et commence le build automatiquement
```

Vous pouvez suivre l'avancement dans le dashboard Vercel.

#### Option B: Déploiement Manual

```bash
vercel --prod
```

## 📊 Après le Déploiement

### Vérification

1. **Accédez à votre URL Vercel**:
   - URL défaut: `https://insforge-xxxxx.vercel.app`
   - Domaine personnalisé: À configurer dans **Settings > Domains**

2. **Testez les Routes SPA**:
   ```
   - https://your-app.vercel.app/ ✅ (Page d'accueil)
   - https://your-app.vercel.app/dashboard ✅ (Route React Router)
   - https://your-app.vercel.app/cloud ✅ (Route React Router)
   ```

3. **Vérifiez la Console du Navigateur**: Les erreurs réseau vers l'API sont normales si le backend n'est pas configuré.

### Domaine Personnalisé

1. Dans Vercel → **Project Settings > Domains**
2. Ajoutez votre domaine (ex: `app.votre-domaine.com`)
3. Suivez les instructions de configuration DNS

## 🔗 Intégration Backend

### Cas 1: Backend InsForge Hébergé

Si vous avez un backend InsForge en production :

1. **Configurez `VITE_API_BASE_URL`** sur Vercel vers votre backend:
   ```
   https://api.your-insforge.region.insforge.app
   ```

2. **CORS**: Assurez-vous que votre backend autorise les requêtes CORS depuis votre domaine Vercel.

### Cas 2: Backend Local / Développement

Pour le développement local :

1. **Le proxy Vite fonctionne** pendant `npm run dev`:
   ```bash
   npm run dev:frontend
   # Les requêtes /api/* sont proxifiées vers le backend local
   ```

2. **En production** : Le proxy ne fonctionne pas, vous avez besoin d'une vraie URL backend.

### Cas 3: Sans Backend (Démo / Visualisation)

Si vous déployez juste pour montrer l'UI :

1. L'application affichera les pages normalement
2. Les appels API échoueront (404 ou CORS) → normal
3. Vous verrez les erreurs de connexion dans la console

Options pour améliorer :
- Créer des **serverless functions** Vercel pour mocker l'API
- Utiliser **MSW** (Mock Service Worker) pour les mocks côté client
- Intégrer avec un service backend externe

## ⚠️ Problèmes Courants et Solutions

### ❌ Erreur: "build` command not found"

**Solution**: Vérifiez que le `buildCommand` dans `vercel.json` ou le dashboard Vercel utilise les bonnes dépendances.

```json
{
  "buildCommand": "npm run build:shared-schemas && npm run build:ui && npm run build:frontend"
}
```

### ❌ Routes React Router retournent 404

**Solution**: Vérifiez que `vercel.json` contient les rewrites SPA:

```json
{
  "rewrites": [
    {
      "source": "/:path*",
      "destination": "/index.html"
    }
  ]
}
```

### ❌ Variables d'environnement non chargées

**Solution**: 
- Assurez-vous qu'elles commencent par `VITE_PUBLIC_` (exposées au navigateur)
- Rechargez la page après avoir ajouté les variables
- Redéployez le projet

```bash
vercel --prod --force  # Force un rebuild
```

### ❌ Build échoue avec erreur de dépendances

**Solution**: Nettoyez le cache Vercel :
- Dashboard → **Settings > Git > Vercel for Git**
- Cliquez sur **Clear Build Cache** et redéployez

### ❌ Gros fichiers / Performance

**Note**: Le bundle JavaScript est ~3.7 MB (compressé). C'est acceptable pour une dashboard, mais à optimiser si nécessaire :
- Vérifiez `frontend/vite.config.ts` pour les code-splitting optimisations
- Utilisez `npm run build` localement pour analyser les dépendances

## 🔍 Surveillance et Logs

### Vérifier les Logs de Build

Dans Vercel Dashboard :
1. **Deployments > [Votre déploiement] > Logs**
2. Consultez les logs de build et runtime
3. Les erreurs sont généralement dues aux variables d'environnement ou aux commandes de build

### Monitoring en Production

Utilisez les outils Vercel intégrés :
- **Analytics**: Visibilité sur les performances
- **Monitoring**: Erreurs et logs runtime
- **Logs**: Accès aux logs serveur/client

## 📝 Résumé des Fichiers Modifiés

| Fichier | Statut | Détail |
|---------|--------|--------|
| `vercel.json` | ✅ Créé | Configuration SPA et build |
| `.env.local` | ✅ Créé | Variables d'environnement locales |
| `frontend/vite.config.ts` | ✅ Vérifié | Aucune modification nécessaire |
| `frontend/package.json` | ✅ Vérifié | Scripts de build OK |
| `package.json` (root) | ✅ Vérifié | Scripts de build OK |

## 🎯 Checklist Finale

Avant de déployer, assurez-vous que :

- [ ] Repository Git est à jour (`git push origin main`)
- [ ] `vercel.json` est présent à la racine
- [ ] Build locale fonctionne (`npm run build:shared-schemas && npm run build:ui && npm run build:frontend`)
- [ ] `.env.local` contient les variables de développement
- [ ] Vous avez un compte Vercel
- [ ] Repository est connecté à Vercel
- [ ] Variables d'environnement Production sont configurées sur Vercel
- [ ] `VITE_API_BASE_URL` pointe vers votre backend (ou URL valide)

## 📚 Ressources Utiles

- [Documentation Vercel](https://vercel.com/docs)
- [Vercel + Vite](https://vercel.com/guides/how-to-deploy-vite-projects-to-vercel)
- [Vite Configuration](https://vitejs.dev/config/)
- [React Router Documentation](https://reactrouter.com/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

## ❓ Questions / Support

Pour des questions sur :
- **Déploiement Vercel**: Consultez la [documentation Vercel](https://vercel.com/docs)
- **Configuration du projet**: Ouvrez une issue sur le [repository InsForge](https://github.com/InsForge/InsForge)
- **Backend InsForge**: Consultez la documentation du projet principal

---

**Dernière mise à jour**: Configuration adaptée pour Vercel avec monorepo Vite ✅

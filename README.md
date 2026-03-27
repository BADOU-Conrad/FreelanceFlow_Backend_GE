# FreelanceFlow Backend

Backend API pour la plateforme FreelanceFlow de gestion de projets freelance.

## 📋 Table des matières

- [Installation](#installation)
- [Conventions de commit](#conventions-de-commit)
- [Workflow Git](#workflow-git)
- [Pipeline CI/CD](#pipeline-cicd)
- [Commandes](#commandes)
- [Husky & Git Hooks](#husky--git-hooks)

## 🚀 Installation

### Prérequis

- Node.js 18.x ou 20.x
- npm 9.x+
- Git

### Setup local

```bash
# 1. Cloner le repository
git clone https://github.com/BADOU-Conrad/FreelanceFlow_Backend_GE.git
cd FreelanceFlow_Backend_GE

# 2. Installer les dépendances
npm install

# 3. Initialiser Husky (automatique via "prepare" script)
# Déjà fait lors de npm install !

# 4. Vérifier que les Git hooks sont actifs
ls -la .husky/
```

## 📝 Conventions de commit

Les commits doivent suivre le format **Conventional Commits** :

```
<type>: <description> (<issue>)
```

### Types autorisés

- `feat` - Nouvelle fonctionnalité
- `fix` - Correction de bug
- `docs` - Documentation
- `style` - Formatage, pas de changement logique
- `refactor` - Refactorisation sans nouveau feature
- `perf` - Amélioration de performance
- `test` - Ajout/modification de tests
- `chore` - Tâches de build, dépendances
- `ci` - Modification CI/CD

### Exemples valides

```bash
git commit -m "feat: ajouter authentification JWT (#123)"
git commit -m "fix: corriger bug login (#45)"
git commit -m "docs: mise à jour API endpoints (#78)"
git commit -m "test: ajouter tests unitaires (#102)"
```

### ❌ Exemples REJETÉS

```bash
git commit -m "nouveau feature"                    # ❌ No type, no issue
git commit -m "feat: update"                       # ❌ No issue
git commit -m "FEAT: Something (#123)"             # ❌ Type uppercase
```

## 🔄 Workflow Git

```
1. Créer une branche depuis dev
   git checkout -b feature/ma-feature

2. Faire des commits avec numéro d'issue
   git commit -m "feat: description (#123)"

3. Pousser et créer une PR vers stage
   git push origin feature/ma-feature
   → Créer PR sur GitHub (destination: stage)

4. Vérifications automatiques (GitHub Actions)
   ✅ check-source-branch (la PR vient de dev)
   ✅ validate-commits (messages valides)
   ✅ run-tests (tous les tests passent)

5. Code review + merge vers stage
   → Tests, déploiement staging

6. Créer une PR stage → main
   ✅ check-source-branch (la PR vient de stage)
   ✅ Approbation humaine
   → Merge → Production 🚀
```

## 🔐 Pipeline CI/CD

### Vérifications automatiques

| Workflow | Quand | Vérifie |
|----------|-------|---------|
| **check-source-branch** | PR vers main | Source = stage uniquement |
| **validate-commits** | PR vers stage/main | Commits contiennent #issue |
| **run-tests** | PR vers stage/main | Tests unitaires passent |

### Branch Protection (main)

- ✅ Require status checks to pass:
  - `check-branch`
  - `validate-commits`
  - `run-tests`
- ✅ Require pull request reviews (approbation)
- ✅ Dismiss stale reviews
- ✅ Require branches to be up to date

## 💻 Commandes

```bash
# Installation
npm install

# Développement
npm run dev

# Tests
npm test                    # Lancer tous les tests
npm test -- --watch         # Mode watch

# Lint & Format
npm run lint                # ESLint
npm run format              # Prettier
npm run lint:fix            # ESLint fix + Prettier

# Build
npm run build

# Vérifier Husky
npx husky install          # Si besoin de réinstaller
```

## 🪝 Husky & Git Hooks

Husky exécute automatiquement des vérifications **avant chaque commit** et **avant chaque push**.

### Hooks configurés

#### 1️⃣ **pre-commit** (avant commit)
```bash
Exécute: npx lint-staged
- Lint les fichiers modifiés
- Format avec Prettier
- Corrige les erreurs automatiquement
```

#### 2️⃣ **commit-msg** (validation message)
```bash
Exécute: npx commitlint --edit
- Valide le format: type(scope): description
- Vérifie la présence du #issue
- Rejette si non conforme
```

#### 3️⃣ **pre-push** (avant push)
```bash
Exécute: npm test
- Lance tous les tests
- Rejette le push si tests échouent
```

### Contourner les hooks (INTERDIT en production)

```bash
# ❌ NE PAS UTILISER EN PROD
git commit --no-verify        # Contourner pre-commit
git push --no-verify          # Contourner pre-push
```

## 📚 Ressources

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Husky Documentation](https://typicode.github.io/husky/)
- [GitHub Actions](https://github.com/features/actions)

---

**Développé par l'équipe FreelanceFlow** 🚀
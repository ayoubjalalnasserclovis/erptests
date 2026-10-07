# ERP Tests — Suite de Tests E2E & Workflows Métier

Dépôt dédié et isolé de la **Software Factory** pour l'ERP Stoniz.  
Ce projet regroupe **508 tests automatisés** (dont **278 workflows multi-étapes**) s'exécutant à 100% en local et in-memory avec zéro dépendance externe.

---

## 📊 Métriques du Dépôt

* **Tests Totaux** : **508 tests** (100% passing)
* **Workflows Multi-Étapes** : **278 tests** (47 suites de bout en bout)
* **Fichiers de Test** : **76 fichiers**
* **Bugs Documentés & Corrigés** : **47 bugs** (BUG-001 à BUG-047)
* **Temps d'exécution complet** : ~11 secondes
* **Moteur BDD** : Emulateur Supabase in-memory (`tests/helpers/mock-db.ts`)

---

## 🚀 Démarrage Rapide

### Prérequis
* Node.js $\ge 18$
* npm

### Exécution des Tests

```bash
# Lancer les 508 tests
npm test

# Lancer uniquement les 278 workflows multi-étapes
npm test tests/workflows

# Vérifier la compilation TypeScript
npm run typecheck

# Vérifier la conformité du code
npm run lint
```

---

## 📁 Organisation du Projet

* `tests/workflows/` : 47 suites testant les cycles de vie complets (Achats, Caisse, Travaux, Propria, Banque).
* `tests/actions/` : Tests unitaires ciblés sur les Server Actions Next.js.
* `tests/helpers/mock-db.ts` : Moteur de simulation Supabase in-memory (clonage mémoire, zéro réseau).
* `tests/lib/` : Tests des modules financiers, algorithmes de conciliation et parseur Banque Chaabi.
* `tests/wave0/` à `tests/wave4/` : Matrice RBAC, conformité réglementaire, audits de sécurité et coupe-circuit.
* `BUG_LOG.md` : Registre complet des 47 bugs détectés avec preuves de résolution.
* `AGENT_GUIDE.md` : Guide exhaustif à destination de tout agent IA pour comprendre, maintenir et étendre la suite.

---

## 🤖 Guide Spécifique pour les Agents IA

Consultez [AGENT_GUIDE.md](AGENT_GUIDE.md) pour les détails d'implémentation, les règles d'immutabilité et les patrons de test.

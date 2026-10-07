# Guide de Référence pour Agents IA (AGENT_GUIDE.md)

> **Bienvenue sur `erptests`.**  
> Ce dépôt contient la suite de tests E2E et de workflows multi-étapes de la **Software Factory** de l'ERP.  
> Tout est configuré pour être exécuté **100% en local et in-memory** (zéro appel réseau externe, zéro dépendance Supabase distante).

---

## 1. Vue d'Ensemble & Métriques

| Métrique | Valeur | Description |
| :--- | :---: | :--- |
| **Total Tests** | **508** | 100% au vert sous Vitest |
| **Workflows Multi-Étapes** | **278** | Répartis sur 47 fichiers dans `tests/workflows/` |
| **Fichiers de Test** | **76** | Couvrant actions, libs, RBAC et workflows |
| **Bugs Détectés & Corrigés** | **47** | Référencés de `BUG-001` à `BUG-047` dans `BUG_LOG.md` |
| **Durée d'Exécution** | **~11s** | Exécution ultra-rapide grâce au mock in-memory |

---

## 2. Structure du Répertoire

```
erptests/
├── tests/
│   ├── helpers/
│   │   ├── mock-db.ts            <-- Cœur du moteur BDD in-memory (Supabase Mock)
│   │   ├── supabase.ts          <-- Helper d'injection client Supabase
│   │   └── test-run.ts          <-- Utilitaires d'exécution
│   ├── workflows/               <-- 47 suites multi-étapes (278 tests)
│   │   ├── achats-procurement-lifecycle-workflow.test.ts
│   │   ├── caisse-stoniz-lifecycle-workflow.test.ts
│   │   ├── propia-menage-lifecycle-workflow.test.ts
│   │   ├── reception-vct-pv-workflow.test.ts
│   │   └── ... (47 fichiers)
│   ├── actions/                 <-- Tests unitaires des Server Actions Next.js
│   ├── lib/                     <-- Tests des calculs financiers, parser Chaabi, etc.
│   ├── wave0/ à wave4/          <-- Tests de socle, matrice RBAC, kill-switch
│   └── fixtures/                <-- Données de test (clients, projets, rôles)
├── app/                         <-- Code source des Server Actions testées
├── lib/                         <-- Logique métier, sécurité, calculs financiers
├── BUG_LOG.md                   <-- Registre complet des 47 bugs résolus
├── AGENT_GUIDE.md               <-- Ce document (instructions pour agents)
├── README.md                    <-- Documentation principale du dépôt
├── vitest.config.ts             <-- Configuration du runner de tests Vitest
├── tsconfig.json                <-- Configuration TypeScript
└── package.json                 <-- Dépendances et scripts de test
```

---

## 3. Le Moteur de BDD In-Memory (`tests/helpers/mock-db.ts`)

Pour garantir une vitesse d'exécution maximale et **aucun risque sur la production**, toutes les requêtes Supabase sont interceptées par une base de données en mémoire.

### Comment l'utiliser dans un test :
```typescript
import { createMockSupabaseClient } from '../helpers/mock-db';

// 1. Déclarer les tables initiales avec des données de test
const initialData = {
  projects: [{ id: 'p1', name: 'Villa Palmeraie', status: 'en_cours' }],
  propria_caisse_wallets: [{ id: 'w1', name: 'Caisse Marrakech', is_active: true, balance: 5000 }],
  propria_caisse_transactions: []
};

// 2. Instancier le client mock
const supabase = createMockSupabaseClient(initialData);

// 3. Exécuter vos Server Actions ou requêtes
const result = await myServerAction({ supabase, walletId: 'w1', amount: 200 });

// 4. Vérifier l'état persistant dans la mémoire du mock
const { data: updatedWallet } = await supabase.from('propria_caisse_wallets').select('*').eq('id', 'w1').single();
expect(updatedWallet.balance).toBe(4800);
```

### Capacités du Mock Engine :
* **Filtres supportés** : `.eq()`, `.neq()`, `.in()`, `.is()`, `.gt()`, `.gte()`, `.lt()`, `.lte()`, `.ilike()`, `.contains()`, `.not()`.
* **Pagination & Tri** : `.order()`, `.limit()`, `.range()`.
* **Écritures** : `.insert()`, `.update()`, `.upsert()`, `.delete()`.
* **Immutabilité** : Snapshot automatique par clonage d'objets (évite l'aliasing mémoire).

---

## 4. Règles Métier & Invariants Critiques

Tout agent travaillant sur ce projet doit respecter les invariants suivants :

1. **Isolation Absolue** : Ne jamais appeler d'API externe ou de BDD Supabase distante. Toujours injecter ou mocker via `createMockSupabaseClient`.
2. **Caisse & Dépenses** :
   * Une dépense validée par la Direction (`is_validated: true`) est **strictement immutable** (`BUG-031`, `BUG-045`).
   * Une caisse fermée (`is_active: false`) refuse tout mouvement (`BUG-037`).
3. **Travaux & Artisans** :
   * Tout artisan payé doit avoir un RIB marocain valide à 24 chiffres et une attestation d'assurance de moins de 6 mois (`BUG-016`).
   * Tout changement de statut en lot terminé exige une facture artisan déposée (`BUG-030`).
   * La signature contradictoire du PV de réception capture l'IP et l'User-Agent (`BUG-017`, `BUG-028`).
4. **Propria & Conciergerie** :
   * Une clé supprimée (soft-deleted) ne peut pas enregistrer de mouvement physique (`BUG-035`).
   * Un inventaire terminé (`termine`) est verrouillé en écriture (`BUG-036`).
   * Les fiches de police repassent en `draft` si un champ obligatoire est vidé (`BUG-029`).
5. **Relevés Bancaires (Banque Chaabi)** :
   * Déduplication stricte par empreinte hash insensible au signe du montant (`BUG-022`).
   * Prise en charge des montants avec devise (`1 250 MAD`), parenthèses comptables `(500,00)` et dates avec slashs `DD/MM/YYYY` (`BUG-020`, `BUG-021`).

---

## 5. Commandes de Test & Validation

```bash
# 1. Exécuter l'intégralité des 508 tests
npm test

# 2. Exécuter uniquement les 278 workflows multi-étapes
npm test tests/workflows

# 3. Exécuter un workflow spécifique
npx vitest run tests/workflows/caisse-stoniz-lifecycle-workflow.test.ts

# 4. Vérifier la stricte compilation TypeScript (0 erreur attendue)
npm run typecheck

# 5. Vérifier la conformité du linter (0 avertissement attendu)
npm run lint
```

---

## 6. Guide pour Ajouter un Nouveau Test Multi-Étapes

Pour ajouter un nouveau workflow dans `tests/workflows/` :

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { createMockSupabaseClient } from '../helpers/mock-db';

describe('Multi-Step Workflow: [Nom du Métier]', () => {
  let db: any;

  beforeEach(() => {
    db = createMockSupabaseClient({
      // Initialiser tables requises
    });
  });

  it('Step 1: Initiation nominale', async () => {
    // Étape 1
  });

  it('Step 2: Transition d état et validation', async () => {
    // Étape 2
  });

  it('Step 3: Tentative invalide (garde fou de sécurité)', async () => {
    // Vérifier le rejet conforme aux règles d'immutabilité
  });
});
```

---

## 7. Registre des Bugs

Consulter [`BUG_LOG.md`](file:///C:/Users/33780/Desktop/agents/DEEPSEEK/NEW%20E2E%20TESTS/erptests/BUG_LOG.md) pour la liste exhaustive et détaillée des 47 bugs diagnostiqués et corrigés.

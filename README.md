# Artextyl — Import Production Tool

Application web pour extraire automatiquement les données des documents fournisseurs
(PFI, CI, BL, PL, Weight Note) et remplir le tableau de production Excel (onglet IM5).

---

## Structure du projet

```
artextyl-prod-tool/
├── api/
│   ├── extract.js       ← Appelle Claude AI pour extraire les données des docs
│   └── fill-excel.js    ← Remplit l'onglet IM5 de votre Excel
├── public/
│   └── index.html       ← Interface utilisateur (upload, résultats, téléchargement)
├── package.json
├── vercel.json
└── README.md
```

---

## Déploiement sur Vercel (5 minutes)

### 1. Créer un compte Vercel
→ https://vercel.com/signup (gratuit)

### 2. Installer Vercel CLI
```bash
npm install -g vercel
```

### 3. Déployer
```bash
cd artextyl-prod-tool
vercel
```
Suivre les instructions (nom du projet, etc.)

### 4. Ajouter la clé API Anthropic
Dans le dashboard Vercel → Settings → Environment Variables :
```
ANTHROPIC_API_KEY = sk-ant-xxxxxxxxxxxxxxxx
```

Obtenir votre clé : https://console.anthropic.com/

### 5. Re-déployer après avoir ajouté la clé
```bash
vercel --prod
```

---

## Logique de remplissage IM5

### Colonnes de l'onglet IM5 (indices originaux, votre fichier)

| Colonne | Index | Contenu | Rempli par |
|---------|-------|---------|------------|
| I | 9 | REFERENCE | Manuel |
| J | 10 | GENDER | Manuel |
| L | 12 | COLOR | Manuel |
| M | 13 | REF.COMPLET | Formule auto |
| N | 14 | CODE FRS | Manuel |
| O | 15 | QTY | Manuel |
| *votre col* | *ex: P=16* | QTY PFI | IA — PFI |
| U | 21 | PRICE FINAL | IA — PFI |
| V | 22 | TOTAUX | IA — PFI |
| W | 23 | REGLEMENT | IA — PFI |
| X | 24 | N° PI | Manuel |
| Y | 25 | ETD | IA — PFI |
| AI | 35 | COMMERCIAL INVOICE | IA — CI |
| AJ | 36 | BOOKING | Manuel |
| AK | 37 | ETD FINAL | IA — CI |
| AL | 38 | ETA | IA — CI |
| AM | 39 | QTY FINAL | IA — CI |
| AO | 41 | BALANCE | IA — CI |
| AP | 42 | BALANCE2 | IA — CI |
| AQ | 43 | MONTANT FINAL | IA — CI |

### Logique de matching

**Depuis PFI :**
1. Chercher dans IM5 les lignes où `N° PI` (col X) = numéro PFI extrait
2. ET `CODE FRS` (col N) correspond au fournisseur
3. Comparer `REFERENCE` (col I) avec le `ref_key` de chaque article du PFI
4. → Remplir : QTY PFI, PRICE FINAL, TOTAUX, REGLEMENT, ETD

**Depuis CI/BL/PL :**
1. Chercher dans IM5 les lignes où `BOOKING` (col AJ) = numéro booking extrait
   *(ce numéro est déjà saisi manuellement par vous)*
2. Comparer `REFERENCE` (col I) avec le `ref_key` de chaque article du CI
3. → Remplir : QTY FINAL, N° CI, ETD FINAL, ETA, BALANCE, BALANCE2, MONTANT FINAL

---

## Colonne QTY PFI

Si vous avez ajouté une colonne QTY PFI dans votre Excel (ex: après la colonne O=QTY),
indiquez son numéro dans l'interface (champ "Colonne QTY PFI").

Exemple : si vous l'avez insérée en colonne P → entrez **16**

---

## Formats de documents acceptés

- **PDF** : recommandé pour PFI, CI, BL
- **Excel (.xlsx)** : tous vos documents fournisseurs
- **Image (.jpg, .png)** : pour les BL scannés ou photos

---

## Utilisation au quotidien

1. **Étape 1 — PFI** :
   - Chargez le PFI reçu du fournisseur
   - Chargez votre tableau Excel PROD
   - Cliquez "Extraire" → téléchargez l'Excel mis à jour

2. **Étape 2 — CI/BL/PL** (après réception des docs de shipping) :
   - Chargez CI, PL, BL, Weight Note
   - Chargez l'Excel mis à jour (depuis l'étape 1)
   - Cliquez "Extraire" → téléchargez l'Excel final

---

## Support

En cas de problème, vérifiez :
- La clé API Anthropic est correctement configurée dans Vercel
- Le fichier Excel contient bien un onglet nommé exactement **"IM5"**
- Le N° PI ou BOOKING est déjà saisi dans Excel avant de lancer l'extraction

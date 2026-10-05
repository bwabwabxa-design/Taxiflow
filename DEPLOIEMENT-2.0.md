# TaxiFlow 2.0 — serveur commun et applications iOS

Serveur : https://taxiflow-54s9.onrender.com

- Client : `/reservation`
- Centrale : `/centrale`
- Chauffeur : `/chauffeur`
- API : `/api/v1` ; documentation : `/docs` ; configuration publique : `/api/v1/config`

Les deux IPA chargent leur interface depuis ce serveur HTTPS. Une mise à jour web ne nécessite plus de réinstaller les IPA. Internet est indispensable ; aucune course fictive n'est transmise. Le mode démo reste explicitement indiqué et séparé des données serveur.

## Render existant

Conserver les variables administrateur existantes. Build :

```
python -m zipfile -e TaxiFlow-admin-ready-fixed.zip . && pip install -r TaxiFlow-admin-ready/api/requirements.txt
```

Démarrage :

```
cd TaxiFlow-admin-ready && uvicorn api.main:app --host 0.0.0.0 --port $PORT
```

Variables non secrètes : `PUBLIC_BASE_URL=https://taxiflow-54s9.onrender.com`, `PUBLIC_BOOKING_ENABLED=1`.

Attention : `TAXIFLOW_DB=/tmp/taxiflow.sqlite3` sur Render Free est temporaire. Un redéploiement ou une recréation de l'instance peut effacer les comptes et les courses. L'administrateur initial est recréé depuis les variables existantes. Pour exploiter réellement une flotte, prévoir une base durable avant de saisir des données importantes. Aucun service payant n'est ajouté automatiquement.

## Connexion Google et Apple

Les boutons sont intégrés. Ils ne deviennent utilisables que lorsque le fournisseur est configuré. Aucun identifiant officiel ni certificat Apple n'est inclus dans ces fichiers. Les clés doivent être saisies uniquement dans les variables du serveur Render, jamais dans GitHub ni dans l'IPA.

Google : créer un client OAuth de type Application Web, configurer l'écran de consentement et le public autorisé, puis définir `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`.

URI de retour Google exacte :
`https://taxiflow-54s9.onrender.com/api/v1/auth/oauth/google/callback`

Apple : configurer Sign in with Apple dans le compte développeur Apple, avec un Services ID associé au bon App ID, le domaine `taxiflow-54s9.onrender.com` et l'URL de retour ci-dessous. Définir `APPLE_CLIENT_ID` (Services ID), `APPLE_TEAM_ID`, `APPLE_KEY_ID`, `APPLE_PRIVATE_KEY` (contenu de la clé .p8 ; les retours `\n` sont acceptés). Ces démarches peuvent nécessiter une adhésion Apple et une application éligible ; LiveContainer ne remplace pas les prérequis Apple.

URI de retour Apple exacte :
`https://taxiflow-54s9.onrender.com/api/v1/auth/oauth/apple/callback`

Première association : se connecter avec l'e-mail et le mot de passe TaxiFlow, ouvrir Réglages, puis Associer Google ou Associer Apple. Une identité externe ne devient jamais administrateur en fonction de son seul e-mail. Les nouveaux chauffeurs utilisent d'abord Créer mon compte chauffeur pour fournir leurs informations de véhicule.

Sur LiveContainer, le navigateur système s'ouvre : saisir le code à 6 chiffres affiché dans TaxiFlow, valider, puis revenir dans l'application. L'application récupère une session à usage unique. Aucun schéma d'URL propre à l'IPA ni droit Sign in with Apple natif n'est requis par ce mécanisme web. Les flux expirent après 10 minutes. Les signatures, issuer, audience, nonce, état et preuve de récupération sont vérifiés côté serveur.

## IPA / LiveContainer

Importer `TaxiFlow-Centrale-2.0.0.ipa` et `TaxiFlow-Chauffeur-2.0.0.ipa` dans LiveContainer. ARM64, iOS 15 minimum. Les bundles ont une signature ad hoc préparée pour la resignature ; aucune signature de distribution Apple n'est fournie. LiveContainer doit assurer sa propre resignature/configuration. Conserver les anciennes applications jusqu'à validation des nouvelles.

La localisation nécessite le consentement iOS et s'arrête en arrière-plan. Les exports PDF utilisent le partage iOS. Les pages externes et OAuth s'ouvrent dans le navigateur système. L'écran de secours permet de retenter une connexion serveur.

Les sources actuelles iOS et les scripts de compilation sont dans `mobile/`. Exemple Linux :

```
python mobile/tools/build_ipa.py --edition driver --site-url https://taxiflow-54s9.onrender.com --zig /chemin/zig --ld64 /chemin/ld64.lld --zsign /chemin/zsign --output TaxiFlow-Chauffeur-2.0.0.ipa
```

## Vérification

27 tests API automatisés réussis : authentification, rôles, inscriptions chauffeur, concurrence d'attribution, courses, réservations publiques, idempotence, paiement simulé, OAuth et validation cryptographique (dont nonce/audience/signature incorrects).

Les deux IPA passent le contrôle du binaire Mach-O ARM64, alignement, Info.plist, ressources et empreintes de signature. Leur lancement effectif dans LiveContainer et les connexions réelles Google/Apple restent à vérifier sur iPhone et avec les identifiants officiels configurés. Aucun test n'affirme le contraire.

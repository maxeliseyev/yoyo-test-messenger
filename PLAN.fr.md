# Plan de réalisation

> Version française de [PLAN.md](PLAN.md), pour la [SPEC.fr.md](SPEC.fr.md).

Chaque étape se termine par un résultat vérifiable, pas par une liste de tâches faites. L'ordre est choisi pour vérifier en premier les points les plus risqués : la crypto en WASM, le push sur iOS, les garanties de livraison.

| Étape | Contenu | Terminé quand |
|---|---|---|
| **M0. Socle** (1 sem.) | Monorepo : `crates/yoyo-proto`, `crates/yoyo-crypto`, `crates/yoyo-server`, `web/`. CI : fmt, clippy, tests, build WASM, squelette Playwright. Docker Compose : Postgres, Redis, MinIO. | `docker compose up` + `pnpm dev` ouvrent un client vide qui reçoit `pong` du serveur par WebSocket. |
| **M1. Cœur crypto** (2 sem.) | `yoyo-crypto` : Argon2id, HKDF, IK/EK, signature des EK, enveloppe (§5.5), STREAM pour les médias, code de sécurité. Liaison `wasm-bindgen`, pont vers le Worker. | Les vecteurs RFC 9180/9106 passent en natif et dans trois navigateurs ; une enveloppe chiffrée dans le navigateur se déchiffre dans un test natif et inversement ; `cargo-mutants` sur la crate sans mutant survivant dans le chiffrement et la vérification de signature. |
| **M2. Comptes** (2 sem.) | Inscription avec phrase seed et preuve de travail (§6.7), connexion avec remplacement d'appareil, `/auth/params` avec faux sel, jetons, rotation du refresh, deux copies du trousseau avec `If-Match`, changement de mot de passe, récupération par phrase (§6.6), réémission de la phrase, journal des connexions, « se déconnecter partout ». | Playwright : inscription dans le contexte A → connexion dans le contexte B → A fermé en `4003`, clés identiques dans B ; récupération par phrase dans C → le nouveau mot de passe fonctionne, l'ancien non, `IK` inchangée ; les réponses de `/auth/params` et `recovery/start` sont indiscernables, en format et en temps, entre identifiant existant et inexistant. |
| **M3. Livraison** (2 sem.) | Conversations, annuaire de clés, WebSocket (§7), réception Lua avec déduplication, file par compte, curseur, `ack` cumulatif, `delivered`, tâche TTL, contre-pression. | Tests du §9.4 au vert ; test de chaos : kill -9 du serveur et de Redis sous charge → 0 `accepted` perdu, 0 doublon dans l'UI ; charge : 20 000 WS, 1 000 messages/s, p95 < 300 ms. |
| **M4. Client** (3 sem.) | Écrans du §12.4, stores MobX, IndexedDB avec `vault_key`, outbox, démarrage hors ligne de la PWA, texte, images (compression, EXIF, aperçu), réactions, modification et suppression (§11), « lu » désactivable, vérification des clés. | Le scénario E2E du §17 passe dans Chrome, Firefox, Safari ; invariant N4 : la phrase témoin est absente des dumps de Postgres, de Redis et du bucket. |
| **M5. Push** (1,5 sem.) | Spike du §13.5 sur de vrais appareils **en début d'étape**, puis abonnements, VAPID, envoi avec nouvelles tentatives, déchiffrement dans le SW, repli sur peek puis sur notification générique. | Le tableau du §13.5 est validé sur Android Chrome et sur PWA iOS 17/18 ; la part de notifications vides sur iOS est mesurée. Au-delà de 5 % : ADR pour passer à l'option D. |
| **M6. Exploitation** (1 sem.) | Métriques du §14.1, tableau de bord Grafana, alertes, canari, Sentry, déploiement sur tag, redémarrage planifié sans perte, sauvegardes. | Un redémarrage du serveur sous charge ne perd aucun message et les clients se reconnectent en 5 s ; le canari déclenche une alerte si la livraison s'arrête. |
| **M7. Audit** (1 sem.) | Revue interne du modèle de menace du §3, fuzzing du décodeur d'enveloppe (`cargo-fuzz`), vérification de la CSP, revue des dépendances (`cargo-deny`, `pnpm audit`). | Tous les constats de niveau high sont corrigés ; rapport dans `docs/audit/`. |

**Total v1 :** ~13,5 semaines pour un développeur avec Claude Code ; environ 9 avec backend et frontend en parallèle.

**Étape 2** (après la v1, spécification séparée) : ADR sur le modèle de clés par appareil → multi-appareil (file par appareil, synchronisation de ses propres appareils) → groupes à composition signée. Les réserves prévues dans le format sont listées au §16.1 de la spécification.

## Comment le travail est mené avec Claude Code

- Chaque étape commence par une note de tâche `docs/tasks/NNN-*.md` : objectif, critère de fin repris du tableau ci-dessus, journal de travail. L'agent complète le journal après chaque session ; la session suivante reprend donc exactement là où la précédente s'est arrêtée.
- Toute décision qui limite les choix futurs (D1–D13 et les suivantes) fait l'objet d'un ADR dans `docs/decisions/`, avec une section obligatoire « Comment font les autres » et un lien vers le code ou la spécification lus.
- Les modifications de `yoyo-crypto` et de `delivery/` répondent dans la description de la PR à trois questions : qu'apprend de nouveau le serveur, qu'une partie peut-elle cacher ou falsifier, où se déplace la frontière de confiance.
- Chaque nouveau test passe une vérification par mutation : si une mutation du code qu'il couvre ne le fait pas échouer, le test est renforcé.
- Les spikes (WASM dans le SW de Safari, push sur iOS) sont faits avant l'implémentation principale de l'étape ; leur résultat est consigné, chiffres à l'appui, dans la note de tâche.

Pour en savoir plus sur la méthode : [HOW_I_WORK.fr.md](HOW_I_WORK.fr.md).

# Konstruct — comment je construis un système avec Claude Code

> Version française de [HOW_I_WORK.md](HOW_I_WORK.md).

**Le projet.** Konstruct est une messagerie chiffrée de bout en bout, avec cryptographie post-quantique (ML-KEM dans la poignée de main et dans le ratchet), plusieurs appareils par compte, appels WebRTC et sa propre couche d'obfuscation du trafic (VEIL). Il compte neuf dépôts :

- un cœur commun en Rust, intégré via UniFFI au client iOS/macOS en SwiftUI et au client Android ;
- un client desktop en Rust ;
- le serveur et les relais ;
- les protocoles en protobuf ;
- une base de connaissances.

Presque tout le code est écrit en binôme avec Claude Code. Je fixe les tâches, je prends les décisions et je fusionne les changements dans le cœur et le serveur. L'agent conçoit, écrit, teste et documente.

## Pourquoi les prompts ne suffisent pas

L'agent commence chaque session sans mémoire. Le contexte du projet vit donc dans des fichiers que l'agent lit lui-même, et non dans mes messages. Il y a trois couches, et au-dessus d'elles des vérifications exécutables.

### 1. AGENTS.md dans chaque dépôt — les lois du projet

Douze règles, et derrière chacune un incident qui nous a déjà coûté de l'argent ou du temps. Par exemple :

- « D'abord la solution des autres » : avant d'inventer un mécanisme, lire comment Signal ou MLS l'ont fait ;
- « Le cœur décide, l'application exécute » ;
- « D'abord reproduire, ensuite corriger » ;
- « Un test qui ne peut pas échouer est pire que pas de test ».

Chaque modification du chemin de livraison ou de la cryptographie répond, dans le commit, à trois questions :

- qu'apprend de nouveau le serveur ;
- qu'une partie peut-elle cacher ou falsifier ;
- où se déplace la frontière de confiance.

### 2. Base de connaissances dans Obsidian — la mémoire du projet

Depuis janvier 2026 :

- 254 décisions d'architecture : ce qui a été choisi, quelles options ont été écartées, comment les autres font, avec un lien vers le code lu ;
- 280 notes de tâche avec le déroulé complet du travail ;
- un journal d'environ 1 900 entrées.

Un script refuse toute décision sans référence à la solution de quelqu'un d'autre. L'agent cherche dans la base avant de proposer une architecture, puis consigne le déroulé dans la note de tâche. C'est ce qui permet de mener des tâches de plusieurs jours sur plusieurs dépôts sans perdre le contexte.

### 3. Mémoire de l'agent — les leçons sur le processus lui-même

118 entrées sur les erreurs de l'agent et sur la façon de ne pas les répéter. Par exemple :

- ne jamais réécrire un historique git déjà publié ;
- ne pas se fier à un `cargo test` vert si le code Rust n'a pas réellement été recompilé ;
- une batterie de mutations a un jour « tué » des mutations qui, en réalité, n'étaient pas appliquées.

## Vérifications exécutables

Une règle que rien ne vérifie finit par ne plus être respectée. Les règles sont donc ancrées dans le code :

- test de mutation de chaque nouveau test : une batterie maison pour Swift, cargo-mutants pour Rust, des mutations manuelles pour Kotlin avec restauration du fichier par haché ;
- un seul script de vérifications pour la CI et le hook pre-push ;
- des bancs d'essai :
  - deux simulateurs ;
  - iOS ↔ Android ;
  - un banc de capture du trafic qui confirme l'absence d'identifiants stables, et qui est lui-même validé par un contrôle positif.

## À quoi ressemble un cycle

Un exemple récent : le transfert de la logique des appels dans le cœur commun.

1. L'agent a inventorié les trois implémentations (iOS, Android, client Rust) et trouvé cinq divergences.
2. Il a lu comment c'est fait dans Signal RingRTC et noté ce qui coïncide chez nous, ce qui nous est propre, et pourquoi.
3. Il a écrit la règle dans le cœur, l'a vérifiée par des tests et des mutations, puis a ouvert une PR. C'est moi qui fusionne le cœur.
4. Il a ensuite migré les trois clients vers cette règle en trois PR, chacune avec build, tests et mutations. En chemin, deux vrais défauts du client Rust ont été trouvés et corrigés.
5. Chaque étape est consignée dans la note de tâche.

Mon rôle dans ce cycle : les décisions qu'on ne peut pas déduire du code — compromis produit, modèle de menace, ordre des travaux — ainsi que la revue et la fusion aux endroits critiques.

## Ce qui se transpose chez YoYo Mobility

Vous avez ~45 services et une équipe qui a besoin d'un standard commun de travail avec Claude. Le processus n'est pas lié à la messagerie et se transpose tel quel :

- des lois du projet tirées des incidents ;
- des décisions d'architecture avec l'analyse des approches des autres ;
- des notes de tâche grâce auxquelles l'agent reprend le travail le lendemain ;
- la vérification des tests par mutation ;
- des commits qui répondent aux questions sur les frontières de confiance.

Il répond aux points de l'offre sur les spécifications, plans, revues et tests avec Claude, et sur la documentation des décisions d'architecture.

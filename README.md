# YoYo Messenger — exercice technique

**FR** · [RU](#ru)

Conception d'une messagerie web (backend + frontend) chiffrée de bout en bout. L'implémentation n'est pas demandée.

| Fichier | Contenu |
|---|---|
| [SPEC.fr.md](SPEC.fr.md) | Spécification du service : exigences, modèle de menace, architecture, cryptographie, protocole, stockage, client, push, exploitation, décisions et questions ouvertes |
| [PLAN.fr.md](PLAN.fr.md) | Plan de réalisation par étapes, avec critères de fin |
| [PROMPT.md](PROMPT.md) | Les prompts qui ont servi à générer et affiner la spécification, dans l'ordre et sans retouche. Ils sont en russe : la spécification a été rédigée en russe ([SPEC.md](SPEC.md)), puis traduite |
| [HOW_I_WORK.fr.md](HOW_I_WORK.fr.md) | Bonus : la méthode avec laquelle je mène un grand projet avec Claude Code |

## Le système en bref

- **Périmètre v1 :** conversations privées, un seul appareil actif par compte, texte, images, réactions, modification et suppression. Groupes et multi-appareil : étape 2.
- **Client :** Next.js (build statique) + React + MobX, PWA. Cryptographie et protocole en Rust compilé en WASM, exécutés dans un Web Worker.
- **Serveur :** un seul processus Rust (Axum + Tokio). REST pour les comptes et les clés, WebSocket pour les messages, Web Push pour les notifications.
- **Données :** PostgreSQL (Supabase) pour les comptes, les clés et les conversations ; Redis pour les files de livraison ; S3 pour les médias chiffrés.
- **E2E :** HPKE (RFC 9180) vers une clé d'époque signée de l'interlocuteur, renouvelée tous les 7 jours. Les clés du compte passent d'un appareil à l'autre via une sauvegarde chiffrée par une clé dérivée du mot de passe (Argon2id) ; en cas d'oubli du mot de passe, une phrase seed de 12 mots rend l'accès et les clés. Le serveur ne voit ni le mot de passe, ni le contenu, ni le type des messages.
- **Livraison :** idempotente (`msg_id` généré par le client), écriture atomique dans la file, at-least-once jusqu'à l'appareil avec déduplication côté client ; rien n'est acquitté avant d'être écrit sur disque.

---

<a id="ru"></a>

## RU

Проект веб-мессенджера (backend + frontend) со сквозным шифрованием. Реализация не требуется.

| Файл | Что внутри |
|---|---|
| [SPEC.md](SPEC.md) | Спецификация сервиса: требования, модель угроз, архитектура, криптография, протокол, хранилища, клиент, push, эксплуатация, решения и открытые вопросы |
| [PLAN.md](PLAN.md) | План реализации по этапам с критериями готовности |
| [PROMPT.md](PROMPT.md) | Промпты, которыми генерировалась и уточнялась спецификация, в порядке использования, без правок |
| [HOW_I_WORK.md](HOW_I_WORK.md) | Бонус: метод, которым я веду большой проект вместе с Claude Code |

### Коротко о системе

- **Объём v1:** личные диалоги, одно активное устройство на аккаунт, текст, изображения, реакции, правка и удаление. Группы и несколько устройств — этап 2.
- **Клиент:** Next.js (статическая сборка) + React + MobX, PWA. Криптография и протокол — Rust, скомпилированный в WASM, работает в Web Worker.
- **Сервер:** один процесс на Rust (Axum + Tokio). REST для аккаунтов и ключей, WebSocket для сообщений, Web Push для уведомлений.
- **Данные:** PostgreSQL (Supabase) — аккаунты, ключи, диалоги; Redis — очереди доставки; S3 — зашифрованные медиа.
- **E2E:** HPKE (RFC 9180) на подписанный эпохальный ключ собеседника, ротация раз в 7 дней. Ключи аккаунта переносятся между устройствами в резервной копии, зашифрованной ключом из пароля (Argon2id); при забытом пароле доступ и ключи возвращает seed-фраза из 12 слов. Сервер не видит ни пароль, ни содержимое, ни тип сообщения.
- **Доставка:** идемпотентная (`msg_id` от клиента), атомарная запись в очередь, at-least-once до устройства с дедупликацией на клиенте, ничего не подтверждается до записи на диск.

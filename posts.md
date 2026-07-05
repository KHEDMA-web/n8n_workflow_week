# 10 Posts LinkedIn — Prêts à publier

> Posts indépendants, style listicle "5 trucs pour...".
> Format testé : accroche forte → liste numérotée → CTA court.

---

## POST 1 — Sécurité Node.js

**5 erreurs de sécurité que font 90% des devs Node.js (et comment les corriger en 10 min)**

J'ai audité des dizaines d'APIs Node.js.
Les mêmes failles reviennent encore et encore.

Voilà les 5 plus fréquentes — et comment les corriger maintenant :

**1. Aucun rate limiting sur les routes sensibles**
N'importe qui peut spammer ton `/login` 10 000 fois.
→ `express-rate-limit` ou `@fastify/rate-limit` : 3 lignes, problème réglé.

**2. Les headers HTTP exposent ta stack**
`X-Powered-By: Express` dit à l'attaquant exactement quoi cibler.
→ `helmet()` en un import supprime ça + ajoute 11 autres protections.

**3. Les inputs ne sont jamais validés côté serveur**
"Le front valide déjà." — oui, mais Postman aussi existe.
→ Zod : `z.string().email().max(255)` sur chaque champ. Pas de runtime error, type inféré automatiquement.

**4. Les JWTs ne sont jamais révoqués**
Un token volé est valide jusqu'à expiration. Parfois 30 jours.
→ Access token court (15 min) + refresh token en base avec blacklist sur logout.

**5. Les erreurs exposent des infos internes**
`stack trace` en prod = cadeau pour un attaquant.
→ Un middleware global qui catch tout et renvoie juste `{ error: "Internal server error" }` en prod.

Lequel tu n'avais pas encore ? ↓

---

## POST 2 — Performance API

**5 choses qui ralentissent ton API Node.js (et que tu peux corriger aujourd'hui)**

Ton API répond en 800ms.
Elle pourrait répondre en 80ms.
Voilà ce qui la ralentit :

**1. Tu bloques la boucle d'événements avec des tâches lourdes**
Resize d'image, parsing CSV, envoi d'email = tout ça dans la route = API bloquée pour tout le monde.
→ BullMQ : envoie ces tâches dans une queue Redis, ta route répond en < 10ms.

**2. Tu requêtes la DB à chaque appel pour des données qui ne changent pas**
`SELECT * FROM plans` à chaque requête, 200 fois par seconde.
→ Redis cache + `stale-while-revalidate` : 1 requête toutes les 60 secondes.

**3. Tu sers des images non compressées**
Une image de 2MB en original vs 180KB compressée → ton LCP s'effondre.
→ Sharp.js : resize + WebP + compression en 5 lignes côté serveur.

**4. Tu utilises Express**
Ce n'est pas une critique — c'est un fait de benchmark.
Fastify fait 3x plus de req/s sur les mêmes routes, même syntaxe (plugins au lieu de middlewares).

**5. Tu n'as pas de connection pooling sur ta DB**
Chaque requête ouvre et ferme une connexion. En charge : catastrophe.
→ `pg-pool` ou `drizzle` avec pool configuré : max 10 connexions, recyclage automatique.

Quel gain tu as observé en appliquant l'un de ces points ? ↓

---

## POST 3 — Remplacer ESLint + Prettier

**J'ai supprimé ESLint et Prettier de tous mes projets. Voilà par quoi je les ai remplacés.**

Non, ce n'est pas clickbait.

Biome fait exactement la même chose.
En une seule dépendance.
50 à 100x plus vite.

**5 raisons de passer à Biome maintenant :**

**1. Une seule commande pour tout**
`biome check --apply .` → lint + format + auto-fix en une passe.
Fini les conflits entre ESLint et Prettier qui se battent sur les virgules.

**2. C'est écrit en Rust**
Sur un projet de 500 fichiers : ESLint + Prettier ≈ 8 secondes. Biome ≈ 0.3 secondes.
En CI sur 200 commits par jour, ça compte.

**3. Zéro config pour démarrer**
`biome init` → un fichier `biome.json` minimal. Ça marche.
Pas de 47 plugins à installer et à maintenir.

**4. Même règles que ce que tu connaissais**
Biome supporte la quasi-totalité des règles ESLint populaires (unicorn, typescript, a11y...).
Migration : `biome migrate eslint` — il lit ton `.eslintrc` et convertit.

**5. Support TypeScript natif**
Pas de parser séparé, pas de `@typescript-eslint`. Tout est intégré, tout est rapide.

Migration depuis un projet existant : moins d'une heure.

Tu utilises encore ESLint + Prettier ? ↓

---

## POST 4 — Checklist avant mise en prod

**5 choses à faire avant de mettre une API en production (que la plupart des devs oublient)**

Ton code marche en local. Les tests passent.
Mais es-tu vraiment prêt pour la prod ?

Voilà ma checklist des 5 points non-négociables :

**1. Logger en JSON structuré**
`console.log("erreur")` en prod = inutilisable.
→ Pino : chaque log est un objet JSON avec timestamp, level, requestId. Parseable par Datadog, Loki, CloudWatch en un clic.

**2. Monitorer les erreurs avec contexte**
Savoir qu'il y a eu une erreur ne suffit pas. Il faut savoir qui, quoi, quand, comment.
→ Sentry : stack trace + user + breadcrumbs + version du déploiement. Gratuit jusqu'à 5000 erreurs/mois.

**3. Définir des limites de mémoire et timeout**
Un process Node.js sans limite peut bouffer tout un serveur en 2 heures.
→ `--max-old-space-size=512` + timeout sur chaque route (60s max, jamais infini).

**4. Gérer les signaux SIGTERM proprement**
Quand Kubernetes redémarre ton pod, il envoie SIGTERM. Si tu l'ignores → requêtes coupées en plein milieu.
→ `process.on('SIGTERM', () => server.close(() => process.exit(0)))` — 3 lignes qui sauvent des utilisateurs.

**5. Avoir un healthcheck qui teste vraiment l'état de l'app**
`GET /health` qui retourne 200 = inutile s'il ne teste pas la DB, Redis, les services critiques.
→ Checker la connexion DB, le ping Redis, et retourner `{ status: "ok", db: true, cache: true }`.

Tu en avais raté combien sur ta dernière mise en prod ? ↓

---

## POST 5 — Redis au-delà du cache

**Redis, c'est pas juste un cache. Voilà 5 usages qui vont changer ta façon de développer.**

La plupart des devs utilisent Redis pour une seule chose : le cache.
Ils passent à côté de 80% de sa valeur.

**1. Queues de tâches (BullMQ)**
Envoi d'email, génération PDF, traitement d'image → ne bloque plus jamais ta route.
BullMQ + Redis = queue persistante avec retry automatique, jobs prioritaires, workers parallèles.

**2. Sessions utilisateur**
Stocker les sessions en DB = lent. En mémoire = perdu au redémarrage.
Redis : rapide, persistant, TTL automatique. `ioredis` + `express-session` en 5 lignes.

**3. Rate limiting distribué**
Avec plusieurs instances de ton app, le rate limiting en mémoire est inutile (chaque instance compte séparément).
Redis centralise les compteurs : 1 limite cohérente pour toutes tes instances.

**4. Pub/Sub pour les événements temps réel**
Pas besoin de WebSocket complexe pour notifier d'autres services d'un événement.
`redis.publish('order:paid', payload)` → tous les subscribers reçoivent instantanément.

**5. Leaderboard et compteurs atomiques**
`INCR views:article:42` → compteur atomique, thread-safe, sans transaction DB.
`ZADD leaderboard score userId` → classement trié en O(log n).

Redis fait tout ça avec une seule dépendance : `ioredis`.

Tu utilisais Redis pour quoi avant de lire ça ? ↓

---

## POST 6 — JWT Best Practices

**5 erreurs JWT que j'ai faites (et que tu fais probablement aussi)**

J'ai implémenté l'auth JWT 3 fois avant de le faire correctement.
Voilà ce que j'aurais voulu savoir dès le début :

**1. Mettre l'access token en localStorage**
C'est la première chose que font les tutos YouTube. C'est aussi la première chose qu'un XSS vole.
→ Access token en mémoire (variable JS), refresh token en cookie `httpOnly + Secure + SameSite=Strict`.

**2. Un seul token avec une longue durée de vie**
"Je mets 30 jours pour que les utilisateurs restent connectés."
→ Token volé = accès 30 jours. Access token = 15 minutes. Refresh token = 7 jours, en base, révocable.

**3. Stocker des données sensibles dans le payload**
Le payload JWT est encodé en Base64, pas chiffré. N'importe qui peut le lire.
→ Stocker seulement : `userId`, `role`, `iat`, `exp`. Jamais email, mot de passe, données perso.

**4. Ne jamais invalider les tokens**
Logout → token toujours valide jusqu'à expiration. Mot de passe changé → ancien token toujours valide.
→ Blacklist Redis des refresh tokens révoqués. Petite table, TTL automatique.

**5. Ne pas vérifier l'algorithme**
L'algorithme `none` est valide dans certaines librairies JWT mal configurées.
→ Toujours spécifier explicitement l'algo attendu : `jwt.verify(token, secret, { algorithms: ['HS256'] })`.

Laquelle de ces erreurs tu as déjà faite ? ↓

---

## POST 7 — Monitoring en prod

**5 outils de monitoring qui changent tout quand ton app est en production**

Sans monitoring, tu découvres les bugs quand tes utilisateurs te les signalent.
Avec ces 5 outils, tu les vois avant eux.

**1. Sentry — pour les erreurs avec contexte**
Pas juste un message d'erreur. La stack trace complète, l'utilisateur concerné, les actions qui ont précédé (breadcrumbs), la version déployée.
Gratuit jusqu'à 5000 events/mois. Setup en 2 minutes.

**2. Pino + Loki + Grafana — pour les logs structurés**
Pino écrit du JSON. Loki indexe. Grafana visualise.
Chercher "toutes les erreurs 500 de l'utilisateur X dans les 24 dernières heures" en 10 secondes.

**3. Prometheus + Grafana — pour les métriques**
CPU, mémoire, req/s, latence p95/p99, erreurs par route.
`prom-client` pour Node.js : expose `/metrics`, Prometheus scrape, Grafana affiche.

**4. Checkly ou UptimeRobot — pour la disponibilité**
Ton API est-elle encore en vie ? Un test toutes les 60 secondes depuis 5 régions du monde.
Alert SMS/Slack si une route dépasse 2 secondes ou renvoie une erreur.

**5. OpenTelemetry — pour tracer les requêtes distribuées**
Une requête passe par ton API → appel DB → appel Redis → service externe. Où est la lenteur ?
OpenTelemetry trace chaque étape avec durée et contexte. Compatible Jaeger, Tempo, Datadog.

Ces 4 premiers sont gratuits pour commencer. Pas d'excuse pour déployer sans monitoring.

Lequel tu as déjà en place ? ↓

---

## POST 8 — Drizzle ORM

**J'ai remplacé Prisma par Drizzle. Voilà pourquoi je ne reviendrai pas en arrière.**

Prisma m'a sauvé des heures au début.
Puis il m'a coûté des jours en prod.

**5 raisons pour lesquelles Drizzle est mieux pour les projets qui grandissent :**

**1. Les migrations sont du SQL lisible**
Avec Prisma : un fichier `.prisma` magique que tu ne comprends pas vraiment.
Avec Drizzle : `drizzle-kit generate` → du SQL pur. Tu sais exactement ce qui va s'exécuter en prod.

**2. Zéro overhead de requête**
Drizzle génère exactement le SQL que tu lui demandes. Pas de sur-fetching, pas de N+1 caché.
Prisma génère parfois des jointures surprenantes que tu découvres en prod à 3h du matin.

**3. Typesafety à 100% sans magie**
Ton schéma Drizzle est ton type TypeScript. Pas de client généré, pas de `prisma generate` à relancer après chaque modif.

**4. Aucun client Prisma à initialiser**
En serverless / Edge functions, Prisma a des problèmes de cold start et de connexions.
Drizzle : juste un objet `db` léger, parfait pour Vercel Edge, Cloudflare Workers.

**5. Tu gardes le contrôle du SQL**
Quand tu veux faire une requête complexe, Drizzle te laisse écrire du SQL brut avec `sql\`...\`` tout en gardant la typesafety. Avec Prisma, tu dois souvent `queryRaw` et perdre le typage.

Migration depuis Prisma : `drizzle-kit introspect` lit ta DB existante et génère le schéma.

Tu es encore sur Prisma ? Qu'est-ce qui te retient ? ↓

---

## POST 9 — Claude Fable 5

**5 choses que Claude Fable 5 fait que GPT ne fait pas (ou pas aussi bien)**

Je ne fais pas du fan service.
J'ai testé les deux sur des cas réels de développement.
Voilà les différences qui comptent vraiment :

**1. Fenêtre de contexte de 1 million de tokens**
Toute ta codebase dans un seul prompt. Pas de découpage, pas de résumé.
J'ai analysé un monorepo de 80 000 lignes en une seule passe — impossible avec GPT-4o (128K).

**2. 128 000 tokens en output**
Générer un document complet, un rapport d'audit entier, un schéma DB avec toutes ses migrations.
En un appel. Sans troncature.

**3. Le thinking adaptatif est natif et transparent**
Fable 5 raisonne en profondeur sur les problèmes complexes sans que tu aies à le demander.
Tu vois un résumé du raisonnement — pas une boîte noire.

**4. Tool use avec raisonnement intermédiaire**
Fable 5 ne se contente pas d'appeler un outil. Il raisonne sur le résultat avant d'appeler le suivant.
Sur des agents complexes (10+ outils), la différence de qualité est massive.

**5. Managed Agents — des agents avec état et workspace persistant**
Pas juste un appel API. Un vrai agent avec un filesystem, des sessions, un contexte qui persiste entre les runs.
Parfait pour des tâches longues : audit de code, génération de docs, migrations automatisées.

Fable 5 n'est pas "le meilleur sur tout". Il est le meilleur sur les tâches longues, complexes, et les agents.

Tu l'as testé ? Quel était ton cas d'usage ? ↓

---

## POST 10 — BullMQ / Queues

**5 cas où tu DOIS utiliser une queue dans ton app (et arrêter de bloquer tes routes)**

Ton API met 3 secondes à répondre ?
Peut-être que tu fais trop de choses dans la route.

Voilà 5 cas où une queue (BullMQ + Redis) change tout :

**1. Envoi d'emails**
Envoyer un email via SMTP dans une route HTTP = attendre 500ms à 2s.
→ La route enqueue le job en < 5ms. Un worker envoie l'email en arrière-plan. L'utilisateur ne voit rien.

**2. Génération de PDF / rapports**
Générer un PDF avec Puppeteer peut prendre 5 à 15 secondes. En route HTTP = timeout garanti.
→ Job en queue, l'utilisateur reçoit un lien de téléchargement par email quand c'est prêt.

**3. Traitement d'images**
Upload → resize → compression → upload vers S3. Tout ça ne doit pas bloquer la réponse.
→ Tu réponds "upload reçu" en < 50ms. Le traitement se fait en arrière-plan avec Sharp.js.

**4. Webhooks sortants**
Ton app doit notifier des services externes quand un événement se passe.
Si l'externe est lent ou down → ta route attend.
→ Webhook en queue avec retry automatique (3 tentatives, backoff exponentiel).

**5. Synchronisation avec des APIs tierces**
Appeler Stripe, HubSpot, Notion dans une route user = dépendant de leur uptime.
→ Queue avec retry : si Stripe est down 2 minutes, le job sera retenté automatiquement.

BullMQ + `ioredis` + un worker process séparé. C'est tout.

Tu bloques encore tes routes sur ces tâches ? ↓

---

_Fichier généré le 2026-07-05. Consulter via `git show origin/_ideas:posts.md`_

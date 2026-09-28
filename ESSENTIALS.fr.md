# 24H by Webcup — L'essentiel pour les développeurs

Résumé du règlement officiel (« 24h by Webcup – The ultimate web development sprint »). Relisez la page officielle avant l'événement : chiffres et détails peuvent évoluer.

## Le format en un paragraphe

En 24 heures, chaque équipe développe une **application web fonctionnelle** (un vrai produit/service, pas un site vitrine). Le sujet est révélé au lancement avec des **fonctionnalités de base obligatoires**. Pendant les 24 heures, **de nouvelles fonctionnalités sont annoncées à intervalles réguliers** — volontairement trop nombreuses pour tout finir. Pour chacune, l'équipe choisit : la faire maintenant, plus tard, ou pas du tout. À la fin, l'équipe livre une application en ligne. Le jury l'évalue **dans les jours qui suivent, sans soutenance orale**, en la testant directement.

## Dates clés à retenir

| Quand | Ce qui se passe | Ce que vous devez faire |
|---|---|---|
| **J-7** | Ouverture des serveurs / hébergement (partenaire technique HODi) | Installer la stack, configurer le serveur, tester le déploiement. Le sujet reste secret : préparez un socle générique. |
| **J0 (H0)** | Lancement officiel : concept, thématique, fonctionnalités de base, règles | Tout lire, poser le modèle de données, déployer le squelette dans les 2 premières heures. |
| **H0 → H24** | Nouvelles fonctionnalités annoncées à intervalles réguliers | Trier chacune (maintenant / plus tard / impasse), garder l'application en ligne fonctionnelle. |
| **H+24** | Fin du développement — échéance ferme | Tout livré et en ligne. Rien après ne compte. |
| **J+1 → J+X** | Évaluation différée par le jury | Rien à faire : votre URL, vos accès, votre récap et votre vidéo parlent pour vous. Laissez l'app en ligne. |
| **J+X** | Résultats et remise des prix | Podium + distinctions complémentaires. |

## Livrables (minimum, à la clôture)

1. **URL de l'application** — version exploitable, en ligne.
2. **Accès jury** — comptes permettant de tester toute l'application, y compris les parties authentifiées (un par rôle).
3. **Récapitulatif des fonctionnalités implémentées** — aide le jury à repérer ce qui a été fait. Ne remplace pas les tests.
4. **Courte vidéo de démonstration** — vue d'ensemble de l'app, de sa logique et des fonctionnalités principales.

Seuls les éléments réellement remis dans les délais sont évalués.

## Comment vous êtes évalués

| Dimension | Ce que regarde le jury |
|---|---|
| **Fonctionnalités implémentées** | Présentes, fonctionnelles, réellement intégrées. Pertinence et finition comptent plus que la quantité. |
| **Qualité technique** | Cohérence de la structure, logique de développement, robustesse, qualité de l'intégration, bonne exploitation des données/API, sécurité applicative. |
| **Design et UX** | Interface claire, cohérente, utilisable : lisibilité, ergonomie, navigation, qualité graphique, expérience utilisateur. |
| **Cohérence d'ensemble** | Équilibre ambition / exécution, logique fonctionnelle, qualité globale du rendu, choix au service de l'application. |

Une grille commune est utilisée pour toutes les équipes. Le jury peut réunir des profils technique, fonctionnel, design/UX et **cybersécurité**.

## Prix

- Classement général : **1er, 2e, 3e prix** (projets les plus complets, cohérents et aboutis).
- Distinctions complémentaires possibles (mentions, certificats, badges) : qualité graphique, UX, qualité technique, fonctionnalité particulièrement réussie, originalité ou cohérence globale. On peut en obtenir une sans être sur le podium.

## Cybersécurité et robustesse

Ce n'est pas un concours de hacking, mais les applications sont testées face à des comportements malveillants simples et à des erreurs de conception fréquentes. **Les familles de vulnérabilités testées seront annoncées à l'avance.** À prévoir :

- authentification
- contrôle d'accès
- validation des entrées utilisateur
- protection des endpoints
- exposition involontaire de données
- brute force simple et mauvaise gestion des rôles

Certaines fonctionnalités annoncées pendant les 24h peuvent elles-mêmes être des fonctionnalités de sécurité.

## Technologies autorisées et IA

- Tout langage, framework, bibliothèque, outil de versioning/déploiement/automatisation et service externe, **tant que c'est compatible avec l'environnement fourni**. L'organisation peut préciser des contraintes techniques en amont.
- **L'IA est autorisée.** Mais l'IA seule ne suffit pas : les choix, la structure, l'intégration et la cohérence font la différence.

### IA dans votre application : OpenRouter (modèles gratuits)

| Limite | Valeur | Condition |
|---|---|---|
| Requêtes / minute | 20 | Toujours |
| Requêtes / jour | 50 | Compte neuf |
| Requêtes / jour | 1000 | Après un premier crédit ponctuel sur le compte |

- Les modèles gratuits se terminent par `:free`. Les tokens coûtent 0 ; la vraie limite est le **nombre de requêtes**.
- Fenêtres de contexte larges (256K à ~1M tokens) — pas une contrainte pour des fonctionnalités classiques.
- Le catalogue gratuit change sans prévenir : **gardez un modèle de secours**.
- Créer plusieurs comptes n'augmente **pas** le quota.
- Chaque appel compte, même les tests. En cas d'erreur, attendre quelques secondes et réessayer.
- **Ne jamais mettre la clé dans du code public** (GitHub, capture d'écran). Appeler uniquement depuis le serveur.
- Vérifier les limites à jour sur openrouter.ai/docs avant l'événement.

## À FAIRE

- Exploiter pleinement J-7 : stack installée, HTTPS, déploiement en une commande, auth + rôles, BDD + migrations, composants UI de base, testé sur le vrai serveur.
- Déployer dans les 2 premières heures, puis après chaque fonctionnalité terminée. L'app en ligne doit fonctionner en permanence.
- Finir complètement une fonctionnalité avant d'en commencer une autre.
- Prioriser : valeur pour le jury vs temps vs risque. Les fonctionnalités sécurité sont souvent peu coûteuses et valorisées.
- Tenir un `FEATURES.md` à jour tout du long (statut + raison des impasses) — il devient le récapitulatif.
- Faire tous les contrôles de sécurité côté serveur (auth, rôles, propriété des ressources, validation).
- Préparer des données de démo réalistes et des comptes jury pour chaque rôle.
- Geler les fonctionnalités vers H+20 ; ensuite finitions, sécurité, tests comme le jury, vidéo.
- Répartir les rôles : back-end/sécurité, front-end/UX, intégration/déploiement, produit/priorisation/livrables.

## À NE PAS FAIRE

- Montrer des fonctionnalités qui ne marchent pas vraiment (mocks, boutons morts, « bientôt disponible »).
- Viser la quantité : une accumulation de fonctionnalités sans intégration est pénalisée.
- Laisser l'app en ligne cassée, même « 5 minutes ».
- Protéger les pages admin uniquement en les cachant dans l'interface.
- Exposer des secrets, stack traces, `.env`, mots de passe ou données d'autres utilisateurs.
- Gaspiller le quota OpenRouter en tests.
- Déployer des changements risqués dans la dernière heure.
- Oublier un livrable : comptes jury, récap, vidéo.

## À vérifier / préparer avant l'événement

- [ ] Page officielle : règlement, dates, infos de l'organisateur local, contraintes techniques éventuelles.
- [ ] Liste des familles de vulnérabilités annoncées (publiée avant le concours).
- [ ] Caractéristiques du serveur (versions runtime, BDD, RAM, disque, domaine, HTTPS, ports).
- [ ] Compte OpenRouter créé, clé stockée en sécurité, limites et modèles gratuits vérifiés, modèle de secours choisi.
- [ ] Déploiement testé deux fois de zéro.
- [ ] Rôles de l'équipe définis, canal de communication prêt, plan snacks et sommeil.
- [ ] Outil d'enregistrement d'écran prêt pour la vidéo.

# Aperçu des fonctionnalités de Launchpad

Ce guide est un résumé concis du kit de démarrage Launchpad destiné aux acheteurs potentiels. Il résume ce qui est inclus, ce qui nécessite des services externes ou une configuration, et ce que le kit ne fournit pas automatiquement. Le guide complet des fonctionnalités et le code source du projet acheté restent les références d'implémentation faisant autorité.

## Ce que Launchpad inclut

Launchpad est une application SaaS modulaire construite sur une base de code partagée. Ses modules optionnels couvrent la facturation, la discussion IA, les organisations, la publication, le support, la communauté, les notifications, le marketing par e-mail, l'analyse, les parrainages, la publicité, le SEO, les intégrations et le stockage privé de fichiers. L'assistant de configuration vous permet de choisir quels modules optionnels conserver et une cible de déploiement. Les composants de base du compte, de la sécurité, des paramètres et de la base de données partagée restent en place.

Dans un checkout Git, la configuration utilise sparse-checkout afin que les fichiers suivis exclus restent dans l'historique Git. Dans un ZIP acheté, les fichiers des modules exclus sont supprimés du projet extrait ; ils ne peuvent pas être restaurés depuis l'historique Git car le ZIP ne contient pas les métadonnées du dépôt. Il s'agit d'une configuration unique, et non d'un outil de bascule de modules sur place. Effectuez une sauvegarde du ZIP extrait avant de le configurer si vous souhaitez peut-être récupérer la source exclue ultérieurement.

L'assistant écrit les modules sélectionnés et les préférences de déploiement dans `launchpad.config.json`, et les secrets des fournisseurs dans le fichier `.env` ignoré par Git. Il peut vérifier les identifiants des fournisseurs sélectionnés lorsque vous choisissez explicitement **Tester la connexion**. Il ne crée pas de comptes fournisseurs, ne déploie pas d'infrastructure, et ne configure pas les tableaux de bord externes. Vous devez toujours compléter et maintenir ces services vous-même.

La zone de plugins expose un ensemble de composants intégrés autorisés par liste blanche ; ce n'est pas un marché de paquets arbitraire et il ne télécharge ni n'exécute le code des plugins téléversés. Le service optionnel de relais d'événements peut être déployé séparément pour l'événement actuellement pris en charge de mise à jour du profil de compte. Ce n'est pas un registre de services général, un bus de messages, ou un système d'accusé de réception par consommateur.

## Comptes et sécurité

L'authentification prend en charge la connexion par e-mail et la connexion optionnelle via Google et GitHub. En développement, Mailpit peut recevoir les e-mails de connexion lorsqu'aucun fournisseur d'e-mail n'est configuré. Les environnements déployés ont besoin d'un fournisseur d'e-mail pour la connexion par e-mail et les e-mails transactionnels. Les pages et API d'administration appliquent les rôles côté serveur.

L'authentification multifactorielle TOTP optionnelle fonctionne avec les applications d'authentification compatibles et inclut des codes de récupération à usage unique. Les paramètres de sécurité incluent la gestion des sessions et l'activité d'audit. Le kit n'inclut pas la connexion basée sur mot de passe ni un flux de récupération de mot de passe.

La liste de contrôle d'intégration du compte couvre les paramètres de profil, la sécurité du compte et le tableau de bord ; elle est optionnelle et peut être revue.

## Facturation et accès payant

Le module de facturation prend en charge Stripe Checkout pour les abonnements configurés et les achats Lifetime à paiement unique, ainsi que le portail client Stripe. La vérification optionnelle des achats Apple App Store et Google Play peut fournir une autre source de facturation lorsque les produits requis, les identifiants, les comptes de boutique et l'intégration client native sont configurés. Les routes de boutique sont indisponibles jusqu'à ce que leurs paramètres soient fournis ; Stripe peut continuer à fonctionner indépendamment.

Le kit traite les événements de paiement et d'abonnement vérifiés, enregistre l'historique des paiements et gère les remboursements et les litiges. Il ne protège **pas** automatiquement chaque fonctionnalité de votre produit simplement parce que la facturation est activée. `paymentBlocked` est un signal de facturation, et non un paywall universel. Chaque fonctionnalité payante doit appliquer l'accès côté serveur ; masquer un bouton dans le navigateur n'est pas suffisant. Décidez quelles fonctionnalités sont payantes et ce que les clients restreints peuvent encore faire, puis implémentez et testez ces règles. Le kit n'inclut pas non plus un flux de travail d'administrateur prêt à l'emploi pour vérifier les paiements effectués en dehors de Stripe.

Les e-mails de facturation utilisent le fournisseur d'e-mail configuré, mais la facturation ne nécessite pas l'e-mail pour fonctionner. La livraison des e-mails peut être retardée, échouer ou être dupliquée après une réponse réseau incertaine ; ce n'est pas un système de livraison exactement une fois.

## E-mail, liste d'attente et marketing

Le transport d'e-mail partagé prend en charge Resend, SendGrid et SMTP. Il est utilisé pour l'authentification, les avis de facturation, les notifications, les infolettres et les messages marketing. Les identités de l'expéditeur et les identifiants du fournisseur doivent être configurés. La vérification SMTP de l'assistant de configuration s'authentifie sans envoyer de message.

La liste d'attente d'accès anticipé localisée nécessite un consentement explicite. Ses statuts sont gérés manuellement : inviter ou marquer quelqu'un comme ayant rejoint ne crée pas de compte, n'envoie pas d'invitation et n'accorde pas d'accès. Le consentement à la liste d'attente est distinct de l'abonnement à l'infolettre.

L'infolettre prend en charge les inscriptions basées sur le consentement, les liens de désabonnement et les campagnes texte brut créées par l'administrateur. L'automatisation marketing prend en charge de courtes séquences d'e-mails pour les nouveaux abonnés à l'infolettre. Ces outils ne sont pas un éditeur d'e-mails visuel, un système de segmentation d'audience, ou un service de conformité/juridique. Ils n'inscrivent pas automatiquement les entrées de la liste d'attente ni n'envoient de messages via des notifications push et des webhooks. Les campagnes planifiées et l'automatisation nécessitent également un planificateur externe ainsi qu'un fournisseur d'e-mail configuré.

## Assistant IA et organisations

L'assistant de tableau de bord optionnel se connecte à une API de texte compatible OpenAI configurée. Il stocke les conversations dans la base de données de l'application, applique les limites de requêtes du compte et prend en charge les conversations personnelles et organisationnelles. Il n'inclut pas de modèle, n'exécute pas d'outils et ne prend pas d'actions autonomes. Les invites et les réponses sont envoyées au fournisseur configuré et stockées par l'application ; configurez les paramètres de confidentialité et de rétention du fournisseur et divulguez ce traitement aux utilisateurs.

Les organisations fournissent des espaces de travail partagés, des rôles de membres et des invitations. L'autorisation est vérifiée par rapport à l'appartenance à l'organisation, mais il s'agit d'une location de base de données partagée, et non d'une base de données ou d'un schéma séparé pour chaque client. La facturation, les paramètres du compte, le support, les notifications, l'analyse, les infolettres et le contenu de publication global ne sont pas automatiquement limités à l'organisation.

## Publication, support et communauté

Des modules séparés fournissent un blog Markdown localisé, un flux de mises à jour de version et des pages CMS. Les administrateurs gèrent les brouillons et le contenu publié. Les pages publiques incluent des métadonnées localisées et la prise en charge des plans de site. Les outils de publication n'incluent pas la publication planifiée, la traduction automatique, le téléversement d'images ou un système de modération des commentaires.

Le module de support fournit des tickets clients et des réponses d'administrateur. Le module de communauté fournit un forum de discussion. L'annuaire curé permet aux administrateurs de gérer les entrées publiques. Ce sont des fonctionnalités de démarrage plutôt que des intégrations avec un service d'assistance hébergé ou un réseau social complet : les pièces jointes, l'automatisation d'affectation/SLA, les réactions au forum, les réponses acceptées et les profils d'utilisateur publics ne sont pas inclus.

## Notifications et données client

Le tableau de bord dispose d'une boîte de réception de notifications persistante et de canaux de livraison optionnels par e-mail, Web Push et webhook signé. Les notifications push nécessitent des clés VAPID et l'abonnement du navigateur ; les consommateurs de webhook doivent vérifier les signatures et dédupliquer les événements. Les tentatives automatiques pour les notifications, le marketing et le nettoyage du stockage nécessitent des tâches planifiées de manière externe.

Les paramètres du compte fournissent une exportation JSON des données de compte prises en charge et un lien séparé vers la gestion de la facturation. L'exportation exclut les identifiants, les sessions, les secrets de signature et d'autres jetons sensibles. C'est un téléchargement en flux, et non une sauvegarde complète de la base de données ou un service d'archivage en arrière-plan.

## Analyse, localisation et personnalisation

Le tableau de bord inclut des vues de compte et de facturation. L'analyse produit optionnelle collecte des événements soumis au consentement et fournit des rapports administratifs ; ils ne sont pas un remplacement pour une plateforme d'analyse tierce. L'attribution, le suivi marketing et les cookies de parrainage peuvent avoir des obligations de consentement dans votre déploiement. Launchpad n'inclut pas de plateforme de gestion du consentement.

L'interface prend en charge 18 locales : anglais, arabe, bengali, allemand, espagnol, français, hindi, indonésien, italien, japonais, coréen, néerlandais, polonais, portugais, russe, turc, vietnamien et chinois. L'anglais est le catalogue de secours et la langue de secours de l'assistant de configuration. Examinez les traductions et les textes juridiques pour votre audience avant le lancement.

Le constructeur de produit modifie un ensemble de sections d'application autorisées par liste blanche, la navigation, les textes localisés et les paramètres de marque. Il ne génère pas de code arbitraire, ne crée pas de routes d'application et ne modifie pas la logique métier. Le thème, le mouvement et d'autres préférences d'interface personnelles sont stockés dans le navigateur, et non synchronisés entre les appareils.

## Stockage, parrainages, publicité et SEO

Le stockage privé de fichiers optionnel utilise un stockage d'objets compatible S3. L'application valide les types et tailles de fichiers, utilise des URL de téléversement/téléchargement à durée limitée et vérifie la propriété des fichiers. Configurez un bucket privé et les identifiants requis. Le module ne scanne pas les fichiers pour détecter les logiciels malveillants, et la suppression du compte ne supprime pas automatiquement les objets distants.

Le module de parrainage suit les liens de partenaires et les commissions dérivées de Stripe. Les paiements sont effectués en dehors de Launchpad. Il ne calcule pas les commissions pour les achats App Store ou Google Play. L'attribution du parrainage utilise un cookie de 30 jours ; Launchpad n'a pas de gestionnaire de consentement intégré, donc évaluez et implémentez les exigences de consentement applicables.

Les outils de publicité organisent les informations de campagne et de compte et importent des rapports depuis CSV. Ils ne se connectent pas aux réseaux publicitaires pour créer ou gérer des campagnes en direct. Les fonctionnalités SEO fournissent des métadonnées localisées, des plans de site, des règles robots, des données structurées et un audit de la base de données de l'application ; ils ne sont pas un robot d'exploration, un suivi de classement, un outil de recherche de mots-clés ou un gestionnaire de redirections.

Le catalogue d'intégrations signale si la configuration attendue est présente. Il ne prouve pas que les identifiants fonctionnent, ne crée pas de comptes fournisseurs et ne garantit pas la livraison.

## Déploiement et responsabilités opérationnelles

Les déploiements nécessitent PostgreSQL et des variables d'environnement appropriées, y compris les connexions à la base de données et `AUTH_SECRET`. Configurez l'e-mail, Stripe, la facturation de boutique, le stockage d'objets, l'IA, le push et les tâches planifiées uniquement pour les fonctionnalités que vous activez. Les migrations de production doivent être appliquées en utilisant le processus de déploiement documenté ; maintenez les environnements d'aperçu sur une base de données isolée.

L'application inclut la validation côté serveur, les vérifications de rôle, les contrôles de sécurité et la journalisation d'audit, mais ce n'est pas un programme complet d'opérations de sécurité. Il ne fournit pas de WAF, de scan de logiciels malveillants, de surveillance de sécurité externe ou d'une politique de sécurité du contenu stricte. Configurez une entrée de confiance et des protections d'hébergement, examinez le modèle de menace et testez l'application déployée avant le lancement.

Pour la configuration détaillée, la propriété des données, les limites, les migrations et les procédures opérationnelles, consultez le guide complet des fonctionnalités fourni avec le projet.

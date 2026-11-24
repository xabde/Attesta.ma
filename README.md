# Attesta.ma — Backend

**Plateforme SaaS de gestion électronique et d’échange de documents commerciaux entre clients et entreprises.**

Attesta.ma vise à dématérialiser les échanges commerciaux : recherche d’entreprises, demandes de devis, création et transmission de documents, validation, signature et suivi.

Ce dépôt est consacré au **Backend Java Spring Boot**, conçu selon une architecture Microservices.

---

## Description du dépôt GitHub

> Backend d’Attesta.ma : plateforme SaaS de gestion et d’échange de documents commerciaux, développée avec Java, Spring Boot et une architecture Microservices. Chaque service possède sa propre base PostgreSQL et communique avec les autres services exclusivement via des API REST synchrones.

**Nom de dépôt suggéré :** `attesta-backend`

## Objectifs du Backend

- Gérer les comptes, les utilisateurs, les rôles et les permissions.
- Gérer les profils des entreprises et leurs activités.
- Permettre aux clients de rechercher des entreprises et de leur adresser des demandes.
- Gérer le cycle commercial : demandes, devis et commandes.
- Générer et gérer les documents commerciaux.
- Gérer les opérations de signature et de cachet, ainsi que leurs éléments de preuve.
- Gérer les notifications et la traçabilité des opérations.
- Garantir l’isolation des données entre les entreprises.

## Architecture technique

Attesta.ma adopte une architecture Microservices. Chaque service métier est autonome et possède sa propre base de données PostgreSQL.

![Architecture technique d'Attesta.ma](docs/architecture-technique.png)

### Principes architecturaux

1. **Un service, une base de données indépendante.** Chaque Microservice est seul responsable de ses données.
2. **Aucun accès direct aux bases des autres services.** Un service obtient les données externes via les API du service propriétaire.
3. **Communication interservices synchrone.** Les échanges métier entre services utilisent des API REST HTTP/JSON.
4. **Aucun broker de messages.** Kafka et RabbitMQ ne font pas partie de l’architecture retenue.
5. **API Gateway.** Spring Cloud Gateway constitue le point d’entrée des API.
6. **Service Discovery.** Netflix Eureka permet l’enregistrement et la découverte des instances de services.
7. **Sécurité.** Les règles d’authentification, d’autorisation et d’isolation des données sont définies pendant la conception, puis appliquées dans les services concernés.

> Eureka est un registre de découverte. Une requête ne traverse pas Eureka comme elle traverse une passerelle : le Gateway et les clients interservices utilisent le registre pour localiser les instances disponibles.

## Microservices métier

| Microservice | Responsabilités | Base de données |
|---|---|---|
| Identity Service | Utilisateurs, authentification, rôles et permissions | `identity_db` |
| Company Service | Entreprises, profils, activités et informations professionnelles | `company_db` |
| Commercial Service | Demandes de devis, devis et commandes | `commercial_db` |
| Document Service | Métadonnées, modèles et génération des documents PDF | `document_db` |
| Signature Service | Processus de signature, cachets et preuves associées | `signature_db` |
| Notification Service | Notifications et historique des envois | `notification_db` |
| Audit Service | Journalisation métier et traçabilité des opérations | `audit_db` |

La répartition exacte des responsabilités, des entités et des contrats REST sera détaillée et validée pendant la phase de conception.

## Technologies retenues

| Domaine | Technologie |
|---|---|
| Langage | Java |
| Framework Backend | Spring Boot |
| API Gateway | Spring Cloud Gateway |
| Service Discovery | Netflix Eureka |
| Communication interservices | REST API synchrone |
| Client REST interservices | Spring Cloud OpenFeign |
| Base de données | PostgreSQL |
| Accès aux données | Spring Data JPA |
| Sécurité | Spring Security |
| Authentification | JWT — stratégie à spécifier dans la conception |
| Validation | Jakarta Bean Validation |
| Documentation API | OpenAPI / Swagger |
| Tests | JUnit 5, Mockito et Spring Boot Test |

Les versions précises de Java, Spring Boot et Spring Cloud seront fixées dans le dossier de conception afin de garantir leur compatibilité.

## Communication entre les services

Les appels interservices sont réalisés exclusivement via REST.

Exemple : un client soumet une demande de devis à une entreprise.

1. La requête arrive au Backend via l’API Gateway.
2. Le Commercial Service vérifie les informations nécessaires auprès du Company Service via REST.
3. Le Commercial Service enregistre la demande dans `commercial_db`.
4. Le Commercial Service appelle le Notification Service via REST pour déclencher la notification.
5. Le Notification Service enregistre son propre historique dans `notification_db`.

Un service ne lit et ne modifie jamais directement les tables d’un autre service.

### Gestion des appels synchrones

L’architecture devra spécifier les timeouts, les stratégies de retry limitées, la gestion des erreurs, la prévention des appels en cascade et les mécanismes de cohérence métier nécessaires en cas d’échec partiel.

## Isolation des données

Chaque base appartient exclusivement à son Microservice. Les identifiants des ressources appartenant à d’autres services peuvent être conservés comme références, mais les relations SQL inter-bases et les accès directs aux bases externes sont exclus.

L’isolation multi-entreprises (multi-tenancy) devra être appliquée dans les services qui manipulent des données appartenant aux entreprises : chaque requête devra vérifier le contexte et les autorisations de l’utilisateur.

## Périmètre fonctionnel

Les familles de documents prévues comprennent notamment :

- Devis
- Factures
- Contrats
- Bons de commande
- Bons de livraison
- Reçus
- Autres documents commerciaux définis dans les règles métier

Les statuts, règles de validation, droits de signature, règles de modification et durées de conservation seront spécifiés dans le cahier des charges fonctionnel.

**Important :** l’intégration d’une signature électronique ayant une valeur juridique, le cachet électronique et les exigences réglementaires doivent faire l’objet d’une analyse dédiée. Une image de signature ou de cachet apposée sur un PDF ne constitue pas, à elle seule, une signature électronique qualifiée.

## Structure cible du dépôt

La structure exacte sera confirmée après validation des modules et des contrats entre services. La structure cible proposée est :

```text
attesta-backend/
├── infrastructure/
│   ├── api-gateway/
│   └── discovery-server/
├── services/
│   ├── identity-service/
│   ├── company-service/
│   ├── commercial-service/
│   ├── document-service/
│   ├── signature-service/
│   ├── notification-service/
│   └── audit-service/
├── docs/
│   ├── architecture-technique.png
│   ├── cahier-des-charges.md
│   ├── architecture-detaillee.md
│   ├── database-design.md
│   └── api-contracts/
├── .gitignore
└── README.md
```

Cette arborescence est une proposition de conception, pas l’affirmation que tous ces modules sont déjà implémentés.

## Méthodologie de réalisation

Le projet suit une démarche séquentielle de type Waterfall :

1. Analyse fonctionnelle et cahier des charges.
2. Conception de l’architecture Microservices.
3. Conception des bases de données.
4. Définition des contrats REST et des règles de sécurité.
5. Développement des services.
6. Intégration et tests interservices.
7. Validation fonctionnelle et technique.

Une phase est considérée comme validée lorsque ses livrables ont été examinés et acceptés. Toute modification ultérieure d’une décision validée doit être explicitement documentée.

## État du projet

**Phase actuelle :** cadrage et conception du Backend.

Les choix d’architecture présentés dans ce README constituent la base de travail. Les diagrammes détaillés, modèles de données, contrats API et règles métier seront ajoutés après validation des livrables correspondants.

## Licence

La licence du projet reste à définir.

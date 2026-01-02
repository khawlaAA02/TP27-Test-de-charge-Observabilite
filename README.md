# TP 27 : Test de charge & Observabilité : Concurrence + Verrou DB + Resilience4j + Actuator Metrics

## Objectif

Déployer un système de microservices composé de :

- book-service (3 instances)
- pricing-service
- MySQL

et démontrer :

- la gestion de concurrence
- la cohérence du stock
- la résilience avec Resilience4j
- le fallback en cas de panne

## Architecture :

Clients
   |
   |----> Book-Service-1 (8081)
   |----> Book-Service-2 (8083)
   |----> Book-Service-3 (8084)
                     |
                     |----> MySQL (stock + verrous)
                     |
                     |----> Pricing-Service (8082)

 ###  Preuve 1 — Stock final = 0

Commande :
      
      curl http://localhost:8081/api/books
<img width="1918" height="707" alt="image" src="https://github.com/user-attachments/assets/5788d62b-09fb-42a3-8e90-2a2af51ce114" />

### Preuve 2 — Fallback (pricing down)

1) Arrêter pricing-service
   
            docker compose stop pricing-service

3) Lancer à nouveau un borrow

           curl -X POST http://localhost:8081/api/books/1/borrow

<img width="1237" height="178" alt="image" src="https://github.com/user-attachments/assets/f77baf75-7ba9-43e3-a104-6299c7987ea0" />

## Pourquoi le verrou DB est obligatoire (multi-instances)

Avec 3 instances de book-service, plusieurs requêtes peuvent arriver en même temps sur des conteneurs différents.
Sans verrou en base de données, deux services peuvent lire le même stock et le décrémenter → stock incohérent ou négatif.
Le verrou transactionnel MySQL garantit qu’une seule instance modifie un livre à la fois, assurant une cohérence globale.

  ### Rôle du Circuit Breaker & Fallback

- Le circuit breaker détecte que pricing-service ne répond plus et stoppe les appels pour éviter les délais et erreurs en cascade.

- Le fallback est une réponse de secours qui permet à book-service de continuer à fonctionner (ici price = 0.0).

- Cela garantit une application stable et tolérante aux pannes.

## Vidéo de démonstration : 


https://github.com/user-attachments/assets/e7dd31b8-7a18-4d1e-893e-9e9a7188495b


## Conclusion 

Dans ce TP, nous avons mis en place un système de microservices avec plusieurs instances du book-service connectées à une base MySQL.
Le test de charge a montré que malgré 50 requêtes concurrentes, le stock final reste cohérent et atteint 0.
Grâce au verrouillage en base de données, aucune incohérence n’apparaît même en multi-instances.
Le circuit breaker empêche les appels inutiles vers un service en panne.
Le fallback permet à l’application de continuer à fonctionner avec un prix par défaut lorsque pricing-service est indisponible.

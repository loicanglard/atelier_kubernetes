# Rendu — Atelier Kubernetes

**Nom / Prénom :** ANGLARD Loïc  
**Groupe :** ING1 SC_AFR1

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?

   > La commande affiche trois nœuds : `k3d-lab-server-0`, qui correspond au control plane, et `k3d-lab-agent-0` ainsi que `k3d-lab-agent-1`, qui correspondent aux deux workers. Le control plane assure la gestion du cluster tandis que les workers exécutent les applications.

2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?

   > La commande affiche quatre conteneurs Docker : les trois nœuds Kubernetes et le load balancer `k3d-lab-serverlb`. Dans k3d, un nœud Kubernetes est concrètement un conteneur Docker exécutant K3s.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)

   > Un Deployment, un ReplicaSet et un Pod contenant Nginx ont été créés. Le Service `web` a ensuite été créé avec la commande `kubectl expose`.

4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?

   > Après la suppression du pod `web-68d995574f-lgc2w`, Kubernetes a automatiquement créé le pod `web-68d995574f-sk72r`. Le ReplicaSet maintient le nombre de réplicas défini dans le Deployment. C'est le principe du self-healing.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?

   > Non, car la modification a été effectuée directement dans le conteneur. Lors de la suppression du pod, Kubernetes recrée un conteneur à partir de l'image Nginx originale. Les modifications locales ne sont donc pas conservées.

6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?

   > Le port-forward établit un tunnel vers un pod précis, même lorsqu'il cible un Service. Lorsque ce pod est supprimé, le tunnel ne bascule pas automatiquement vers le nouveau pod. Il est donc nécessaire de relancer la commande.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?

   > `kubectl create` permet de créer une nouvelle ressource Kubernetes, tandis que `kubectl apply` permet de créer ou de mettre à jour une ressource à partir d'un fichier YAML décrivant son état souhaité.

8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?

   > Le selector permet au Service d'identifier les pods vers lesquels rediriger le trafic grâce à leurs labels. Si aucun pod ne correspond au selector, le Service ne dispose d'aucun endpoint correspondant et ne peut pas transmettre les requêtes à l'application.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.

   > Les trois réplicas sont répartis sur les trois nœuds du cluster : les deux workers et le control plane.

   ```
   NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
   site-75775484b-d7pbq    1/1     Running   0          16s   10.42.0.6   k3d-lab-agent-1    <none>           <none>
   site-75775484b-ltth9    1/1     Running   0          16s   10.42.1.5   k3d-lab-server-0   <none>           <none>
   site-75775484b-v8vnm    1/1     Running   0          33s   10.42.3.8   k3d-lab-agent-0    <none>           <none>
   ```

10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?

    > Le pod `site-fc576997-g2h87` était en statut `ErrImagePull`, car Kubernetes ne parvenait pas à télécharger l'image `nginx:9.99`. La commande `kubectl describe pod -l app=site` a permis de diagnostiquer cette erreur.

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?

    > Oui, grâce au mécanisme de Rolling Update, Kubernetes a conservé les trois anciens pods utilisant l'image `nginx:1.28`, car le nouveau pod n'était pas opérationnel. Le Service pouvait donc continuer à transmettre les requêtes aux pods disponibles. L'accessibilité du site a été vérifiée avec la commande `curl http://localhost:8080`. Un rollback a ensuite été effectué avec `kubectl rollout undo deploy/site`.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :

    > https://humble-fiesta-vpg6q4jq6wjcx577-8585.app.github.dev/

13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?

    > Le Deployment peut être retrouvé avec la commande `kubectl get deploy mario -o yaml`. Portainer et kubectl communiquent tous les deux avec le Kubernetes API Server, qui permet de consulter et de gérer les ressources du cluster.

## Bonus (facultatif)

14. Bonus réalisés et remarques :

    > Aucun bonus réalisé. L'ensemble des tests obligatoires du script de vérification a été validé.

---

**Score obtenu avec `./scripts/check.sh` :** 15 / 18 (15/15 hors bonus, l'exercice 3 n'étant pas vérifiable automatiquement).
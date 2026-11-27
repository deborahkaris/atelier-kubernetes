# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :BAYIHE Karis**
**Groupe :RS1**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
> 3 noeuds s'affichent:
k3d-lab-server-0	control-plane	
k3d-lab-agent-0	worker	
k3d-lab-agent-1	worker
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >docker ps montre que chaque conteneur k3s fait tourner son propre containerd, qui lance les pods à l'intérieur du nœud. 

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > 3 objets ont été créés:
Deployment - deployment.apps/web
ReplicaSet - replicaset.apps/web-68d995574f
Pod - pod/web-68d995574f-sxrcj
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > En supprimant le pod, on a déclenché l'auto-réparation de Kubernetes : le ReplicaSet a constaté qu'il n'y avait plus de pod alors qu'il en voulait 1, et en a recréé un nouveau avec un autre nom. L'ancien tunnel port-forward restait lié à l'ancien pod, donc il fallait le relancer.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non il n'a pas survécu car le tunnel port-forward était attaché à web-68d995574f-r89cw, qui n'existe plus, donc il est cassé.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Le message a disparu, parce que le ReplicaSet a recréé un pod neuf, repartant de l'image nginx d'origine.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > kubectl create est impératif : il crée l'objet et échoue s'il existe déjà. kubectl apply est déclaratif : il crée l'objet s'il n'existe pas, ou le met à jour pour qu'il corresponde au fichier.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector d'un Service désigne, par leurs labels, les pods qui reçoivent le trafic. S'il ne correspond à aucun pod, le Service existe bien mais n'a aucun endpoint : le trafic ne va nulle part et aucune erreur n'est signalée à la création.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ``` Les 3 réplicats se trouvents sur le noeud: k3d-lab-server-0
   11:  replicas: 3                      # Exercice 5 : vous changerez cette valeur
configmap/site-html unchanged
deployment.apps/site configured
service/site unchanged
NAME                   READY   STATUS    RESTARTS   AGE     IP          NODE               NOMINATED NODE   READINESS GATES
site-75775484b-wd85v   1/1     Running   0          8m32s   10.42.2.5   k3d-lab-server-0   <none>           <none>

   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le pod site-fc576997-p7cck a d'abord été Pending, puis ErrImagePull et ImagePullBackOff, avec READY 0/1. La cause se lit avec kubectl describe pod site-fc576997-p7cck, dans la section Events. La commande qui donne la cause est: kubectl describe pod site-fc576997-p7cck

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui, le site est resté accessible pendant toute la panne. Le Deployment ne supprime les anciens pods qu'une fois les nouveaux Ready, et le pod en ImagePullBackOff ne l'est jamais. Les 3 anciens pods ont donc continué à servir le trafic.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://legendary-space-couscous-x5gwg6r4q5gg26jrw-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui, Portainer et kubectl parlent au même composant : l'API server Kubernetes, qui tourne dans le control-plane (k3d-lab-server-0). Le Deployment mario créé depuis Portainer est donc visible avec kubectl, puisque les deux lisent et écrivent dans le même état du cluster.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...

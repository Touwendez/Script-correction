

## [4 — Contexte]

Une voiture autonome, avant de prendre la moindre décision, doit d'abord voir son environnement. Et c'est précisément là que les choses se compliquent.

Aujourd'hui, la grande majorité des accidents de la route sont dus à des erreurs humaines. C'est toute la promesse de la voiture autonome : supprimer cette part d'erreur.

Mais quand une voiture autonome a un accident, c'est très souvent parce qu'elle n'est pas parvenu a détecter ce  qu'elle aurait du détecter. Sur cette image, le véhicule n'a pas détecté l'obstacle devant lui. Et ce type de défaillance arrive presque toujours dans les mêmes conditions c'est a dire des condition climatique défavorables qui ont tendance a dégrader  les systèmes de perception. 

*(silence)* C'est exactement le problème sur lequel nous travaillons.

## [5 — Le projet DriveSense]


Comme le montre ce schéma qui illustre la chaine de conduite autonome , la perception est le maillon central de la chaîne : elle vient après les capteurs, et tout dépend d'elle c'est a dire la planification de la trajectoire, le contrôle du véhicule — dépend d'elle. Autrement dit, si la perception se trompe, l'erreur se propage jusqu'au volant.


## [6 — Problématique]

Chaque capteur a ses forces et ses faiblesses selon la situation. La caméra voit mal la nuit et sous la neige. Le radar, lui, reste fiable. Le LiDAR est gêné par le brouillard.

On serait tenté de croire qu'il suffit de combiner tous les capteurs ensemble, en permanence, pour bien voir. Eh bien non, et c'est tout le cœur de notre sujet.

Le problème, c'est qu'ajouter un capteur qui voit mal ne fait pas qu'ajouter de l'information : il ajoute des erreurs, et peut dégrader le résultat finale.



La question devient donc : comment apprendre à la voiture à choisir automatiquement les bons capteurs, selon ce qu'elle a devant elle ?

Avant de répondre, regardons ce qui existe déjà.

## [7 — Séparateur : État de l'art]


## [8 — Les stratégies de fusion]

Pour combiner plusieurs capteurs, il existe quatre grandes familles de méthodes.
Fusion précoce
Fusion intermédiaire
Fusion tardive
Les trois premières — précoce, intermédiaire, tardive — ont un point commun : elles traitent tous les capteurs de la même façon, tout le temps, quelle que soit la situation.

La quatrième, la fusion adaptative, fait quelque chose de différent : elle ajuste l'importance de chaque capteur selon le contexte. C'est la seule qui permet de mettre de côté un capteur qui voit mal. C'est donc celle que nous avons choisie.

Un travail en particulier nous a servi de point de départ.

## [9 — Travaux de référence : HydraFusion]

Ce travail, c'est HydraFusion, publié en 2022, sur les mêmes données que nous.

HydraFusion propose une fusion sélective fondée sur un module de gating, décliné en trois strategies. 


## [10 — Trois stratégies de gating]

On peut choisir les capteurs de deux manières opposées : soit avec des règles écrites à la main par un expert, soit en laissant le réseau de neurones apprendre à choisir tout seul.

La première, à base de règles, est simple à comprendre mais rigide. La deuxième, à base d'apprentissage, est plus souple.

Nous avons implémenté ces deux approches, justement pour les comparer. C'est le fil conducteur de toute la suite de la présentation.

Mais avant de comparer, il faut des données. Voyons lesquelles.

## [11 — Séparateur : Travaux 2023 et 2024]

*(2 secondes)* Le projet ne part pas de zéro.

## [12 — CARLA et WeatherCNN]

Nous avons d'abord travaillé en simulation, dans un environnement virtuel appelé CARLA. L'intérêt, c'est qu'on y contrôle la météo à volonté : on peut générer autant de pluie ou de brouillard qu'on veut. Nous avons ainsi produit 36 000 images dans toutes les conditions.

Sur ces images, nous avons entraîné un petit réseau capable de reconnaître la météo à partir de la caméra. Assez fiable.

Mais on a été obliger de changer d'approche basculer sur RADIATE et cela du a des problèmes de ressource .

## [13 — De CARLA à RADIATE]

Le problème de la simulation, c'est qu'elle est trop propre. Les signaux y sont parfaits. Or ce que nous voulons justement gérer, c'est le bruit, les imperfections des vrais capteurs. Et ça, on ne l'a que dans des données réelles.

Nous sommes donc passés à un jeu de données réel, appelé RADIATE, enregistré dans de vraies conditions : vraie pluie, vrai brouillard, vraie neige.

Un point important sur ce graphique : il y a énormément de voitures dans les données, mais très peu de vélos et de piétons. Ce déséquilibre reviendra dans nos résultats tout à l'heure — gardez-le en tête.

Encore une étape technique, et nous pourrons mesurer les capteurs.

## [14 — Synchronisation]

Les trois capteurs ne prennent pas leur image exactement au même instant. Le radar est plus lent que les autres : il prend environ quatre images par seconde, les autres sont plus rapides.

Le souci, c'est qu'un même numéro d'image ne correspond pas au même moment réel. Entre deux prises, une voiture a eu le temps de bouger. Nous avons donc tout recalé sur l'horloge du radar, pour être sûrs que les trois capteurs décrivent bien la même scène, au même instant.

Maintenant, mesurons ce que vaut chaque capteur, tout seul.

## [15 — Séparateur : Détecteurs monomodaux]

*(2 secondes)* Que vaut chaque capteur, séparément ?

## [16 — Protocole]

Un mot rapide sur la méthode, parce qu'elle vaut pour tout ce qui suit. Nous avons évalué les capteurs sur 22 scènes de test, couvrant toutes les conditions météo. Et surtout, tout est mesuré exactement de la même façon. Cela veut dire que tous les chiffres que je vais vous montrer sont directement comparables entre eux.

Un seul point de méthode que nous assumons : parmi les trois capteurs, nous n'avons réentraîné que le LiDAR.

Voici le verdict.

## [17 — Performances et limites]

Le message de cette slide tient en une phrase : aucun capteur n'est bon partout.

Le radar est le plus solide dans l'ensemble, mais il produit parfois de fausses détections. La caméra voit très bien de jour, mais elle s'effondre la nuit et sous la neige. Et le LiDAR est le plus faible des trois, même s'il reste précis sur la forme des objets.

*(silence)* Conclusion : si l'on fige la fusion une fois pour toutes, on subira forcément le capteur défaillant du moment. Il faut donc choisir les capteurs selon la scène.

Mais avant de choisir, nous avons dû résoudre un obstacle. Les capteurs ne voient pas dans le même repère.

## [19 — Le socle commun : LSS]

Voici le problème. Le radar et le LiDAR voient la scène d'en haut, comme sur un plan : chaque point a une position réelle, en mètres. La caméra, elle, voit de face, comme nos yeux, et sans notion de distance.

Résultat : on ne peut pas les combiner directement, ils ne parlent pas la même langue géométrique. La solution que nous utilisons s'appelle le LSS. Elle transforme l'image de la caméra en une vue de dessus, pour la mettre dans le même repère que les deux autres.

*(silence)* Et un point vraiment important : ce module sert à nos *deux* méthodes. Ce n'est pas propre à l'une ou à l'autre. La seule vraie différence entre nos deux approches, ce sera la façon de choisir les capteurs.

Justement, première façon de choisir : les règles.

## [18 — Séparateur : Knowledge Based Gating]

*(2 secondes)* Première approche : des règles d'expert.

## [20 — Architecture des règles]

L'idée de cette première méthode est simple, et c'est sa force : on devine la météo, et une table décide quels capteurs privilégier.

Concrètement, le réseau météo dont je vous ai parlé lit la caméra et annonce la condition : brouillard, neige, nuit… Ensuite, une table écrite par nous attribue une importance à chaque combinaison de capteurs. Par exemple : s'il y a du brouillard, on privilégie le radar. En ville par temps clair, on privilégie la caméra.

Le grand avantage, c'est qu'on comprend toujours pourquoi tel capteur a été choisi. C'est totalement transparent.

Est-ce que ça marche ? En partie.

## [21 — Résultats des règles]

Cette méthode a un vrai atout : elle retrouve davantage d'objets que n'importe quel capteur utilisé seul. La fusion récupère des objets que les capteurs manquaient chacun de leur côté.

Mais elle a aussi une limite nette : elle localise moins bien les objets que le radar seul. Elle plafonne. Et la raison est simple : des règles écrites à l'avance ne peuvent pas coller parfaitement à toutes les situations réelles.

*(On ne commente pas le tableau du bas en détail)* Nous avons aussi choisi, au passage, la meilleure méthode pour combiner les détections.

Un exemple concret pour illustrer.

## [22 — Exemple qualitatif des règles]

Voici une scène de ville, par temps clair. Ici, la règle fait confiance à la caméra, et c'est le bon choix : la caméra apporte la texture, le LiDAR apporte la précision. Dans ce cas favorable, la méthode fonctionne bien.

Mais cette approche a des limites de fond.

## [24 — Limites des règles]

Le vrai défaut de cette méthode, le voici : pour deviner la météo, elle dépend de la caméra. Or la caméra, c'est justement le capteur le plus fragile, celui qui lâche en premier quand les conditions se dégradent. C'est un peu paradoxal : on s'appuie sur le capteur le moins fiable pour décider.

Il y a d'autres limites : les règles sont figées, donc une situation imprévue n'a pas de règle adaptée ; et on ne considère qu'une seule météo pour toute l'image, ce qui est trop grossier.

*(silence)* De là est née notre idée principale : garder exactement le même pipeline, mais au lieu de dicter des règles au réseau, le laisser apprendre lui-même à choisir. C'est notre contribution.

## [25 — Séparateur : PercepFlow]

*(2 secondes)* Deuxième approche : laisser le réseau apprendre.

## [26 — Architecture PercepFlow]

Notre méthode, que nous avons appelée PercepFlow, garde le même pipeline que les règles. Mais elle remplace la table écrite à la main par un module qui apprend tout seul quels capteurs sont fiables selon la scène.

Il y a trois capteurs, donc sept combinaisons possibles, et le réseau en retient les trois meilleures à chaque instant. Et pour décider, il ne regarde pas que les images : il regarde aussi l'état des capteurs, la luminosité, le contraste.

*(silence)* La différence essentielle avec la méthode précédente : ici, aucune règle météo n'est écrite à la main. Le réseau apprend tout seul.

Un mot sur la façon dont on l'entraîne, parce que l'ordre a son importance.

## [27 — Entraînement]

Nous entraînons le système en deux temps, jamais les deux en même temps.

Dans un premier temps — en orange sur le tableau — les capteurs apprennent à détecter les objets, pendant que le module de choix reste gelé. Dans un second temps — en bleu — on fait l'inverse : on gèle les capteurs, et seul le module de choix apprend.

Pourquoi séparer ? Parce que si on entraîne tout en même temps, le module apprend à choisir sur des bases encore instables, et ça part dans tous les sens. C'est d'ailleurs exactement ce qui bloquait le système l'an dernier.

Venons-en aux résultats.

## [28 — Résultats globaux]

S'il y a une chose à retenir de cette slide, c'est celle-ci : notre méthode obtient la meilleure localisation de toutes les approches testées. Meilleure que chaque capteur seul, et meilleure que la méthode par règles.

*(silence)* Je veux être transparent sur un point : sur un autre critère, qui mélange la détection et la précision, le radar seul reste très légèrement devant nous. Nous localisons mieux, mais il nous reste une marge de progression sur cet équilibre-là. C'est un de nos chantiers pour la suite.

Et pour situer les choses : le prototype de l'an dernier était instable, autour de 13 %. Aujourd'hui, le système est stable, et nous avons presque triplé la performance.

Voyons maintenant où ça marche moins bien.

## [29 — Résultats par classe]

Les gros objets — voitures, camions — sont très bien détectés. Les petits — piétons, vélos, motos — restent difficiles.

Mais attention, et c'est important : ce n'est pas notre méthode de fusion qui est en cause. C'est le déséquilibre des données dont je vous parlais au début : il y a très peu de vélos et de piétons dans les données d'entraînement. Le problème est dans les données, pas dans la fusion.

Le plus parlant, maintenant, c'est de voir le système en action, sur trois scènes réelles.

## [30 — Scène de nuit]

Première scène : la nuit, en pleine obscurité.

Regardez ce que fait la caméra toute seule : elle détecte quatre bus… qui n'existent pas. Ce sont juste des reflets de phares et des lampadaires.

Notre système, lui, donne à la caméra un score minuscule — vous le voyez dans le tableau — et il se fie au radar. Résultat : au lieu de quatre fausses détections, il garde deux détections, et les deux sont correctes.

Sous la neige, même histoire.

## [31 — Scène de neige]

Ici, sous la neige, la caméra détecte un bus sur le bord droit de l'image, là où il n'y a en réalité qu'un panneau. Le radar, lui, voit le vrai bus, au bon endroit.

Notre système écarte à nouveau la caméra, élimine la fausse détection, et conserve le vrai bus.

Et dans le brouillard le plus épais, c'est encore plus net.

## [32 — Scène de brouillard]

Dans ce brouillard très dense, la caméra ne voit plus rien du tout : zéro détection. Sans le radar, l'objet serait tout simplement invisible. Mais le radar, lui, identifie la voiture, et le système la restitue correctement.

*(silence, c'est le moment fort)* Et voici le point le plus important de toute la présentation. Sur ces trois scènes — nuit, neige, brouillard — le système a mis la caméra de côté tout seul, et s'est appuyé sur le radar. Sans qu'on lui écrive la moindre règle de météo. Il l'a appris.

C'est exactement ce que faisaient les règles écrites à la main… mais cette fois, c'est appris tout seul.

Un dernier mot sur l'outil que nous avons construit.

## [33 — Séparateur : Interface et conclusion]

*(2 secondes)* Pour finir : l'outil, et le bilan.

## [34 — Interface]

Nous avons développé une interface qui permet de voir, en direct, ce que le système décide : les trois capteurs, les détections de chacun, et les scores de choix en temps réel.

C'est grâce à elle que nous avons pu comprendre le comportement du système. D'ailleurs, les trois scènes que je viens de vous montrer viennent directement de cet outil.

En résumé.

## [35 — Conclusion]

Nous avons construit un système qui choisit tout seul les bons capteurs selon la scène, et qui obtient la meilleure localisation de toutes les approches que nous avons testées.

Il ramène la caméra dans le bon repère, il s'adapte à la météo sans qu'on lui dicte de règles, et il reste à progresser sur les petits objets. Pour le projet DriveSense, ce qui était l'an dernier un prototype instable est devenu un système stable, qu'on peut évaluer sérieusement.

Et pour la suite ?

## [36 — Perspectives]

Pour la seconde moitié du stage, nous avons trois chantiers. D'abord, mieux détecter les petits objets, les piétons et les vélos. Ensuite, améliorer encore la partie caméra. Et enfin, préparer le passage vers le temps réel, qui est le deuxième axe du projet DriveSense.

*(optionnel, pour ouvrir la discussion)* Et sur la question des petits objets en particulier, nous sommes tout à fait preneurs de vos idées.

## [37 — Merci]

Merci beaucoup pour votre attention. Nous répondrons à vos questions avec plaisir.

---

# Réponses simples aux questions probables

**« C'est quoi la vue de dessus, le BEV ? »**
C'est une carte vue d'en haut, comme un plan. Le radar et le LiDAR nous la donnent naturellement. La caméra non, puisqu'elle voit de face — c'est pour ça qu'on la convertit.

**« Pourquoi le radar marche dans le brouillard et pas la caméra ? »**
Le radar envoie des ondes qui traversent le brouillard et la pluie. La caméra dépend de la lumière visible, exactement comme notre œil : dans le brouillard, elle est aussi aveugle que nous.

**« Pourquoi ne pas simplement tout fusionner en permanence ? »**
Parce qu'un capteur qui voit mal ajoute des erreurs. Nous l'avons mesuré : tout fusionner donne un moins bon résultat que bien choisir.

**« Votre système est-il meilleur que le radar tout seul ? »**
Pour localiser les objets, oui, nettement. Sur l'équilibre global entre détection et précision, le radar reste très légèrement devant — et c'est justement un de nos points de travail pour la suite.

**« Est-ce que ça fonctionne en temps réel ? »**
Pas encore, et c'est précisément l'objet du deuxième axe du projet. Cela dit, nous exécutons seulement trois capteurs sur sept, donc nous avons déjà réduit le coût de calcul.

**« Pourquoi n'avoir réentraîné que le LiDAR ? »**
Le radar et la caméra utilisaient déjà des modèles fiables. Le LiDAR, dans notre représentation en vue de dessus, avait besoin d'être réadapté pour être exploitable.

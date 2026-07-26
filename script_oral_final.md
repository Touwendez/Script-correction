# Script oral final — Soutenance PercepFlow
**Présentation3 · 39 slides (37 de corps + 2 annexes) · cible 15 minutes**
Touwendé Ouédraogo · Projet DriveSense · Audensiel

---

## Repères de durée

Corps parlé : environ **2 150 mots**.

| Débit | Durée |
|---|---|
| 150 mots/min (soutenu) | 14 min 20 |
| 140 mots/min (posé) | 15 min 20 |

Les slides 3, 7, 11, 15, 18, 23, 31 sont des **séparateurs de section** : ils défilent en 2 à 3 secondes avec la seule phrase indiquée. Les slides 37, 38 et 39 sont en **annexe** : vous ne les présentez pas, vous y allez seulement si le jury pose une question.

**Le LSS est désormais présenté comme socle commun aux deux architectures (slide 19), avant le Knowledge Based Gating.** Ne le présentez plus comme un apport propre à PercepFlow.

Tous les chiffres correspondent au protocole 22 séquences.

---

# SCRIPT

## Ouverture · 35 s

### [1 — Titre]

Bonjour à toutes et à tous. Je m'appelle Touwendé Ouédraogo, étudiant en Master Systèmes Machines Autonomes et Réseaux Terrain à l'Université de Lille.

Je vous présente le travail mené chez Audensiel dans le cadre du projet DriveSense, sous l'encadrement de Madame Amira Mimouna : la fusion sélective multicapteurs pour la perception robuste en conduite autonome.

### [2 — Sommaire]

Je poserai le contexte, puis l'état de l'art. Je présenterai les travaux antérieurs, l'évaluation des capteurs isolés, puis les deux stratégies de fusion étudiées : une approche experte, puis l'architecture PercepFlow. Je terminerai par l'interface et les perspectives.

---

## 1 · Contexte et problématique · 1 min 40

### [3 — Séparateur]
Commençons par le contexte.

### [4 — Contexte]

La perception est la fonction critique du véhicule autonome : avant de décider, il faut voir.

Plus d'un million trois cent mille décès par an sur les routes, dont 94 % liés à une erreur humaine : c'est la promesse du véhicule autonome, supprimer cette part d'erreur.

Mais les accidents impliquant des véhicules autonomes ont un point commun. Sur cette photo, une Tesla a percuté à pleine vitesse un camion renversé en travers de la chaussée : le véhicule n'a pas vu l'obstacle. La défaillance est une défaillance de perception, et elle survient presque toujours en conditions dégradées.

### [5 — Projet DriveSense]

Ce travail s'inscrit dans DriveSense, projet de R&D mené chez Audensiel, structuré en trois axes : perception hors ligne, perception temps réel, et modélisation de la scène.

Comme le montre ce schéma, la perception est le maillon entre les capteurs et la planification. Toute la chaîne en dépend : une erreur de perception se propage jusqu'au contrôle du véhicule. Je me situe sur le premier axe, la perception hors ligne.

### [6 — Problématique]

Chaque capteur dépend du contexte : la caméra se dégrade la nuit et sous la neige, le radar reste robuste, le LiDAR souffre du brouillard.

L'intuition serait de tout fusionner en permanence. C'est faux, et c'est le cœur du sujet : quand un capteur devient bruité, l'intégrer dégrade la détection au lieu de l'améliorer.

D'où ma question centrale : comment concevoir une architecture capable de **sélectionner dynamiquement les modalités les plus pertinentes selon la scène**, en tenant compte du contexte météorologique ?

---

## 2 · État de l'art · 1 min 40

### [7 — Séparateur]
Voyons comment la littérature aborde ce problème.

### [8 — Stratégies de fusion]

Quatre familles existent. La fusion précoce combine les données brutes, l'intermédiaire les caractéristiques, la tardive les détections finales. Ces trois-là partagent un défaut : elles sont **fixes**, elles traitent tous les capteurs de la même manière, y compris quand l'un est aveugle.

La quatrième, la fusion adaptative, conditionne la sélection des modalités au contexte d'acquisition, ce qui réduit l'influence d'un capteur dont la fiabilité chute en conditions dégradées. C'est celle que j'ai retenue.

### [9 — Travaux de référence]

Trois travaux structurent le domaine, mais c'est HydraFusion qui est ma référence directe : même jeu de données RADIATE, même principe de sélection de branches.

Regardez ses résultats. Fusionner **toutes** les branches donne 65,47 % de mAP ; sélectionner les **trois meilleures** en donne 81,31 %. Moins fusionner donne un meilleur résultat, à condition de bien choisir. C'est le fondement de l'approche.

L'article établit aussi que l'Attention Gating surpasse le Deep puis le Knowledge Gating, que k égal 3 est optimal, et que les gates retiennent la caméra moins de 10 % du temps.

### [10 — Trois stratégies de gating]

HydraFusion propose une fusion sélective fondée sur un module de gating, décliné en trois stratégies.

Le Knowledge Gating associe un contexte externe à un profil de branches : interprétable, mais fondé sur des règles écrites à l'avance. Le Deep Gating apprend la sélection par convolutions. L'Attention Gating y ajoute une couche de self-attention.

J'ai implémenté la première et la troisième, précisément pour les comparer sur mes propres données.

---

## 3 · Travaux antérieurs · 1 min 30

### [11 — Séparateur]
Le projet ne démarre pas de zéro.

### [12 — CARLA et WeatherCNN]

En 2023, un jeu de données simulé a été généré sous CARLA : neuf conditions météorologiques, quatre villes, deux véhicules, 36 000 captures au total dont 11 851 images annotées.

Sur ces données a été développé le WeatherCNN, qui identifie la condition depuis l'image caméra. Il atteint 97,6 % de F1 sur CARLA et 97 % sur RADIATE, ce dernier en trois epochs de fine-tuning seulement : le travail en simulation sert donc de préentraînement efficace.

### [13 — De CARLA à RADIATE]

Pourquoi ne pas rester en simulation ? Parce que les signaux radar et LiDAR simulés sont trop propres : le bruit capteur, précisément ce qu'on cherche à gérer, n'y est pas représentatif.

D'où le passage à RADIATE, jeu de données réel acquis en conditions dégradées, et jeu de référence de HydraFusion.

Ce graphique est important : l'échelle est logarithmique, et l'écart est considérable, plus de 72 000 voitures annotées en zone urbaine contre quelques dizaines de vélos. Ce déséquilibre expliquera une grande partie des écarts de performance.

### [14 — Synchronisation]

Une étape préalable méritait attention. Le radar tourne à environ 4 hertz, la caméra et le LiDAR sont plus rapides : utiliser le même numéro de frame ne garantit donc pas que les trois modalités décrivent le même instant.

Le SDK RADIATE prend le radar comme référence et apparie chaque modalité par le timestamp le plus proche. Ici, la frame radar 151 est associée à la caméra 576 et au LiDAR 385 : trois numéros différents, un même instant physique.

---

## 4 · Détecteurs monomodaux · 1 min 50

### [15 — Séparateur]
Avant de fusionner, il fallait mesurer chaque capteur isolément.

### [16 — Protocole]

Un mot sur le protocole, car il vaut pour toutes les architectures présentées : 22 séquences de test couvrant ville, carrefour, autoroute, brouillard, pluie, neige et nuit. Huit classes, seuils à 0,5. Tous les chiffres qui suivent sont donc directement comparables.

Un point de méthode que j'assume : seul le LiDAR a été réentraîné dans ce travail, le radar et la caméra utilisent des modèles préentraînés.

### [17 — Performances et limites]

Voici les résultats et le profil de chaque capteur.

Le radar est le plus robuste globalement, mais ses réflexions parasites créent des fausses détections. La caméra a un rappel élevé et une précision faible : elle s'effondre de nuit et sous la neige, et produit beaucoup de fausses alarmes. Le LiDAR est le plus faible des trois : son entraînement reste restreint et la projection en vue de dessus perd l'information verticale.

Le constat qui commande toute la suite : **aucune modalité n'est supérieure en toutes circonstances**. Une fusion fixe subirait donc le capteur défaillant du moment. Il faut sélectionner.

Mais avant de sélectionner, il fallait résoudre un problème géométrique.

---

## Socle commun · 1 min

### [19 — Socle LSS]

Radar et LiDAR produisent une vue de dessus, où chaque pixel correspond à une position réelle en mètres. La caméra, elle, fournit une vue perspective, sans profondeur. Pour fusionner les trois dans un repère unique, la caméra doit être amenée dans le même espace.

La méthode retenue est Lift, Splat, Shoot : un module entraînable qui estime la profondeur de chaque pixel, sur 24 intervalles entre 2 et 52 mètres, puis projette les caractéristiques visuelles dans la grille en vue de dessus. Les deux caméras stéréo sont ainsi réunies en une représentation BEV commune.

Ce point est important : **le LSS est un socle partagé par mes deux architectures**. Le Knowledge Based Gating comme PercepFlow reposent dessus. La seule chose qui les distingue, c'est la manière de sélectionner les branches.

---

## 5 · Knowledge Based Gating · 1 min 50

### [18 — Séparateur]
Première stratégie de sélection : des règles expertes.

### [20 — Architecture]

Le WeatherCNN prédit la condition depuis la caméra. Une table experte attribue alors un poids à chacune des sept branches, et les trois plus élevées sont exécutées, avant fusion par Soft-NMS.

La table se lit facilement : en brouillard et en neige, le radar est à 1,0 et la caméra à 0,3 ; de nuit et sous la pluie, radar plus LiDAR atteint 1,0 ; en ville, les branches caméra sont privilégiées, car c'est la texture visuelle qui apporte le sens.

L'atout de cette approche est sa transparence : on sait toujours pourquoi telle branche est active.

### [21 — Résultats]

Les résultats sont contrastés, et c'est ce contraste qui est instructif.

Le rappel atteint 64,11 %, largement au-dessus de chaque capteur isolé : la fusion récupère des objets manqués par les modalités seules. Mais la mAP plafonne à 29,01 %, **en dessous du radar seul** à 32,36 %. On détecte plus, on localise moins bien.

J'ai aussi comparé les deux modes de fusion : Soft-NMS devance le WBF de près de sept points de mAP. C'est ce qui m'a conduit à l'utiliser dans les deux architectures.

### [22 — Analyse qualitative]

Un exemple, scène urbaine en conditions normales. Le gate applique la règle « normal » et privilégie les branches caméra : camera_lidar reçoit le poids maximal, car la caméra apporte la texture et le LiDAR la précision de profondeur.

### [23 — Séparateur / Limites]
*(Ce slide est le slide de limites, pas un séparateur — enchaînez directement.)*

### [24 — Limites]

Quatre limites expliquent ce plafond. Les règles sont figées a priori. La condition est prédite **depuis la caméra**, donc depuis la modalité la plus fragile, exactement quand les conditions se dégradent. La granularité est grossière, une seule condition par scène. Et la performance reste sous celle du radar seul.

D'où PercepFlow : conserver le même pipeline de fusion et la même mise en BEV, mais remplacer les règles expertes par une sélection apprise directement à partir des caractéristiques capteurs.

---

## 6 · PercepFlow · 3 min 45

### [25 — Séparateur]
Seconde stratégie, ma contribution principale.

### [26 — Architecture]

Voici l'architecture. Trois encodeurs partagés produisent les caractéristiques : radar et LiDAR en vue de dessus, caméra stéréo via LSS.

Le gate contextuel reçoit ces caractéristiques, mais aussi l'état des capteurs, la luminosité et le contraste de la scène. Sept branches Faster R-CNN, chacune avec son RPN et sa tête RoI. Le gate leur attribue un score, les trois meilleures sont exécutées, et leurs détections fusionnées par Soft-NMS.

La différence fondamentale avec l'approche précédente : **aucune règle météorologique n'est écrite**. Le réseau apprend seul.

### [27 — Entraînement]

L'entraînement se fait en deux stages, et ce tableau le résume.

Au Stage A, les trois encodeurs et les têtes des sept branches sont entraînés, en orange ; le gate reste gelé, en bleu. Au Stage B, tout s'inverse : les encodeurs sont figés, seul le gate reçoit un gradient, et la sélection s'active.

Pourquoi geler ? Parce que si les branches et le gate apprenaient ensemble, le gate optimiserait sa sélection sur des représentations encore instables. C'est exactement ce qui bloquait le système l'année dernière autour de 13 % de mAP.

### [28 — Résultats]

PercepFlow obtient **la meilleure mAP de toutes les configurations évaluées**, à 36,02 %, près de 4 points au-dessus du radar seul et 7 points au-dessus du Knowledge Based Gating.

Je dois être précis sur un point : le radar seul conserve un F1 légèrement supérieur, 61,56 % contre 60,30 %. PercepFlow localise mieux, mais n'améliore pas encore le compromis précision-rappel. C'est une limite que j'assume et qui figure dans mes perspectives.

### [29 — Par classe et ablation]

Le détail par classe confirme l'effet du déséquilibre : 52,96 % de mAP sur les véhicules, 19,09 % sur les classes rares.

L'ablation valide k égal 3 : on passe de 33,24 % à 36,02 % entre une et trois branches. Mais le gain est porté par les véhicules. Les classes rares sont insensibles au nombre de branches : leur limite tient aux données, pas à la fusion.

### [30 — Scène de nuit]

Passons au comportement réel du système, sur trois scènes.

Une scène de nuit, 7 % de luminosité. La caméra seule produit quatre détections de bus, toutes fausses : des reflets de phares et des lampadaires. Regardez la colonne des scores : la caméra reçoit 0.0002. Le gate retient radar, radar_lidar et all, et la fusion ne conserve que deux détections, toutes deux correctes.

### [31 — Scène de neige]

Sous la neige, même logique. La caméra détecte un bus à 0,82 sur le bord droit, là où il n'y a qu'un panneau ; le radar détecte le bus réel à 1,00. Le gate écarte à nouveau la caméra, score 0.0005, et la fusion élimine la fausse détection en conservant le bus réel.

### [30bis / 32 — Scène de brouillard]

Enfin le brouillard. La caméra ne produit **aucune** détection. Le radar identifie une voiture à 0,93, la fusion la restitue à 0,92. Sans le radar, l'objet serait invisible.

Et c'est ici que le point essentiel apparaît. Sur ces trois scènes, le score de la branche caméra vaut 0.0002, 0.0005 et 0.0004 : toujours de l'ordre du dix-millième. Mais camera_radar monte à 0.2746 dans le brouillard, deuxième branche retenue. La caméra reste donc informative en combinaison, alors qu'elle est inutilisable seule.

Le réseau a appris tout cela sans qu'aucune règle météorologique ne soit écrite.

---

## 7 · Interface et conclusion · 1 min 30

### [33 — Séparateur]
Un mot sur l'outil développé, puis le bilan.

### [34 — Interface]

J'ai développé une interface de visualisation, backend Python Flask et frontend HTML, CSS, JavaScript. Elle affiche les trois flux capteurs, les détections par branche et après fusion, et les scores du gate en temps réel.

C'est elle qui m'a permis de diagnostiquer le comportement du modèle, et pas seulement de constater ses métriques. Les trois scènes précédentes en sont issues.

### [35 — Conclusion]

En résumé. PercepFlow est une architecture de fusion sélective fondée sur un gate contextuel appris, qui opère dans l'espace BEV commun où la caméra stéréo est amenée par LSS.

Elle obtient la meilleure mAP de toutes les configurations évaluées, 36,02 %. Le radar seul garde un F1 légèrement supérieur, ce qui reste à améliorer. Les véhicules atteignent près de 53 % de mAP, les classes rares 19 %.

Pour DriveSense, le système prototypé en 2024, alors instable autour de 13 % de mAP, est aujourd'hui stabilisé, complété par une branche caméra géométriquement cohérente, et évalué sur un protocole unifié.

### [36 — Perspectives]

Cinq perspectives. Poursuivre l'entraînement de la branche caméra en BEV. Renforcer les classes rares par équilibrage et sur-représentation des données. Améliorer le compromis précision-rappel, encore en retrait sur le radar. Et préparer le passage vers la perception temps réel, deuxième axe du projet.

### [37 — Merci]

Je vous remercie de votre attention et je suis à votre disposition pour vos questions.

---

# Annexes (à ne présenter que sur question)

### [38 — Soft-NMS]
À utiliser si le jury interroge sur le mode de fusion. Message : trois branches détectent souvent le même objet ; le NMS classique supprime les boîtes redondantes, y compris un objet voisin distinct ; Soft-NMS réduit le score au lieu de supprimer, ce qui préserve les détections dans les scènes denses. Validé empiriquement : 29,01 % contre 22,08 % pour le WBF.

### [39 — Comparaison HydraFusion]
À utiliser si le jury compare à l'article. Message : les deux utilisent sept branches, mais HydraFusion en consacre trois aux caméras séparées, tandis que je réunis les caméras par LSS pour libérer des branches multimodales ; et surtout, HydraFusion projette les détections vers la caméra en perspective, alors que mon pipeline amène la caméra en BEV.

---

# Questions probables du jury

**« Votre F1 est inférieur à celui du radar seul, votre architecture apporte-t-elle vraiment quelque chose ? »**
La mAP, qui mesure la qualité de localisation sur l'ensemble des seuils de confiance, est supérieure de près de 4 points. Le F1 est mesuré à un seuil unique et reflète surtout un excès de détections. C'est une limite réelle, que j'identifie dans mes perspectives, mais la capacité à localiser correctement est le critère premier en détection d'objets.

**« Pourquoi la branche caméra a-t-elle un score aussi faible ? »**
Le module LSS est entraîné from scratch et la supervision de profondeur est indirecte, donc la représentation caméra en vue de dessus reste imparfaite. Mais ce faible score n'est pas un échec : c'est le gate qui fonctionne correctement. Et camera_radar figure parmi les branches les plus sélectionnées, ce qui montre que la caméra apporte de l'information en combinaison.

**« Le LSS est-il votre contribution ou celle de PercepFlow ? »**
Le LSS est un socle commun à mes deux architectures. Le Knowledge Based Gating l'utilise aussi. Ce qui distingue mes deux méthodes, c'est uniquement le mécanisme de sélection : règles expertes d'un côté, gate appris de l'autre.

**« Vos 36 % sont loin des 81 % de HydraFusion. »**
Les protocoles diffèrent : sous-ensemble de séquences, mapping de classes, seuils. Une comparaison chiffrée directe serait trompeuse. Ce que je compare, ce sont les tendances, et sur trois points elles convergent : hiérarchie des branches, valeur optimale de k, comportement du gate vis-à-vis de la caméra.

**« Le Knowledge Based Gating est-il inutile ? »**
Non. Il reste interprétable et prévisible, ce qui compte pour l'analyse de sûreté en automobile. Ma perspective est de l'hybrider avec l'attention : utiliser le contexte météorologique prédit pour guider la sélection apprise.

**« Pourquoi Soft-NMS et pas WBF ? »**
Je les ai comparés à sélection identique : 29,01 % contre 22,08 % de mAP. Le WBF fusionne les boîtes par moyenne pondérée, ce qui lisse les positions et pénalise la localisation quand les branches ont des précisions géométriques différentes.

**« Comment gérez-vous le coût de calcul ? »**
Trois branches sur sept sont exécutées, soit environ 43 % du coût d'une fusion complète, pour un meilleur résultat. Le chiffrage en latence embarquée relève du deuxième axe de DriveSense.

---

# Aide-mémoire

| Chiffre | Signification |
|---|---|
| **36,02 %** | mAP de PercepFlow, meilleure de toutes |
| **60,30 %** | F1 de PercepFlow |
| **32,36 % / 61,56 %** | mAP et F1 du radar seul |
| **28,84 %** | mAP de la caméra seule |
| **15,44 %** | mAP du LiDAR seul |
| **29,01 %** | mAP du Knowledge Based Gating |
| **22,08 %** | mAP du KnowledgeGate avec WBF |
| **81,31 %** | mAP de HydraFusion dans l'article, protocole différent |
| **97 %** | F1 du WeatherCNN sur RADIATE |
| **52,96 % / 19,09 %** | mAP véhicules / classes rares |
| **0.0002 · 0.0005 · 0.0004** | score de la caméra en nuit, neige, brouillard |
| **0.2746** | score de camera_radar dans le brouillard |
| **22** | séquences du protocole de test |
| **k = 3** | branches activées sur 7 |

---

# Deux vérifications avant le jour J

1. **L'ordre réel des slides 18 et 19 dans votre fichier.** Dans l'export que j'ai lu, le séparateur de section 5 (slide 18) apparaît après le socle LSS (slide 19). Ouvrez votre présentation et vérifiez que l'enchaînement à l'écran est bien : performances monomodales → socle LSS → séparateur Knowledge Based Gating → architecture. Si l'ordre diffère, adaptez l'enchaînement du commentaire des sections 4 et 5.

2. **La séquence night_1_1.** Elle ne figure pas dans la liste des 22 séquences de test. Vérifiez avec votre encadrante si elle appartient à l'entraînement ou à la validation, pour pouvoir répondre si on vous interroge sur cette scène qualitative.

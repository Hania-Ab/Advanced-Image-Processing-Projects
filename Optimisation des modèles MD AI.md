# Optimisation des Modèles d'Imagerie Médicale avec MD.ai

Dans ce projet, j'ai exploré l'impact des paramètres d'entraînement et des techniques d'augmentation de données sur la performance du modèle MD.ai, une plateforme spécialisée dans l'analyse et le développement de modèles d'IA pour l'imagerie médicale. Mon objectif principal était de comprendre comment différents paramètres influencent la précision, la capacité de généralisation et la robustesse du modèle dans des tâches de segmentation d'images médicales, en particulier pour les poumons.

## 1. Paramètres d'entraînement

### Paramètres initiaux

Pour commencer, j'ai utilisé les paramètres par défaut suivants :

- steps_per_epoch = 100
- epochs = 10
- batch_size = 8

Les résultats initiaux ont montré une bonne performance du modèle :

- Training Accuracy : 96.9%
- Validation Accuracy : 98.3%
- DICE Score : 94.4%

Les masques prédits pour la segmentation des poumons étaient proches de la vérité terrain, indiquant que le modèle capturait bien la forme des poumons. Voici une visualisation des résultats obtenus :

![image](https://github.com/user-attachments/assets/a6bdb23a-a098-42a7-8fe7-94b89aecddcf)

### Variation du Batch Size 
Pour optimiser le modèle, j'ai modifié la taille des lots (batch size) et observé son impact :

| Batch Size |	Training Accuracy |	Validation Accuracy |	DICE Score |	Temps d'exécution |
| -------- | -------- | -------- | -------- | -------- |
| 32 |	0.966 |	0.966 |	0.92 |	9 min 24 s |
| 16 |	0.966 |	0.961 |	0.925 |	5 min 26 s |
| 8 |	0.969 |	0.983 |	0.944 |	4 min 27 s |

La taille de lot **8** a donné les meilleurs résultats avec une meilleure précision et un temps d'exécution plus court. Les lots plus petits ont permis au modèle de mieux généraliser, ce qui est essentiel dans les projets d'imagerie médicale.

- Visualisation avec Batch Size = 8 :

![image](https://github.com/user-attachments/assets/68b4581d-145a-4210-aab5-e6ce8fd33e60)

- Visualisation avec Batch Size = 16 :

![image](https://github.com/user-attachments/assets/47a9e43e-42da-4c5c-92a6-64d53b7107c4)

- Visualisation avec Batch Size = 32 :

![image](https://github.com/user-attachments/assets/dc3da3ac-3634-4769-9086-13b269643b3a)

### Variation du nombre d'époques :

Ensuite, j'ai testé différentes valeurs pour le nombre d'époques :

| Nombre d'époques |	Training Accuracy |	Validation Accuracy |	DICE Score |	Early Stopping |
| -------- | -------- | -------- | -------- | -------- |
| 20	| 0.962 |	0.933 |	0.907 |	15 |
| 10 |	0.969 |	0.983 |	0.944 |	Non |
| 5	| 0.960 |	0.941 |	0.921 |	Non |

Il est apparu que 10 époques étaient optimales. À 20 époques, le modèle commençait à surajuster, et l'early stopping a été déclenché à la 15ème époque, signalant que le modèle avait déjà convergé.

## 2. L'impact de l'augmentation de données 

L'augmentation de données est une technique clé pour générer des variations artificielles des images d'entraînement, augmentant ainsi la taille du jeu de données disponible. J'ai appliqué des transformations comme la rotation, le décalage, le zoom et le retournement pour tester l'impact sur le modèle.

### Paramètres d'Augmentation

- Rotation Range : 180°
- Width/Height Shift Range : 0.3
- Shear Range : 0.1
- Zoom Range : 0.1
- Horizontal/Vertical Flip : Activé
- Fill Mode : Constant

### Variation du Shift Range 

Lorsque j'ai augmenté les valeurs de width_shift_range et height_shift_range à **0.8**, la performance du modèle a diminué :

- Training Accuracy : 0.97
- Validation Accuracy : 0.89
- DICE Score : 0.775

Le masque prédit englobait une partie incorrecte du poumon, indiquant une sursegmentation. Le score DICE de **77%** était relativement faible.

![image](https://github.com/user-attachments/assets/5d5bd89d-c67e-4838-a116-a9e7107f7974)

![image](https://github.com/user-attachments/assets/860fc581-e441-45a6-b60a-abc32064cafc)

### Variation de la Rotation

En réduisant le paramètre **rotation_range** de **180°** à **90°**, j'ai observé une légère amélioration du score DICE (**89.5%**). Cependant, le modèle souffrait de surajustement avec une différence significative entre les précisions d'entraînement et de validation.

![image](https://github.com/user-attachments/assets/65fcdfc3-ca00-4dbd-8821-49d57aa9cebd)

![image](https://github.com/user-attachments/assets/c0ca0be4-9421-41ae-885c-071074100764)


### Variation du Shear et Zoom Range 

Lorsque j'ai augmenté les valeurs de **shear_range** et **zoom_range** à **0.8**, le modèle a montré des signes de distorsion excessive des images, ce qui a réduit la performance :

- Training Accuracy : 0.94
- Validation Accuracy : 0.88
- DICE Score : 0.82

![image](https://github.com/user-attachments/assets/a1171b04-2ec5-4c03-b7d2-df20d2b8d1a4)

Cela souligne que des augmentations trop importantes peuvent avoir un impact négatif, déformant les objets dans les images et rendant plus difficile la généralisation du modèle.

## 3. Impact de l'Exclusion de l'Augmentation de Données

Enfin, j'ai entraîné le modèle sans aucune augmentation de données pour voir si cela améliorerait la performance. Sans augmentation, j'ai obtenu une meilleure précision d'entraînement, mais une précision de validation plus faible, indiquant un surapprentissage.

| Condition |	Training Accuracy |	Validation Accuracy |	DICE Score |
| -------- | -------- | -------- | -------- |
| Avec augmentation |	0.969 |	0.983 |	0.944 |
| Sans augmentation	| 0.9867	| 0.96 |	0.95 |
| Sans aug. (90% data)	| 0.987 |	0.968 |	0.95 |

## Conclusion

À travers cette étude, j'ai pu démontrer que l'ajustement précis des paramètres d'entraînement et d'augmentation de données est essentiel pour améliorer la performance des modèles MD.ai. L'augmentation de données permet de réduire le surajustement, tandis que des paramètres tels que le batch size et le nombre d'époques influencent directement la précision du modèle. Ces expériences m'ont permis d'optimiser les performances du modèle et d'en améliorer la capacité de généralisation.


# 3D Design Review Agent

Prototype d'agent IA qui vérifie des règles de conception sur une maquette 3D, surligne en rouge les pièces non conformes et rédige un rapport de revue.

> 🚧 **Projet en cours d'amélioration.** Ce dépôt est un premier prototype. Les prochaines étapes sont l'utilisation d'une base de modèles 3D réels et l'évaluation systématique de l'agent (voir [Feuille de route](#feuille-de-route)).




---

## Objectif

Ce projet m'a permis de mettre en pratique ce que j'apprends sur les agents IA, et de comprendre concrètement comment un agent peut assister la **revue et la validation d'une maquette numérique 3D**.

L'utilisateur écrit une demande en langage naturel, par exemple :

> *« Vérifie que les supports respectent un jeu minimal de 2 mm avec le socle, et détecte les interférences. »*

L'agent choisit les outils à utiliser, effectue les mesures, compare les résultats à la règle, colore en rouge les pièces non conformes dans la vue 3D et rédige un court rapport.

## Principe

```
Demande en langage naturel
        ↓
L'agent (LLM) choisit les outils à appeler
        ↓
Les outils calculent des mesures exactes (trimesh)
        ↓
L'agent compare les résultats à la règle
        ↓
Pièces non conformes en rouge + rapport de revue
```

**Le principe clé :** les **outils mesurent**, le **LLM décide** quoi vérifier et explique les résultats, et **l'humain valide** le rapport final.



## Technologies

| Outil | Rôle |
|---|---|
| [smolagents](https://github.com/huggingface/smolagents) (`CodeAgent`) | Framework d'agents : le LLM écrit du code Python qui appelle les outils |
| LLM open source via [Groq](https://groq.com) | Le « cerveau » de l'agent, accessible gratuitement par API |
| [trimesh](https://trimesh.org) + python-fcl | Création de la maquette, calcul des distances et des interférences |
| [Gradio](https://www.gradio.app) | Interface web avec visualiseur 3D |
| Google Colab | Environnement d'exécution |

## Outils de l'agent

| Outil | Description |
|---|---|
| `lister_pieces()` | Renvoie les pièces de l'assemblage et leurs dimensions en mm |
| `distance_min(a, b)` | Calcule la distance minimale exacte entre deux pièces |
| `detecter_interferences()` | Renvoie les paires de pièces qui se chevauchent |
| `surligner_pieces(noms)` | Indique les pièces à colorer en rouge dans la vue 3D |

## Maquette de test

La maquette contient un socle, deux supports et une vis, avec des **défauts placés volontairement** :

| Pièce | Situation | Résultat attendu (jeu min. 2 mm) |
|---|---|---|
| `support_a` | 1,2 mm du socle | ❌ Non conforme |
| `support_b` | 4,8 mm du socle | ✅ Conforme |
| `vis_1` | Traverse le socle | ❌ Interférence |

Les bonnes réponses étant connues, on peut vérifier que l'agent les trouve. En changeant la règle (1 mm, 5 mm), on vérifie aussi qu'il ne colore pas les pièces « au hasard ».

## Lancer le projet

1. Ouvrir le notebook dans Colab avec le badge ci-dessus.
2. Créer une clé API gratuite sur [console.groq.com](https://console.groq.com).
3. L'ajouter dans les **Secrets** de Colab (icône de clé) sous le nom `GROQ_API_KEY`.
4. Exécuter toutes les cellules, puis utiliser l'interface Gradio.

En local :

```bash
pip install -r requirements.txt
```

## Observations et limites

- **Les mesures sont fiables**, car elles viennent des outils et non du LLM.
- **Le LLM ajoutait parfois des recommandations techniques inventées.** Ses consignes sont donc limitées aux résultats des outils.
- **L'agent peut se corriger seul :** lors d'une erreur de format, il a reçu l'erreur comme observation et l'a corrigée à l'étape suivante.
- **L'agent est limité par ses outils :** il ne peut vérifier que ce que ses outils savent mesurer.
- La maquette est volontairement simple. Sur un assemblage réel, de nombreux contacts entre pièces sont normaux, et les règles doivent être plus précises.

## Feuille de route

Le projet est en cours d'amélioration.

**1. Base de modèles 3D réels**
- [ ] Charger des assemblages des [Khronos glTF Sample Assets](https://github.com/KhronosGroup/glTF-Sample-Assets) (ex. *2CylinderEngine*, *GearboxAssy*)
- [ ] Ajouter des défauts volontaires (pièces déplacées) pour connaître les résultats attendus
- [ ] Tester sur des assemblages CAO réels du [Fusion 360 Gallery Dataset](https://github.com/AutodeskAILab/Fusion360GalleryDataset)

**2. Évaluation de l'agent**
- [ ] Créer un jeu de scénarios de test avec les résultats attendus
- [ ] Mesurer le taux de réussite, les outils appelés, le nombre d'étapes, le temps et le coût en tokens
- [ ] Comparer `CodeAgent` et `ToolCallingAgent`, ainsi que plusieurs LLM

**3. Évolutions de l'agent**
- [ ] Lire les règles depuis un fichier d'exigences (RAG)
- [ ] Transformer l'agent en workflow contrôlé avec LangGraph
- [ ] Ajouter de nouveaux outils (angles, alignements, comparaison de versions)
- [ ] Analyser des captures de la vue 3D avec un modèle vision-langage

## Auteure

**Aya Boussaid** 

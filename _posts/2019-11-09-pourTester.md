---
title: Pour tester
author:
  name: Abou ismail tajmouti
  link: https://github.com/cotes2020
date: 2025-11-09 20:55:00 +0800
categories: [Blogging, Tutorial]
tags: [getting started]
pin: true
---

## Prerequisites

Follow the instructions in the [Jekyll Docs](https://jekyllrb.com/docs/installation/) to complete the installation of `Ruby`, `RubyGems`, `Jekyll`, and `Bundler`. In addition, [Git](https://git-scm.com/) is also required to be installed.

## Installation

### Creating a New Site

There are two ways to create a new repository for this theme:

- [**Using the Chirpy Starter**](#option-1-using-the-chirpy-starter) - Easy to upgrade, isolates irrelevant project files so you can focus on writing.
- [**Forking on GitHub**](#option-2-forking-on-github) - Convenient for custom development, but difficult to upgrade. Unless you are familiar with Jekyll and are determined to tweak or contribute to this project, this approach is not recommended.

#### Option 1. Using the Chirpy Starter

Create a new repository from the [**Chirpy Starter**][use-starter] and name it `<GH_USERNAME>.github.io`, where `GH_USERNAME` represents your GitHub username.

#### Option 2. Forking on GitHub

[Fork **Chirpy**](https://github.com/cotes2020/jekyll-theme-chirpy/fork) on GitHub and rename it to `<GH_USERNAME>.github.io`. Please note that the default branch code is in development.  If you want the site to be stable, please switch to the [latest tag][latest-tag] and start writing.

And then execute:

```console
$ bash tools/init.sh
```

> If you don't want to deploy your site on GitHub Pages, append option `--no-gh` at the end of the above command.
{: .prompt-info }

The above command will:

1. Removes some files or directories from your repository:
    - `.travis.yml`{: .filepath}
    - files under `_posts`{: .filepath}

2. If the option `--no-gh` is provided, the directory `.github`{: .filepath} will be deleted. Otherwise, set up the GitHub Action workflow by removing the extension `.hook`{: .filepath} of `.github/workflows/pages-deploy.yml.hook`{: .filepath}, and then remove the other files and directories in the folder `.github`{: .filepath}.

3. Removes item `Gemfile.lock` from `.gitignore`{: .filepath}.

4. Creates a new commit to save the changes automatically.

### Installing Dependencies

Before running for the first time, go to the root directory of your site, and install dependencies as follows:

```console
$ bundle
```

## Usage

### Configuration

Update the variables of `_config.yml`{: .filepath} as needed. Some of them are typical options:

- `url`
- `avatar`
- `timezone`
- `lang`

### Customing Stylesheet

If you need to customize the stylesheet, copy the theme's `assets/css/style.scss`{: .filepath} to the same path on your Jekyll site, and then add the custom style at the end of the style file.

Starting from [`v4.1.0`][chirpy-4.1.0], if you want to overwrite the SASS variables defined in `_sass/addon/variables.scss`{: .filepath}, create a new file `_sass/variables-hook.scss`{: .filepath} and assign new values to the target variable in it.

### Customing Static Assets

Static assets configuration was introduced in version `5.1.0`. The CDN of the static assets is defined by file `_data/assets/cross_origin.yml`{: .filepath }, and you can replace some of them according to the network conditions in the region where your website is published.

Also, if you'd like to self-host the static assets, please refer to the [_chirpy-static-assets_](https://github.com/cotes2020/chirpy-static-assets#readme).

### Running Local Server

You may want to preview the site contents before publishing, so just run it by:

```console
$ bundle exec jekyll s
```

Or run the site on Docker with the following command:

```console
$ docker run -it --rm \
    --volume="$PWD:/srv/jekyll" \
    -p 4000:4000 jekyll/jekyll \
    jekyll serve
```

After a while, the local service will be published at _<http://127.0.0.1:4000>_.

## Deployment

Before the deployment begins, check out the file `_config.yml`{: .filepath} and make sure the `url` is configured correctly. Furthermore, if you prefer the [**project site**](https://help.github.com/en/github/working-with-github-pages/about-github-pages#types-of-github-pages-sites) and don't use a custom domain, or you want to visit your website with a base URL on a web server other than **GitHub Pages**, remember to change the `baseurl` to your project name that starts with a slash, e.g, `/project-name`.

Now you can choose ONE of the following methods to deploy your Jekyll site.

### Deploy by Using Github Actions

For security reasons, GitHub Pages build runs on `safe` mode, which restricts us from using plugins to generate additional page files. Therefore, we can use **GitHub Actions** to build the site, store the built site files on a new branch, and use that branch as the source of the GitHub Pages service.

Quickly check the files needed for GitHub Actions build:

- Ensure your Jekyll site has the file `.github/workflows/pages-deploy.yml`{: .filepath}. Otherwise, create a new one and fill in the contents of the [sample file][workflow], and the value of the `on.push.branches` should be the same as your repo's default branch name.

- Ensure your Jekyll site has file `tools/deploy.sh`{: .filepath}. Otherwise, copy it from here to your Jekyll site.

- Furthermore, if you have committed `Gemfile.lock`{: .filepath} to the repo, and your runtime system is not Linux, don't forget to update the platform list in the lock file:

  ```console
  $ bundle lock --add-platform x86_64-linux
  ```

After the above steps, rename your repository to `<GH_USERNAME>.github.io` on GitHub.

Now publish your Jekyll site by:

1. Push any commit to remote to trigger the GitHub Actions workflow. Once the build is complete and successful, a new remote branch named `gh-pages` will appear to store the built site files.

2. Browse to your repository on GitHub. Select the tab _Settings_, then click _Pages_ in the left navigation bar, and then in the section **Source** of _GitHub Pages_, select the `/(root)` directory of branch `gh-pages` as the [publishing source][pages-src]. Remember to click <kbd>Save</kbd> before leaving.

    ![gh-pages-sources](/2019-08-09-getting-started/gh-pages-sources.png){: width="1580" height="250" }

3. Visit your website at the address indicated by GitHub.

### Manually Build and Deploy

On self-hosted servers, you cannot enjoy the convenience of **GitHub Actions**. Therefore, you should build the site on your local machine and then upload the site files to the server.

Go to the root of the source project, and build your site as follows:

```console
$ JEKYLL_ENV=production bundle exec jekyll b
```

Or build the site on Docker:

```console
$ docker run -it --rm \
    --env JEKYLL_ENV=production \
    --volume="$PWD:/srv/jekyll" \
    jekyll/jekyll \
    jekyll build
```

Unless you specified the output path, the generated site files will be placed in folder `_site`{: .filepath} of the project's root directory. Now you should upload those files to the target server.

## Upgrading

It depends on how you use the theme:

- If you are using the theme gem (there will be `gem "jekyll-theme-chirpy"` in the `Gemfile`{: .filepath}), editing the `Gemfile`{: .filepath} and update the version number of the theme gem, for example:

  ```diff
  - gem "jekyll-theme-chirpy", "~> 3.2", ">= 3.2.1"
  + gem "jekyll-theme-chirpy", "~> 3.3", ">= 3.3.0"
  ```
  {: .nolineno file="Gemfile" }

  And then execute the following command:

  ```console
  $ bundle update jekyll-theme-chirpy
  ```

  As the version upgrades, the critical files (for details, see the [Startup Template][starter]) and configuration options will change. Please refer to the [Upgrade Guide](https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide) to keep your repo's files in sync with the latest version of the theme.

- If you forked from the source project (there will be `gemspec` in the `Gemfile`{: .filepath} of your site), then merge the [latest upstream tags][latest-tag] into your Jekyll site to complete the upgrade.
The merge is likely to conflict with your local modifications. Please be patient and careful to resolve these conflicts.

[starter]: https://github.com/cotes2020/chirpy-starter
[use-starter]: https://github.com/cotes2020/chirpy-starter/generate
[workflow]: https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/.github/workflows/pages-deploy.yml.hook
[chirpy-4.1.0]: https://github.com/cotes2020/jekyll-theme-chirpy/releases/tag/v4.1.0
[pages-src]: https://docs.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
[latest-tag]: https://github.com/cotes2020/jekyll-theme-chirpy/tags


Absolument. En tant qu'architecte de solutions TI, votre démarche est parfaitement structurée et logique. Présenter une solution, c'est avant tout raconter une histoire business étayée par des faits techniques.

Voici un cadre complet que vous pouvez utiliser pour votre présentation, incluant les tableaux comparatifs et les arguments solides que vous recherchez.

---

### **Proposition d'Architecture de Solution : [Nom du Projet]**

**Rôle :** Architecte de Solution TI
**Objectif :** Recommander la plateforme logicielle optimale pour répondre aux besoins business, en accord avec la stratégie TI de l'entreprise.

---

### **Partie 1 : Analyse Stratégique - Best of Suite vs. Best of Breed**

Cette première analyse démontre pourquoi l'approche **Best of Suite** est la plus optimale pour notre contexte.

**Tableau Comparatif 1 : Best of Suite vs. Best of Breed**

| Critère d'Évaluation | Best of Suite (Recommandé) | Best of Breed | Analyse et Argumentation |
| :--- | :--- | :--- | :--- |
| **Coût Total de Possession (TCO)** | **Faible à Modéré** | **Élevé** | **Best of Suite :** Coûts de licence groupés, réduction du nombre de fournisseurs, support unifié. <br> **Best of Breed :** Licences multiples (souvent plus chères), coûts d'intégration élevés, support fragmenté. |
| **Ressources Humaines & Implémentation** | **Plus Simple** | **Complexe** | **Best of Suite :** Courbe d'apprentissage unique, équipes formées sur une plateforme cohérente. Déploiement plus rapide. <br> **Best of Breed :** Besoin d'experts spécialisés pour chaque outil. Risques et délais d'intégration importants. |
| **Time-to-Market & Agilité** | **Rapide** | **Lent** | **Best of Suite :** L'intégration native entre les modules permet un déploiement accéléré et des mises à jour coordonnées. <br> **Best of Breed :** Le temps de conception et de test des interfaces (APIs, ETL) rallonge considérablement le cycle de livraison. |
| **Cohérence & Intégration** | **Élevée** | **Variable (Risque élevé)** | **Best of Suite :** Expérience utilisateur unifiée, données cohérentes dans un "single source of truth", workflows natifs. <br> **Best of Breed :** Expérience utilisateur disparate, risques d'incohérence des données ("silos"), intégration souvent fragile. |
| **Flexibilité Fonctionnelle** | **Bonne** (répond aux besoins principaux) | **Excellente** (meilleure fonction par fonction) | **Best of Breed :** C'est son seul avantage majeur. Il permet de choisir l'outil absolument le meilleur dans chaque domaine. <br> **Cependant,** cet avantage est souvent annulé par la complexité et le coût de l'intégration, sauf pour des besoins hyper-spécialisés et critiques. |

**Conclusion de la Partie 1 :**
Pour notre projet, les impératifs business sont la **maîtrise des coûts, la rapidité de déploiement et la simplicité opérationnelle**. L'approche **Best of Suite** s'aligne parfaitement sur ces objectifs. Le léger compromis sur la fonctionnalité "ultime" dans certains domaines est largement compensé par les gains en efficacité, cohérence et agilité.

---

### **Partie 2 : Évaluation des Solutions Best of Suite**

Maintenant, comparons les trois produits de la suite retenue pour identifier la solution optimale.

**Grille d'Évaluation des Produits :**

Nous allons noter chaque produit (Produit A, B, C) sur une échelle de 1 à 5 (1=Faible, 5=Excellent) pour les critères suivants.

**Tableau Comparatif 2 : Analyse des Solutions Best of Suite**

| Critère d'Évaluation | Poids | Produit A | Produit B | Produit C | Commentaires |
|:---|:---:|:---:|:---:|:---:|:---|
| **1. Adéquation Fonctionnelle** | 30% | 4 | 5 | 3 | *Évalue dans quelle mesure le produit couvre les processus métier critiques.* <br> Le Produit B couvre 95% des besoins, le A 85% et le C seulement 70%. |
| **2. Sécurité & Conformité** | 25% | 5 | 4 | 5 | *Respect des normes (ISO 27001, SOC2), RGPD, politiques de sécurité de l'entreprise.* <br> A et C sont leaders en sécurité. B a un bon niveau mais nécessite une configuration avancée. |
| **3. Normes d'Architecture** | 15% | 4 | 3 | 5 | *Compatibilité avec le cloud (AWS/Azure/GCP), APIs ouvertes, conteneurisation, modèle de données.* <br> Le Produit C est le plus moderne et aligné avec notre cible d'architecture cloud-native. |
| **4. Coût & Valeur (ROI)** | 15% | 5 | 3 | 4 | *Coût total de licence, maintenance, et valeur business apportée.* <br> A est le plus compétitif, B est premium, C est dans la moyenne. |
| **5. Facilité d'Implémentation** | 10% | 4 | 3 | 2 | *Complexité de déploiement, qualité de la documentation, écosystème de partenaires.* <br> A est connu pour sa simplicité. C est réputé complexe à mettre en œuvre. |
| **6. Évolutivité & Support** | 5% | 3 | 5 | 4 | *Capacité à gérer la croissance, qualité du support technique.* <br> B, étant l'acteur majeur, offre le meilleur support et la meilleure évolutivité. |
| **→ Score Pondéré Total** | **100%** | **4.25** | **4.15** | **3.65** | **Calcul :** (Note * Poids) pour chaque critère, puis somme. |

**Analyse des Résultats du Tableau 2 :**

*   **Produit A (Score: 4.25)** : C'est le **meilleur compromis**. Il offre une excellente adéquation fonctionnelle, une sécurité de premier plan et le meilleur rapport qualité-prix. Son coût moindre et sa facilité d'implémentation renforcent les arguments initiaux du Best of Suite (réduction des coûts et rapidité).
*   **Produit B (Score: 4.15)** : C'est le leader fonctionnel, mais son coût plus élevé et sa légère complexité le placent en seconde position. Il pourrait être reconsideré si des besoins métier très spécifiques non couverts par A émergeaient.
*   **Produit C (Score: 3.65)** : Bien qu'aligné sur notre architecture cible, son inadéquation fonctionnelle et sa complexité d'implémentation en font un choix risqué pour les objectifs de time-to-market.

---

### **Recommandation Finale et Synthèse**

En ma qualité d'Architecte de Solutions TI, je recommande :

1.  **L'adoption de l'approche Best of Suite** pour ses avantages stratégiques en matière de coût, d'agilité et de simplicité.
2.  **La sélection du Produit A** comme la plateforme de la suite.

**Pourquoi le Produit A est la solution optimale ?**

*   **Alignement Business :** Il répond aux besoins critiques tout en étant **moins coûteux et plus rapide** à déployer, soutenant directement les objectifs de time-to-market et de maîtrise du budget.
*   **Robustesse Technique :** Il présente un **niveau de sécurité excellent** et respecte suffisamment nos normes d'architecture pour assurer une intégration pérenne et sécurisée.
*   **Simplicité Opérationnelle :** Sa **facilité d'implémentation** et son coût réduit en font le choix le plus **pragmatique et à moindre risque**.

**Prochaines Étapes :**
*   Organiser un proof-of-concept (POC) avec le Produit A pour valider l'adéquation fonctionnelle sur un cas d'usage concret.
*   Engager des discussions contractuelles avec l'éditeur du Produit A.
*   Élaborer un plan de projet et de migration détaillé.

Cette approche nous permet de livrer une valeur business rapide et durable, tout en respectant les contraintes et la stratégie de l'entreprise.


## Une précision à ajouter

Excellente précision stratégique ! Cela démontre une vision architecturale à long terme et une agilité très appréciée dans le monde TI actuel. Voici comment intégrer cette idée de manière convaincante dans votre présentation.

---

### **Partie 3 : Vision Évolutive et Feuille de Route Architecturale**

**Intégrer l'Agilité Future dans notre Choix Présent**

Votre point est crucial : le choix du **Best of Suite aujourd'hui n'équivaut pas à un verrouillage architectural à vie.** Il s'agit plutôt de la première phase d'une stratégie plus large.

#### **Le Concept : "Best of Suite avec Greffe Best of Breed"**

Nous recommandons une approche que nous pourrions appeler **"Backbone & Modules"** :
*   **La Colonne Vertébrale (The Backbone) :** Le produit Best of Suite (notre recommandation, le **Produit A**) sert de socle intégré et de source de vérité principale.
*   **Les Modules Spécialisés (The Modules) :** À l'avenir, nous pourrons greffer, via des APIs robustes, des solutions Best of Breed spécifiques pour des capacités où le besoin de spécialisation devient critique.

**Pourquoi cette vision est-elle réalisable et prudente ?**

1.  **Réduction du Risque Initial :** Commencer par la suite intégrée nous permet de livrer de la valeur rapidement et à moindre coût, sans attendre la conception complexe d'une architecture multi-best-of-breed dès le jour 1.
2.  **Focus sur le Core Business :** Pendant les 6 à 12 premiers mois, l'équipe se concentre sur l'adoption et la maîtrise de la plateforme centrale, plutôt que sur l'intégration de multiples systèmes.
3.  **Décision Différée et Éclairée :** Attendre nous permet d'identifier *réellement* quelles capacités méritent un investissement spécialisé. Nous prendrons cette décision future basée sur des données et des retours d'expérience concrets, et non sur des hypothèses.

#### **Tableau Comparatif 3 : Stratégies d'Évolution Architecturale**

| Scénario | Avantages | Inconvénients / Risques | Notre Recommandation |
| :--- | :--- | :--- | :--- |
| **1. Best of Suite Strict (Puriste)** | Simplicité, cohérence, coûts maîtrisés. | Manque de flexibilité à long terme si des besoins très spécialisés émergent. | **Idéal pour la Phase 1.** Pose les fondations. |
| **2. Best of Breed Intégral (Jour 1)** | Fonctionnalités optimales dans chaque domaine. | **Très risqué et coûteux initialement.** Complexité, time-to-market lent. | **Non recommandé.** C'est une sur-optimisation prématurée. |
| **3. Backbone & Modules (Notre Vision)** | **Agile et pragmatique.** Livre vite, apprend, et évolue là où cela compte. Mélange le meilleur des deux mondes. | Nécessite une gouvernance des APIs et une bonne hygiène des données. | **STRATÉGIE RECOMMANDÉE.** C'est le chemin le plus sage et le plus rentable. |

#### **Comment cela se concrétisera techniquement ?**

Le critère décisif entre le Produit A, B et C sera également leur **capacité à servir de plateforme d'intégration** :

*   **Vérification des APIs :** Le produit choisi (Produit A) doit offrir une suite d'APIs modernes (RESTful), bien documentées et sécurisées.
*   **Écosystème de Connectivité :** Il doit avoir un marché d'applications (app marketplace) ou des connecteurs pré-construits vers des plateformes spécialisées potentielles (ex : un outil d'IA spécialisé, un système de gestion de campagnes avancé, etc.).
*   **Architecture orientée Événements :** Idéalement, la plateforme supporte une architecture basée sur les événements, ce qui facilite l'intégration en temps réel avec des systèmes externes.

---

### **Synthèse et Recommandation Finale Revisitée**

**Recommandation : Adopter le Produit A selon une stratégie "Backbone & Modules".**

1.  **Phase 1 (Déploiement Immédiat - 0 à 12 mois) :**
    *   Implémenter le **Produit A (Best of Suite)** comme colonne vertébrale de notre paysage applicatif.
    *   Bénéficier de la **rapidité, de la simplicité et des coûts réduits**.
    *   Stabiliser l'environnement et former les équipes.

2.  **Phase 2 (Optimisation Continue - À partir du 6ème mois) :**
    *   **Auditer régulièrement** les capacités du Produit A par rapport aux besoins émergents.
    *   **Identifier les lacunes fonctionnelles** qui justifient un investissement supplémentaire.
    *   **Évaluer des solutions Best of Breed candidates** uniquement pour ces lacunes spécifiques.

3.  **Phase 3 (Évolution Ciblée - Si et seulement si le besoin est avéré) :**
    *   **Greffer la solution spécialisée** sur la colonne vertébrale du Produit A via ses APIs.
    *   Maintenir la logique métier centrale et les données maîtres dans le Produit A.

**Conclusion :**

En choisissant cette voie, nous ne fermons aucune porte. Au contraire, nous **empruntons le chemin le plus efficace et le moins risqué pour livrer de la valeur immédiate, tout en nous donnant la flexibilité architecturale pour nous spécialiser à l'avenir, de manière ciblée et justifiée.** C'est la démarche d'une architecture TI moderne et alignée avec le business.
---------------------------------------------------------------------------------------------------
ATELIER ANSIBLE
---------------------------------------------------------------------------------------------------
L'idee en 30 secondes : Dans cet atelier, vous allez apprendre a **automatiser le deploiement d'un serveur web avec Ansible**, directement depuis GitHub **Codespaces**, sans infrastructure complexe. L'objectif est de comprendre comment **decrire un etat cible** (installer, configurer, deployer) et laisser l'outil l'appliquer automatiquement, de maniere reproductible et idempotente. On passe ainsi d'une logique manuelle a une logique DevOps industrialisee.

**Architecture cible :** Ci-dessous, voici l'architecture cible souhaitee.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/8064eb95-da73-4bdd-9ef2-a7cebbbd71c8" />

---------------------------------------------------------------------------------------------------
Sequence 1 : Codespace de Github
---------------------------------------------------------------------------------------------------
Objectif : Creation d'un Codespace Github
Difficulte : Tres facile (~5 minutes)
---------------------------------------------------------------------------------------------------

**Faites un Fork de ce projet**. Si besoin, voici une video d'accompagnement pour vous aider a "Forker" un Repository Github : [Forker ce projet](https://youtu.be/p33-7XQ29z0)

Ensuite depuis l'onglet **[CODE]** de votre nouveau Repository, **ouvrez un Codespace Github**.

------------------------------------------------
Sequence 2 : Creation du votre environnement de travail
------------------------------------------------
Objectif : Creer votre environnement de travail
Difficulte : Simple (~10 minutes)
---------------------------------------------------------------------------------------------------

Vous allez dans cette sequence mettre en place votre environnement et les logiciels Ansible. Depuis le terminal de votre Codespace copier/coller les codes ci-dessous etape par etape :

**Installation de Ansible et Nginx**

```bash
sudo apt update
sudo apt install -y ansible curl
```

**Verifier l'environnement**

```bash
ansible --version
curl --version
```

**Tester la cible locale**

```bash
ansible -i inventory.ini local -m ping
```

**Executer le playbook**

```bash
ansible-playbook -i inventory.ini playbook.yml
```

**Verifier le resultat**

```bash
curl http://localhost
```

Recuperation de l'URL de votre serveur Nginx. Votre serveur Nginx (et sa page Web) est deploye dans Codespace. Pour obtenir votre URL cliquez sur l'onglet **[PORTS]** dans votre Codespace (a cote de Terminal), ouvrez le port 80 et rendez public votre port (Visibilite du port). Ouvrez l'URL dans votre navigateur et c'est termine.

---------------------------------------------------------------------------------------------------
Sequence 3 : Exercices
---------------------------------------------------------------------------------------------------
Difficulte : Facile (~30 minutes)

### Exercice 1 : Customisation de la page d'accueil

Customisez la page d'accueil de votre site Web en ajoutant la ligne suivante dans votre fichier index.html

```html
<h1>Deploiement realise par Lucas RAGOT</h1>
```

Cette ligne a ete ajoutee dans le fichier `files/index.html`.

### Exercice 2 : Parametrer dynamiquement votre serveur avec Ansible

Voici le contenu de votre nouveau fichier index.html avec les variables Ansible :

```html
<!DOCTYPE html>
<html>
<head>
<title>{{ page_title }}</title>
</head>
<body>
<h1>{{ page_title }}</h1>
<p>Deploye par : {{ author }}</p>
<p>Utilisateur systeme : {{ app_user }}</p>
<h1>Deploiement realise par Lucas RAGOT</h1>
</body>
</html>
```

Le playbook a ete modifie avec les variables suivantes :
- **page_title** : "Serveur deploye avec Ansible"
- **author** : "Lucas RAGOT"
- **app_user** : "lucas"

Le module `copy` a ete remplace par le module `template` afin d'interpreter les variables Ansible (syntaxe `{{ }}`).

---------------------------------------------------------------------------------------------------
Sequence 4 : Questions
---------------------------------------------------------------------------------------------------
Difficulte : Moyenne (~45 minutes)

### Question 1 : Pourquoi Ansible est-il qualifie d'outil "declaratif" ?

Ansible est qualifie d'outil declaratif car on **decrit l'etat final souhaite** du systeme, et non les etapes pour y parvenir. On dit a Ansible "je veux que nginx soit installe et demarre", sans preciser comment l'installer pas a pas. Ansible se charge de determiner les actions necessaires pour atteindre cet etat. De plus, Ansible est **idempotent** : si l'etat souhaite est deja present (par exemple nginx est deja installe), Ansible ne fait rien. On peut executer le meme playbook plusieurs fois sans risque d'effets de bord, ce qui garantit la coherence et la stabilite de l'infrastructure.

En pratique, dans notre `playbook.yml`, nous avons ecrit :
```yaml
- name: Installer nginx
  apt:
    name: nginx
    state: present
```
Nous ne disons pas "execute apt-get install nginx". Nous disons "nginx doit etre present". C'est la logique declarative.

---

### Question 2 : Pourquoi l'utilisation de variables est-elle essentielle dans un playbook ?

L'utilisation de variables dans un playbook Ansible est essentielle pour plusieurs raisons :

**1. Reutilisabilite** : une variable comme `{{ page_title }}` ou `{{ author }}` permet d'utiliser le meme playbook pour differents environnements (DEV, PROD) ou differents utilisateurs sans modifier le code. On change simplement la valeur de la variable.

**2. Maintenabilite** : si une valeur change (par exemple le chemin d'installation), il suffit de modifier la variable en un seul endroit plutot que de chercher et remplacer dans tout le playbook.

**3. Flexibilite** : les variables peuvent etre definies dans l'inventaire, dans le playbook, dans des fichiers de variables, ou passees en ligne de commande. Cela permet une grande souplesse de configuration.

**4. Lisibilite** : un playbook avec des variables est plus clair et plus facile a comprendre. Le nom de la variable porte du sens (ex: `page_dest`, `app_user`).

Dans notre projet, les variables `page_title`, `author` et `app_user` permettent de personnaliser le deploiement pour chaque environnement (DEV ou PROD) via l'inventaire.

---

### Question 3 : En quoi Ansible facilite-t-il la gestion de plusieurs serveurs ?

Ansible facilite la gestion de plusieurs serveurs grace a plusieurs mecanismes :

**1. L'inventaire** : le fichier `inventory.ini` liste tous les serveurs cibles, regroupes en groupes logiques (ex: `[dev]`, `[prod]`, `[web]`, `[db]`). Il suffit d'ajouter un serveur dans l'inventaire pour qu'il soit pris en compte.

**2. L'execution parallele** : Ansible peut executer les taches sur plusieurs serveurs simultanement (en parallele), ce qui reduit le temps de deploiement. Le parametre `forks` controle le nombre de connexions paralleles.

**3. Sans agent** : Ansible se connecte aux serveurs via SSH, sans necessiter l'installation d'un agent sur chaque machine cible. Cela simplifie enormement la gestion d'un grand parc.

**4. L'idempotence** : on peut relancer le meme playbook sur 100 serveurs sans craindre des effets de bord. Seuls les serveurs qui ne sont pas dans l'etat souhaite seront modifies.

**5. Les groupes et variables de groupes** : on peut appliquer des configurations differentes a des groupes de serveurs (ex: configuration DEV vs PROD) via les variables de groupe dans l'inventaire.

---

### Question 4 : Quels sont les avantages et les limites d'Ansible dans un contexte DevOps ?

**Avantages :**

- **Sans agent** : aucun logiciel a installer sur les serveurs cibles, Ansible utilise SSH. Cela simplifie l'adoption et la securite.
- **Courbe d'apprentissage douce** : la syntaxe YAML des playbooks est lisible et accessible, meme pour des profils non-developpeurs.
- **Idempotence** : les playbooks peuvent etre re-executes sans risque, garantissant la stabilite des environnements.
- **Communaute et modules** : Ansible dispose d'un ecosysteme tres riche (Ansible Galaxy) avec des milliers de roles et modules pre-faits.
- **Integration CI/CD** : Ansible s'integre facilement dans des pipelines CI/CD (GitHub Actions, Jenkins, GitLab CI).
- **Infrastructure as Code** : les playbooks sont versionnables dans Git, permettant un suivi des changements et une collaboration en equipe.

**Limites :**

- **Performance** : pour un tres grand nombre de serveurs (des centaines), Ansible peut etre lent car il se connecte en SSH a chaque machine. Des outils comme Terraform ou Salt peuvent etre plus adaptes.
- **Gestion d'etat complexe** : Ansible ne garde pas d'etat de l'infrastructure (contrairement a Terraform). Il est difficile de detecter des changements manuels sur les serveurs.
- **Debugging difficile** : les erreurs dans les playbooks complexes peuvent etre difficiles a diagnostiquer.
- **Pas de gestion native des dependances** : l'ordre d'execution des taches doit etre gere manuellement.
- **Windows** : le support de Windows est moins mature que Linux/Unix.

---

### Question 5 : Quelle est la difference entre les modules copy et template dans Ansible ?

| Critere | Module `copy` | Module `template` |
|---------|----------------|---------------------|
| **Fonction** | Copie un fichier statique tel quel | Copie un fichier en interpretant les variables Jinja2 |
| **Variables** | Les variables `{{ }}` ne sont PAS interpretees | Les variables `{{ }}` SONT remplacees par leurs valeurs |
| **Format source** | Fichier quelconque (.html, .conf, .txt...) | Fichier `.j2` (Jinja2) ou tout fichier avec syntaxe Jinja2 |
| **Usage typique** | Copier un fichier binaire ou un fichier fixe | Generer des fichiers de configuration dynamiques |
| **Exemple** | Copier une image, un script fixe | Generer un `nginx.conf` ou un `index.html` personnalise |

**Exemple concret** :

Avec `copy`, si le fichier source contient `{{ author }}`, cette chaine sera copiee litteralement (non remplacee).

Avec `template`, si le fichier source contient `{{ author }}`, Ansible remplacera cette variable par sa valeur (ex: "Lucas RAGOT").

Dans notre projet, nous avons utilise le module `template` pour deployer notre `index.html` afin que les variables `{{ page_title }}`, `{{ author }}` et `{{ app_user }}` soient correctement remplacees lors du deploiement.

---------------------------------------------------------------------------------------------------
Sequence 5 : Atelier
---------------------------------------------------------------------------------------------------
Difficulte : Moyenne (~1 heure)

### Structurer votre deploiement Ansible afin de pouvoir choisir entre un role DEV ou un role PROD

Le fichier `playbook.yml` a ete modifie pour supporter les deux roles. Le fichier `inventory.ini` definit les groupes `[dev]` et `[prod]` avec leurs variables respectives.

**Deploiement en DEV**
```bash
ansible-playbook -i inventory.ini playbook.yml --limit dev
```

Variables appliquees :
- `app_name`: "Application DEV"
- `env`: "dev"
- `author`: "Lucas RAGOT"
- `page_title`: "Serveur deploye avec Ansible - DEV"
- `app_user`: "lucas"

**Deploiement en PROD**
```bash
ansible-playbook -i inventory.ini playbook.yml --limit prod
```

Variables appliquees :
- `app_name`: "Application PROD"
- `env`: "prod"
- `author`: "Equipe Ops"
- `page_title`: "Serveur deploye avec Ansible - PROD"
- `app_user`: "lucas"

**Comment ca fonctionne ?**

Le fichier `inventory.ini` definit deux groupes avec des variables differentes via les sections `[dev:vars]` et `[prod:vars]`. Quand on lance le playbook avec `--limit dev`, Ansible n'execute le playbook que sur les hotes du groupe `[dev]` et utilise les variables definies dans `[dev:vars]`. Le module `template` injecte ces variables dans le fichier `index.html`, generant une page personnalisee selon l'environnement.

---------------------------------------------------------------------------------------------------
Zoom sur votre environnement Codespace
---------------------------------------------------------------------------------------------------

Codespace est un outil propose par GitHub soumis a quota (4$/mois). Si vous depassez votre quota mensuel vous ne serez plus en mesure de pouvoir utiliser Codespace. C'est pourquoi a la fin de votre atelier, pensez a supprimer votre Codespace apres avoir mis a jour votre GitHub.

Vous pouvez voir l'etat de vos Codespace ici : https://github.com/codespaces

---------------------------------------------------------------------------------------------------
Evaluation
---------------------------------------------------------------------------------------------------

Cet atelier ANSIBLE, note sur 20 points, est evalue sur la base du bareme suivant :

- Mise en oeuvre (2 points)
- Exercice N1 - Customisation de la page d'accueil (2 points)
- Exercice N2 - Parametrer dynamiquement votre serveur avec Ansible (2 points)
- Questions + Qualite du Readme (lisibilite, erreur, ...) (5 points)
- Atelier - Roles DEV et PROD (6 points)
- Processus travail (quantite de commits, coherence globale, interventions externes, ...) (3 points)

**Realise par : Lucas RAGOT**

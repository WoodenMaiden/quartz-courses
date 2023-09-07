---
title: Introduction à Terraform
enableToc: true
openToc: true
tags:
  - terraform 
  - devops 
  - cloud 
  - architecture
---
**Outils/Compétences**

![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white) ![Git](https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white) 


**Branche**: [tf-intro](https://github.com/WoodenMaiden/exercises/tree/tf-intro)

> [!info]- (Optionnel) Point sur la révocation de la license FOSS: 7 Septembre 2023
> Après 9 ans de croissance en tant qu'outil open-source, Terraform s'est vu révoquer sa license Mozilla Public License (MPL v2.0) le 10 Août 2023 pour la Business Source License (BUSL v1.1). 
>
> Avec ce choix brutal et sans justifications, Hashicorp (les développeurs de Terraform) mettent en danger des millions de personnes utilisant l'outil, allant du simple étudiant à la grande multinationale. À cause de cette license et des conditions vagues donnés par Hashicorp, certains projets et business s'axant autour de Terraform peuvent se retrouver devant les tribunaux.
> 
> Un [manifeste](https://opentf.org/) à été publié par la communauté OpenTF,  annonçant faire un fork de Terraform si Hashicorp ne revenait pas sur sa décision, comme vous pouvez le deviner Hashicorp n'ayant pas répondu à l'appel, [les menaces on été mises à exécution](https://help.obsidian.md/Editing+and+formatting/Callouts).
>
> En soit rien ne vous empêche d'utiliser Terraform, mais faites attention à ce que vos projets rentrent dans les conditions d'utilisation très vagues écrites par Hashicorp. Pour ce qui est d'OpenTF, les développeurs assurent que leur fork permettra d’exécuter les configurations écrites avant ce dernier, mais le projet est encore jeune et reste ouvert [aux contributions](https://github.com/opentffoundation/opentf/blob/main/CONTRIBUTING.md). Ce cours ayant été écrit juste avant le changement de license vous n'avez pas à vous en soucier.

- - -
# Présentation

Terraform est un outil d'IAC (Infrastructure As Code), son but est de déclarer une architecture, de l'instancier et de la détruire.

> ⚠️ L'outil a pour but de configurer une ARCHITECTURE, pas une machine, pour ça il y a [Ansible](https://www.ansible.com/). néanmoins il est commun de coupler les deux, Ansible configurant les machines virtuelles créées par Terraform. 

Terraform est très souvent utilisé pour créer des architectures sur un [cloud provider](https://www.techtarget.com/searchitchannel/definition/cloud-service-provider-cloud-provider) tel que AWS et Google Cloud, sur [Kubernetes](https://kubernetes.io/docs/concepts/overview/), sur [Heroku](https://www.heroku.com/) et bien d'autres. Dans ce TP nous l'utiliserons avec une instance locale de [Dokku](https://dokku.com/) une alternative gratuite de Heroku et [MongoDB Atlas](https://www.mongodb.com/atlas/database).

## Pourquoi ?

Les différents modèles de cloud computing (listés ci-dessous) proposent chacun une répartition des responsabilités différentes entre le client et la plateforme, ce qui veut dire que c'est le rôle du client d’assurer certaines parties de son architecture.

![[images/cloud_service_models.png]]

Créer une architecture sous forme de code avec Terraform garantit plusieurs choses: 

- **Automatisation des déploiements ⚙️**: L'approvisionnement de l'infrastructure par les interfaces graphiques ou de scripts Bash est lent, source d'erreurs, et [scale](https://www.suse.com/suse-defines/definition/scalability/) très mal.

- **Des résultats prévisibles et reproductibles 🔭**: le code Terraform est relativement simple (pas de notions de classes, de fonctions etc.) et ne se résume la plupart du temps qu'à de la description de ressources.

- **La possibilité de créer mais aussi de détruire 💣**: Terraform utilise [[#Les providers]] qui peuvent chacun créer mais aussi détruire les ressources qu'ils ont créé. 

- **Une architecture évolutive 🧬**: Terraform est un langage, il existe donc les notions de variables que nous verrons plus tard dans le TP, cela permet de paramétrer l'architecture comme on le souhaite en changeant par exemple le mot de passe des bases de données, les régions dans lesquelles seront déployées les ressources etc.

- **La possibilité de versionner son architecture 📦**: Il s'agit de loin de <mark style="background: #FFF3A3A6;">l'argument le plus important de l'IAC</mark>: vu qu'il s'agit de code on peut le versionner avec [git](https://git-scm.com/), [subversion](https://subversion.apache.org/) ou autres gestionnaires de version, ce qui permet une grande liberté quant à la gestion de notre architecture. 
  Faut il changer des network policies ? Faisons un commit l'ajoutant. S'avère t-il que cet ajout casse notre application ? Vite un `git revert` !

Pour commencer à travailler avec Terraform, créez un nouveau répertoire avec un fichier ayant l’extension `.tf` avec ce contenu: 

```hcl
terraform {

}
```

Lancez `terraform init` et on peut commencer à utiliser Terraform!
## Comment fonctionne Terraform?

### La CLI

![[images/tf_flow.png]]
La CLI de terraform propose des commandes pour interagir avec son architecture

- `init` -> Crée les fichiers nécessaires pour Terraform et installe les providers
- `plan` -> Terraform va exécuter le code HCL des fichiers `.tf` mais ne va pas faire appel aux providers, cette commande permet d'avoir un aperçu des ressources qui seront générées.
- `apply` -> Terraform lance la phase de `plan` décrite précédemment, après confirmation, les providers s'occupent de créer les ressources générées.
  Grâce au fichier de state que nous évoquerons plus tard Terraform sera capable de créer les ressources nouvellement déclarées, de mettre à jour celles qui ont changée et de supprimer celles qui ont été supprimées.
  Il est possible de forcer la re-création d'une ressource avec l'option `-remplace="<type_de_ressource>.<nom_de_la_ressource>"` 
- `destroy` -> ["I think we all know where this is going"](https://media1.giphy.com/media/oe33xf3B50fsc/giphy.gif?cid=ecf05e47b61oing7kc29fwjmqsm1dmn6r7v0k3ubl54us292&ep=v1_gifs_search&rid=giphy.gif&ct=g)

### Les providers

Terraform en lui même n'a aucun idée de comment créer une ressource tel qu'un bucket S3, un Ingres Kubernetes, une application sur Heroku/Dokku. Pour créer ces dites ressources il va utiliser des Providers, il s'agit concrètement de plugin écrits en go qui vont fournir les `ressources` que vous voulez créer. Après que Terraform exécute le code HCL (HashiCorp Language)  pour carlculer les attributs des ressources, il va laisser le provider faire les appels aux APIs nécessaires à la création de la ressource.

![[images/tf_providers.png]]

Ces providers sont recensés et documentés en ligne sur https://registry.terraform.io, vous y trouverez pour chacun d'entre eux leur documentation.  

Pour ajouter un provider vous avez juste à le déclarer dans le bloc `terraform`, et de le configurer avec le bloc `provider "<nom du provider>"`, les options de configurations sont trouvables dans le registre.

```hcl
terraform {
	required_providers {
		aws = {
			source = "hashicorp/aws"
			version = "5.11.0"
		}
	}
}

provider "aws" {
	# options de configuration 
} 

# et maintenant on peut utiliser les ressources que le provider aws propose ☁️
```

### HCL

#### Les ressources

Les ressources représentent ce qu'un provider peut créer, cela peut aller d'une machine virtuelle, d'une règle réseau, d'une politique d'IAM etc.

Une ressource se définit ainsi en HCL.

```hcl
resource "type_de_ressource" "identifiant_de_ma_ressource" {
	argument = valeur
	# autres arugments mentionnés dans la documentation du provider
}
```

Les différents types de ressources sont disponibles grâce aux providers et chaque provider liste ses ressources dans sa documentation. L'identifiant lui sert à différencier les ressources d'un même type.

Et c'est là où ça commence à être amusant, car ces arguments ont des types d'entrées, donc si on a des [types](https://developer.hashicorp.com/terraform/language/expressions/type-constraints) on a des opérations à appliquer dessus 🙃. Et c'est là que commence la partie "programmation" de terraform. Pourquoi ne pas attribuer le résultat d'une fonction? 

```hcl
resource "local_file" "foo" {
  content  = lower("HELLO") 
  #                 👇 oui on parlera des modules plus tard, vous n'avez rien vu 
  filename = "${path.module}/foo.bar"

  # ah oui, et ${} permet la substitution des variables
  # https://en.wikipedia.org/wiki/String_interpolation
}
```

Et pourquoi pas faire référence à la valeur d'une propriété appartenant à une autre ressource ? C'est possible en l'invoquant sous la forme `<TYPE DE RESSOURCE>.<NOM>.<ATTRIBUT>`: 

```hcl
resource "aws_vpn_connection" "example_connection" {
  # ...
  transit_gateway_id  = "donnez_moi_un_tag"
  # ...
}

resource "aws_ec2_tag" "example_tag" {
  resource_id = aws_vpn_connection.example_connection.transit_gateway_attachment_id
  key         = "Name"
  value       = "Hello World"
}
```

> 💬 Mais cela induit donc une dépendance entre les ressources n'est ce pas ?

-- Oui, mais Terraform est assez malin pour le gérer seul! Il est rare que vous ayez à définir le méta-argument `depends-on` 😉

##### "Des méta-arguments ?"

Hé oui, il existe des arguments communs à tous les bloc ressources, peu importe leur provenance qui vont modifier le comportement des ressources, [la doc](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on) évoque ces derniers mais ceux qui vont nous intéresser sont `for_each` et `count`. 

Concrètement ces deux là ont un objectif commun: créer plusieurs instance d'une ressource, mais ils le font différemment. `count` lui créée bêtement des copies conformes de la ressource en question en leur ajoutant un compteur, `for_each` lui prend une collection de `string` une `map` ou un `objet` et peut attribuer les clés et valeurs de ce dernier aux copies

```hcl
resource "local_file" "foocount" {
  count    = 2 # Nombre de copies
  content  = "Je suis la copie n°${count.index}" 
  filename = "${path.module}/copie${count.index}.txt"
  
  # copie1.txt => "Je suis la copie n°1" 
  # copie2.txt => "Je suis la copie n°2" 
}

resource "local_file" "bareach" {
  for_each = toset(["bar1", "bar2"])
  content  = "je suis le fichier ${each.key}"
  filename = "${path.module}/${each.key}.txt"
  
  # bar1.txt => bar1
  # bar2.txt => bar2
}

resource "local_file" "totoeach" {
  for_each = {
      tick = "tock"
      ding = "dong"
  }

  content  = each.value
  filename = "${path.module}/${each.key}.txt"

  # tick.txt => tock
  # ding.txt => dong
}
```

On utilise donc `count` pour créer des copies conformes et `for_each` pour des copies avec des variantes.

À savoir qu'il est possible d'accéder aux différents membres d'un count avec son index (par ex `resource.0`) et d'accéder à la totalité des variantes avec `*` à la place de l'index
#### Les variables

Tout bon langage de programmation qui se respecte intègre un système de variables, c'est le cas de Terraform qui intègre le bloc variable 

```hcl
variable "nom_de_ma_variable" {
  description = "une string pour décrire la variable (optionnel)"
  type        = type # le type de votre variable 
  default     = "Une valeur par défaut (optionnel)"
  sensitive   = false # n'affiche pas la variable sur la sortie standart
  nullable    = false # est ce que la variable à le droit d'être nulle
  
  validation { # ce bloc stoppe l’exécution du code si la condition donnée n'est pas remplie.
    condition     = # mettez ici une expression évaluée à true ou false
    error_message = "Un message d'erreur"
  }
}
```

Ces variables sont plus à voir comme des inputs constant que l'utilisateur donnerait sur une ligne de commande dans le sens ou ces dernières ne peuvent être ré-assignées.

Pour renseigner ces variables on à trois manières de faire:

- **La ligne de commande** :  avec l'option var `terraform apply -var="nom_de_ma_variable=ma_valeur"`
- **Les variables d'environnement**: En exportant une variable d'environnement préfixée par  `TF_VAR_`, ex: `TF_VAR_myvar="value"`
- **Le fichier .tfvars**: créez un fichier finissant par l’extension .tfvars et précisez le avec l'option CLI `-var-file`, dans ce dernier affectez les valeurs (pas besoin si le fichier s'appelle terraform.tfvars), 
```hcl
mosse = "Moose"
goose = 12
beef = {
	grilled = true
}
```

On peut donc utiliser ces variables dans notre code HCL avec `var.nom_de_ma_variable`
 
> [!attention]
> Pensez à ne pas commit ce fichier, penser à l'ignorer avec un `.gitignore` par exemple

> [!docs]
> Avec la myriade d’expressions et d'opérations disponibles sur les types, il est plus pertinent d'aller les regarder par vous même dans [la documentation à se sujet](https://developer.hashicorp.com/terraform/language/expressions) 

#### Les sources de données

Parfois vous aurez besoin de données relatives à une ressource qui n'est disponible qu'auprès de la cible d'un provider, par exemple l'identifiant d'une organisation, les providers proposent en plus des ressources des bloc `data` qui servent justement à accéder à ce genre de données.

```hcl
data "type" "nom" {
	contrainte_une  = true
	contrainte_deux = "value"
	# autres contraintes
}
```

Dans ce type de bloc (tout aussi bien documenté) les arguments données sont des contraintes que la donnée doit respecter pour qu'elle soit retournée tout comme le feraient la clause `WHERE` et `HAVING` en SQL. Et tout comme les ressources, elles sont identifiées par le couple type/nom et ont des propriété accessibles avec `data.<type_de_data>.<nom>.<attribut>` 
### Les modules

![[images/tf_modules.svg]]
Un projet Terraform est doté de modules, les modules sont des ensembles de fichiers Terraform qui prennent des variables en entrée et qui renvoie des variables de sorties, ils peuvent aussi bien provenir du registre terraform, de git ou encore de sous-répertoires.

On définit les variables d'entrées comme le chapitre [[#Les variables]] (sans les initialiser!), les valeurs de sortie elle sont définies grâce au mot clé `output`:

```hcl
output "db_password" {
  value       = # Mettez ici la valeur retournée
  description = "Mettez ici une description de l'output"
  sensitive   = true # définit si elle sera affichée dans la console

  precondition {
	  condition = # mettez ici une expression évaluée à true ou false
	  error_message = "un message d'erreur"
  }
}
```

Le bloc `précondition` stoppe l’exécution du code si la condition donnée n'est pas remplie.

Pour importer et utiliser un module on utilise le mot clé `module`:

```hcl
module "mon_module" {
  source  = "source" # https://developer.hashicorp.com/terraform/language/modules/sources
  version = "X.X.X"
  
  variable_une = "valeur 1"
  variable_deux = true
  variable_trois = type.nom.attribut
  # le reste des variables définies par le module
}
```

Comme vous avez du le deviner on peut accéder aux valeurs `output` avec `<nom_du_module>.<nom_de_la_valeur_output>`.
### Le state

#### Qu'est ce que c'est ? 

Terraform n'a pas de moyen de vérifier l'état de l'architecture en temps réel avec les providers, quand lancez `terraform apply` un fichier `.tfstate` va être écrit et mis a jour.

> [!error] DANGER
> Ce fichier contient l'ENTIÈRETÉ de l'état actuel de l'architecture y compris toutes les variables et autres données sensibles en clair, donc par pitié tout comme le fichier de variables, ne le comittez pas.

Hé oui! si Terraform sait quelles ressources il doit modifier et les quelles il doit créer/supprimer c'est en comparant la sortie du code HCL et ce qui il y a dans ce state.
#### Problématiques

> [!dialogue]  Remarque
> >"Mais donc si je le supprime par pur accident, Terraform ne pourra plus savoir où il en est ! De plus si on est plusieurs dans mon équipe à faire des `apply` comment on peut garantir la cohérence entre nos states"
> 
> On peut répondre à ces deux problématiques avec la décentralisation du state grâce au bloc `backend`.

```hcl
terraform {
  backend "<type>" {
    #...
  }
}
```

Un `backend` désigne la manière dont sera stocké le fichier state, il est donc totalement possible de le mettre dans une base de données et d'utiliser les commandes de [push et de pull](https://developer.hashicorp.com/terraform/cli/state/recover) pour récupérer et mettre à jour le state, il existe une [dizaine de backends disponibles](https://developer.hashicorp.com/terraform/language/settings/backends/configuration) allant de la base de données PostgreSQL au bucket S3, c'est à vous de voir.

![[images/tf_backend.png]]

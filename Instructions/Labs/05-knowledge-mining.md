---
lab:
  title: Créer une solution d'exploration des connaissances
  description: Utilisez Azure AI Search pour extraire les informations clés des documents et faciliter leur recherche et leur analyse.
---

# Créer une solution d'exploration des connaissances

Dans cet exercice, vous utilisez AI Search pour indexer un ensemble de documents gérés par Margie's Travel, une agence de voyages fictive. Le processus d'indexation consiste à utiliser des compétences en IA pour extraire des informations clés afin de les rendre consultables, et à générer une base de connaissances contenant des données pour une analyse plus approfondie.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications similaires à l'aide de plusieurs SDK spécifiques à un langage, notamment :

- [Bibliothèque cliente Azure AI Search pour Python](https://pypi.org/project/azure-search-documents/)
- [Bibliothèque cliente Azure AI Search pour Microsoft .NET](https://www.nuget.org/packages/Azure.Search.Documents)
- [Bibliothèque cliente Azure AI Search pour JavaScript](https://www.npmjs.com/package/@azure/search-documents)

Cet exercice prend environ **40** minutes.

## Créer des ressources Azure

La solution que vous allez créer pour Margie's Travel nécessite plusieurs ressources dans votre abonnement Azure. Dans cet exercice, vous allez les créer directement dans le portail Azure. Vous pouvez également les créer à l'aide d'un script, d'un modèle ARM ou BICEP, ou encore créer un projet Azure AI Foundry qui inclut une ressource Azure AI Search.

> **Important** : Vos ressources Azure doivent être créées au même emplacement !

### Créer une ressource Recherche Azure AI

1. Dans un navigateur web, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Sélectionnez le bouton **&#65291;Créer une ressource** ., recherchez `Azure AI Search`, et créez une ressource **Azure AI Search** avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Nom du service** : *Un nom valide pour votre ressource de recherche*
    - **Emplacement** : *N’importe quel emplacement disponible*
    - **Niveau tarifaire** : Gratuit

1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Passez en revue la page **Vue d’ensemble** du panneau de votre ressource Recherche Azure AI dans le Portail Azure. Ici, vous pouvez utiliser une interface visuelle pour créer, tester, gérer et surveiller les différents composants d’une solution de recherche ; y compris les sources de données, les index, les indexeurs et les ensembles de compétences.

### Créez un compte de stockage.

1. Revenez à la page d’accueil, puis créez une **ressource de compte** de stockage avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Le même groupe de ressources que vos ressources Azure AI Search et Azure AI Services*
    - **Nom du compte de stockage** : *Un nom valide pour votre ressource de stockage*
    - **Région** : * la même région que votre ressource Recherche Azure AI*
    - **Service principal** : stockage Blob Azure ou Azure Data Lake Storage Gen2
    - **Performances** : standard
    - **Redondance** : stockage localement redondant (LRS)

1. Attendez la fin du déploiement, puis accédez à la ressource déployée.

    > **Conseil** : Gardez la page du portail du compte de stockage ouverte, vous en aurez besoin pour la procédure suivante.

## Télécharger des documents vers Azure Storage

Votre solution d'exploration des connaissances extraira des informations à partir de brochures touristiques stockées dans un conteneur blob Azure Storage.

1. Dans un nouvel onglet de navigateur, téléchargez [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip) à partir de `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` et enregistrez-le dans un dossier local.
1. Extrayez le fichier *documents.zip* téléchargé et affichez les fichiers de brochure de voyage qu’il contient. Vous allez extraire et indexer des informations à partir de ces fichiers.
1. Dans l’onglet du navigateur contenant la page Portail Azure de votre compte de stockage, dans le volet de navigation de gauche, sélectionnez **Navigateur de stockage**.
1. Dans le navigateur de stockage, sélectionnez **Conteneurs Blob**.

    Actuellement, votre compte de stockage doit contenir uniquement le conteneur de **$logs** par défaut.

1. Dans la barre d'outils, sélectionnez **+ Conteneur** et créez un nouveau conteneur avec les paramètres suivants :
    - **Nom :** `documents`
    - **Niveau d'accès anonyme** : Privé (pas d’accès anonyme)\*

    > **Remarque** : \*Sauf si vous avez activé l’option permettant d’autoriser l’accès anonyme au conteneur lors de la création de votre compte de stockage, vous ne pourrez pas sélectionner d’autres paramètres !

1. Sélectionnez le conteneur de **documents** pour l’ouvrir, puis utilisez le bouton **Charger** la barre d’outils pour charger les fichiers .pdf que vous avez extraits de **documents.zip** précédemment à la racine du conteneur, comme illustré ici :

    ![Capture d'écran du navigateur de stockage Azure avec le conteneur de documents et son contenu.](./media/blob-containers.png)

## Créer et exécuter un indexeur

Maintenant que vous disposez des documents, vous pouvez créer un indexeur pour en extraire les informations.

1. Dans le Portail Azure, accédez à votre ressource Recherche Azure AI. Ensuite, dans sa page **Vue d’ensemble**, sélectionnez **Importer des données**.
1. Dans la page **Se connecter à vos données**, dans la liste **Source de données**, sélectionnez **Stockage Blob Azure**. Renseignez ensuite les détails du magasin de données avec les valeurs suivantes :
    - **Source de données** : Stockage Blob Azure
    - **Nom de la source de données** : `margies-documents`
    - **Données à extraire** : Contenu et métadonnées
    - **Mode d’analyse** : Par défaut
    - **Abonnement** : *votre abonnement Azure*
    - **Chaîne de connexion** : 
        - Sélectionnez **Choisir une connexion existante**
        - Sélectionner votre compte de stockage
        - Sélectionner le conteneur de **documents**
    - **Authentification d’identité managée** : Aucun
    - **Nom du conteneur** : documents
    - **Dossier d’objets blob** : *laissez ce champ vide*
    - **Description** : `Travel brochures`
1. Passez à l’étape suivante (**Ajouter des compétences cognitives**), qui comporte trois sections extensibles à terminer.
1. Dans la section **Attacher Azure AI Services** , sélectionnez **Gratuit (enrichissements limités**)\*.

    > **Remarque** : \*La ressource Azure AI Services gratuite pour Recherche IA Azure peut être utilisée pour indexer un maximum de 20 documents. Dans une solution réelle, vous devez créer une ressource Azure AI Services dans votre abonnement afin d'activer l'enrichissement par l'IA pour un plus grand nombre de documents.

1. Dans la section **Ajouter des enrichissements** :
    - Changez le **Nom du sous-réseau** en `margies-skillset`.
    - Sélectionnez l’option **Activer la reconnaissance optique de caractères et fusionner tout le texte dans le champ merged_content**.
    - Vérifiez que le **champ Données sources** est défini sur **merged_content**.
    - Laissez le **niveau de granularité de l’enrichissement** comme **Champ source**, qui est défini sur l’ensemble des contenus du document indexé. Notez cependant que vous pouvez modifier cette option pour extraire des informations à des niveaux plus précis, tels que des pages ou des phrases.
    - Sélectionnez les champs enrichis suivants :

        | Compétence cognitive | Paramètre | Nom du champ |
        | --------------- | ---------- | ---------- |
        | **Compétences cognitives en matière de texte** | |  |
        | Extraire les noms de personne | | people |
        | Extraire les noms d’emplacement | | locations |
        | Extraire les phrases clés | | keyphrases |
        | **Capacités cognitives liées à l'image** | |  |
        | Générer les balises des images | | imageTags |
        | Générer les légendes des images | | imageCaption |

        Vérifiez vos sélections (il peut être difficile de les modifier ultérieurement).

1. Sous **Enregistrer les enrichissements dans une base de connaissances**, sélectionnez :
    - Cochez uniquement les cases suivantes (une <font color="red">erreur</font> s’affiche, vous le résoudrez sous peu) :
        - **Projections de fichiers Azure** :
            - Projections d’image
        - **Projections de tables Azure** :
            - Documents
                - Phrases clés
        - **Projections de blobs Azure** :
            - Document
    - Sous **le compte de stockage chaîne de connexion** (sous les <font color="red">messages</font> d’erreur) :
        - Sélectionnez **Choisir une connexion existante**
        - Sélectionner votre compte de stockage
        - Sélectionnez le conteneur de **documents** (*cela n’est nécessaire que pour sélectionner le compte de stockage dans l’interface de navigation. Vous devez spécifier un autre nom de conteneur pour les ressources de connaissances extraites !*)
    - Modifiez le **nom du conteneur** en `knowledge-store`.
1. Passez à l’étape suivante (**Personnaliser l’index cible**), où vous spécifiez les champs de votre index. 
1. Remplacez le **nom de l’index** par `margies-index`.
1. Vérifiez que la **Clé** est définie sur **metadata_storage_path**, et laissez le **Nom du suggesteur** vide et le **Mode de recherche** sur **analyzingInfixMatching**.
1. Apportez les modifications suivantes aux champs d’index, en laissant tous les autres champs avec leurs paramètres par défaut (**IMPORTANT** : vous devrez peut-être faire défiler vers la droite pour afficher la table entière) :

    | Nom du champ | Récupérable | Filtrable | Triable | À choix multiples | Peut faire l’objet d’une recherche |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | people | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

    Vérifiez vos sélections, en veillant à ce que les options **Récupérables**, **Filtrables**, **Triables**, **À choix multiples** et avec une **Possibilité de recherche** soient sélectionnées pour chaque champ (il peut être difficile de les modifier ultérieurement).

1. Passez à l’étape suivante (**Créer un indexeur**), où vous allez créer et planifier l’indexeur.
1. Remplacez le **nom de l’indexeur** par `margies-indexer`.
1. Laissez la **Planification** définie sur **Une fois**.
1. Sélectionnez **Envoyer** pour créer la source de données, les compétences, l’index et l’indexeur. L’indexeur est exécuté automatiquement et exécute le pipeline d’indexation, qui :
    - Extrait les champs de métadonnées des documents et le contenu de la source de données
    - Exécute l’ensemble de compétences de compétences cognitives pour générer des champs enrichis supplémentaires
    - Mappe les champs extraits à l’index.
    - Enregistre les données extraites dans le référentiel de connaissances.
1. Dans le volet de navigation de gauche, sous la vue **gestion de la recherche**, la **page Indexeurs** doit afficher **margies-indexer** nouvellement créé. Attendez quelques minutes, puis cliquez sur **&orarr; Actualiser** jusqu’à ce que l’**État** indique la **réussite**.

## Rechercher l’index

Maintenant que vous avez un index, vous pouvez y effectuer des recherches.

1. En haut de la page **Vue d’ensemble** de votre ressource Recherche Azure AI, sélectionnez **Explorateur de recherche**.
1. Dans l’Explorateur de recherche, dans la zone **Chaîne de requête**, entrez `*` (un astérisque unique), puis sélectionnez **Rechercher**.

    Cette requête extrait tous les documents dans l’index au format JSON. Examinez les résultats et notez les champs de chaque document, qui contiennent du contenu, des métadonnées et des données enrichies extraites par les compétences cognitives que vous avez sélectionnées.

1. Dans le menu **Affichage** , sélectionnez **Affichage SJON** et vérifiez que la requête JSON pour la recherche est affichée, comme ici :

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Les résultats comprennent un champ **@odata.count** en haut de la page qui indique le nombre de documents renvoyés par la recherche.

1. Modifiez la requête JSON pour inclure le paramètre **select** comme indiqué ici :

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,locations"
    }
    ```

    Cette fois-ci, les résultats ne comprennent que le nom du fichier et les emplacements mentionnés dans le contenu du document. Le nom du fichier se trouve dans le champ **metadata_storage_name**, qui a été extrait du document source. Le champ **emplacements** a été généré par une compétence IA.

1. Essayez la chaîne de requête suivante :

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    Cette recherche recherche des documents qui mentionnent « New York » dans l’un des champs pouvant faire l’objet d’une recherche et retourne le nom de fichier et les expressions clés du document.

1. Essayons une de requête supplémentaire :

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "metadata_storage_name,keyphrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    Cette requête renvoie le nom de fichier et les phrases clés de tous les documents mentionnant « New York » dont la taille est inférieure à 380 000 octets.

## Créer une application cliente de recherche

Maintenant que vous avez un index utile, vous pouvez l’utiliser à partir d’une application cliente. Vous pouvez le faire en consommant l’interface REST, en envoyant des demandes et en recevant des réponses au format JSON via HTTP ; ou vous pouvez utiliser le kit de développement logiciel (SDK) pour votre langage de programmation préféré. Dans cet exercice, nous allons utiliser le kit de développement logiciel (SDK).

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

### Obtenir le point de terminaison et les clés de votre ressource de recherche

1. Dans le portail Azure, fermez la page de l'explorateur de recherche et revenez à la page **Aperçu** de votre ressource Azure AI Search.

    Notez la valeur d’**URL**, qui doit être similaire à **https://*your_resource_name*.search.windows.net**. Il s’agit du point de terminaison de votre ressource de recherche.

1. Dans le volet de navigation à gauche, développez **Paramètres** et consultez la page **Clés**.

    Notez qu'il existe deux clés **admin** et une seule clé **requête**. Une clé d’*administration* est utilisée pour créer et gérer des ressources de recherche. Une clé de *requête* est utilisée par les applications clientes qui doivent uniquement effectuer des requêtes de recherche.

    *Vous aurez besoin du point de terminaison et de la clé de **requête** pour votre application cliente.*

### Se préparer à l’utilisation du kit de développement logiciel (SDK) de Recherche Azure AI

1. Utilisez le bouton **[\>_]** situé à droite de la barre de recherche en haut du portail Azure pour créer un nouveau Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** sans stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure. Vous pouvez redimensionner ou agrandir ce volet pour faciliter le travail. Au départ, vous devrez consulter à la fois le Cloud Shell et le portail Azure (afin de trouver et de copier le point de terminaison et la clé dont vous aurez besoin).

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Conseil** : lorsque vous saisissez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :

    ```
   cd mslearn-ai-info/Labfiles/knowledge/python
   ls -a -l
    ```

1. Installez le SDK Azure AI Search et les packages d'identité Azure en exécutant les commandes suivantes :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-search-documents==11.5.1
    ```

1. Exécutez la commande suivante pour modifier le fichier de configuration de votre application :

    ```
   code .env
    ```

    Le fichier de configuration est ouvert dans un éditeur de code.

1. Modifiez le fichier de configuration pour remplacer les valeurs par défaut suivantes :

    - **your_search_endpoint** (*Remplacez par le point de terminaison de votre ressource Azure AI Search*)
    - **your_query_key** *(remplacez par la clé de requête de votre ressource Recherche Azure AI*)
    - **your_index_name** (*remplacez par le nom de votre index, qui devrait être `margies-index`*)

1. Lorsque vous avez mis à jour les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer le fichier, puis utilisez la commande **Ctrl+Q** pour la fermer.

    > **Conseil** : Maintenant que vous avez copié le point de terminaison et la clé à partir du portail Azure, vous pouvez agrandir le volet Cloud Shell afin de faciliter votre travail.

1. Exécutez la commande suivante pour ouvrir le fichier de code de votre application :

    ```
   code search-app.py
    ```

    Le fichier de code est ouvert dans un éditeur de code.

1. Examinez le code et notez qu'il effectue les actions suivantes :

    - Récupère les paramètres de configuration de votre ressource Azure AI Search et de votre index à partir du fichier de configuration que vous avez modifié.
    - Crée un **SearchClient** avec le point de terminaison, la clé et le nom d'index pour se connecter à votre service de recherche.
    - Demande à l'utilisateur de saisir une requête de recherche (jusqu'à ce qu'il tape « quit »)
    - Recherche dans l'index à l'aide de la requête, renvoyant les champs suivants (classés par metadata_storage_name) :
        - metadata_storage_name
        - locations
        - people
        - keyphrases
    - Analyse les résultats de recherche renvoyés afin d'afficher les champs renvoyés pour chaque document dans le jeu de résultats.

1. Fermez le volet de l'éditeur de code (*CTRL+Q*), gardez le volet de la console de ligne de commande Cloud Shell ouvert
1. Entrez la commande suivante pour exécuter l'application :

    ```
   python search-app.py
    ```

1. Lorsque vous y êtes invité, entrez une requête telle que `London` et affichez les résultats.
1. Essayez une autre requête, par exemple `flights`.
1. Lorsque vous avez terminé de tester l'application, appuyez sur `quit` pour la fermer.
1. Fermez le Cloud Shell et revenez au portail Azure.

## Afficher la base de connaissances

Une fois que vous avez exécuté un indexeur qui utilise un ensemble de compétences pour créer une base de connaissances, les données enrichies extraites par le processus d’indexation sont conservées dans les projections de la base de connaissances.

### Afficher les projections d’objets

Les projections d’*objets* définies dans l’ensemble de compétences de Margie’s Travel se composent d’un fichier JSON pour chaque document indexé. Ces fichiers sont stockés dans un conteneur d’objets blob dans le compte de stockage Azure spécifié dans la définition de l’ensemble de compétences.

1. Dans le Portail Azure, affichez le compte de stockage Azure que vous avez créé précédemment.
1. Sélectionnez l’onglet **Explorateur de stockage** (dans le volet de gauche) pour voir le compte de stockage dans l’interface de l’Explorateur de stockage du portail Azure.
1. Développez **Conteneurs d’objets blob** pour voir les conteneurs dans le compte de stockage. En plus du conteneur de **documents** où sont stockées les données sources, il devrait y avoir deux nouveaux conteneurs : **knowledge-store** et **margies-skillset-image-projection**. Ceux-ci ont été créés par le processus d’indexation.
1. Sélectionnez le conteneur **knowledge-store**. Il doit contenir un dossier pour chaque document indexé.
1. Ouvrez l’un des dossiers, puis sélectionnez le fichier **objectprojection.json** qu’il contient et utilisez le bouton **Télécharger** dans la barre d’outils pour le télécharger et l’ouvrir. Chaque fichier JSON contient une représentation d'un document indexé, y compris les données enrichies extraites par le skillset, comme illustré ici (formatées pour faciliter la lecture).

    ```json
    {
        "metadata_storage_content_type": "application/pdf",
        "metadata_storage_size": 388622,
        "<more_metadata_fields>": "...",
        "key_phrases":[
            "Margie’s Travel",
            "Margie's Travel",
            "best travel experts",
            "world-leading travel agency",
            "international reach"
            ],
        "locations":[
            "Dubai",
            "Las Vegas",
            "London",
            "New York",
            "San Francisco"
            ],
        "image_tags":[
            "outdoor",
            "tree",
            "plant",
            "palm"
            ],
        "more fields": "..."
    }
    ```

La possibilité de créer des projections d’*objets* comme celle-ci vous permet de générer des objets de données enrichis qui peuvent être incorporés dans une solution d’analyse des données d’entreprise.

### Afficher des projections de fichiers

Les projections de *fichiers* définies dans l’ensemble de compétences créent des fichiers JPEG pour chaque image extraite des documents pendant le processus d’indexation.

1. Dans l’interface *Navigateur de stockage* du Portail Azure, sélectionnez le conteneur d’objets blob **margies-skillset-image-projection**. Ce conteneur contient un dossier pour chaque document qui contenait des images.
2. Ouvrez l’un des dossiers et affichez son contenu : chaque dossier contient au moins un fichier \*.jpg.
3. Ouvrez l'un des fichiers image, téléchargez-le et affichez-le pour voir l'image.

La capacité à générer des projections de *fichiers* comme celle-ci permet d’envisager l’indexation comme un moyen efficace d’extraire des images incorporées à partir d’un grand volume de documents.

### Afficher des projections de tables

Les projections de *tables* définies dans l’ensemble de compétences forment un schéma relationnel de données enrichies.

1. Dans l’interface *Navigateur de stockage* du Portail Azure, développez **Tables**.
2. Sélectionnez la table **margiesSkillsetDocument** pour afficher les données. Ce tableau contient une ligne pour chaque document qui a été indexé :
3. Affichez la table **margiesSkillsetKeyPhrases** , qui contient une ligne pour chaque expression clé extraite des documents.

La possibilité de créer des projections de *tables* vous permet de développer des solutions analytiques et de reporting qui interrogent un schéma relationnel. Les colonnes clés générées automatiquement peuvent être utilisées pour joindre les tables dans les requêtes, par exemple pour renvoyer toutes les expressions clés extraites d'un document spécifique.

## Nettoyage

Maintenant que vous avez terminé l’exercice, supprimez toutes les ressources dont vous n’avez plus besoin. Supprimez les ressources Azure :

1. Dans le **portail Azure**, sélectionnez Groupes de ressources.
1. Sélectionnez le groupe de ressources dont vous n’avez pas besoin, puis **Supprimer le groupe de ressources**.

## Plus d’informations

Pour en savoir plus sur la plateforme Recherche Azure AI, consultez la [documentation de Recherche Azure AI](https://docs.microsoft.com/azure/search/search-what-is-azure-search).

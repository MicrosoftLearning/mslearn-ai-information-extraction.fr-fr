---
lab:
  title: "Analyser des formulaires avec des modèles personnalisés d’Azure\_AI\_Document\_Intelligence"
  description: Créez un modèle Document Intelligence personnalisé pour extraire des données spécifiques à partir de documents.
---

# Analyser des formulaires avec des modèles personnalisés d’Azure AI Document Intelligence

Supposons qu’une entreprise demande actuellement à ses employés d’acheter manuellement des feuilles de commande et de saisir les données dans une base de données. Ils souhaitent utiliser des services IA pour améliorer le processus d’entrée de données. Vous décidez de créer un modèle Machine Learning qui lit le formulaire et produit des données structurées qui peuvent être utilisées pour mettre à jour automatiquement une base de données.

**Azure AI Intelligence documentaire** est un service Azure AI qui permet aux utilisateurs de générer un logiciel de traitement de données automatisé. Ce logiciel peut extraire du texte, des paires clé/valeur et des tables à partir de documents de formulaire à l’aide de la reconnaissance optique de caractères (OCR). Azure AI Intelligence documentaire propose des modèles prédéfinis pour reconnaître les factures, les reçus et les carte d’entreprise. Le service offre également la possibilité d’entraîner des modèles personnalisés. Dans cet exercice, nous allons nous concentrer sur la création de modèles personnalisés.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications similaires à l'aide de plusieurs SDK spécifiques à un langage, notamment :

- [Bibliothèque cliente Azure AI Document Intelligence pour Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Bibliothèque cliente Azure AI Document Intelligence pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Bibliothèque cliente Azure AI Document Intelligence pour JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Cet exercice prend environ **30** minutes.

## Créer une ressource Azure AI Intelligence documentaire

Pour utiliser le service Azure AI Intelligence documentaire, vous avez besoin d’une ressource Azure AI Intelligence documentaire ou Azure AI Services dans votre abonnement Azure. Vous utilisez le Portail Azure pour créer une ressource.

1. Dans un onglet de navigateur, ouvrez le Portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Sur la page d’accueil du Portail Azure, accédez à la zone de recherche supérieure, tapez **Intelligence documentaire**, puis appuyez sur **Entrée**.
1. Sur la page **Intelligence documentaire**, sélectionnez **Créer**.
1. Dans la page **Créer Document Intelligence** , créez une ressource avec les paramètres suivants :
    - **Abonnement**: Votre abonnement Azure.
    - **Groupe de ressources** : créez ou sélectionnez un groupe de ressources.
    - **Région** : Toute région disponible
    - **Nom** : Un nom valide pour votre ressource Document Intelligence
    - **Niveau tarifaire** : Free F0 (*si vous ne disposez pas d'un niveau Free, sélectionnez* Standard S0).
1. Le déploiement terminé, sélectionnez **Accéder à la ressource** pour afficher la page **Vue d’ensemble** de la ressource.

## Préparer le développement d’une application dans Cloud Shell

Vous développerez votre application de traduction de texte à l'aide de Cloud Shell. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

1. Dans le portail Azure, cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Ajustez la taille du volet Cloud Shell afin de pouvoir voir à la fois la console de ligne de commande et le portail Azure. Vous devrez utiliser la barre de séparation pour basculer entre les deux volets.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## Collecter des documents pour l’entraînement

Vous allez utiliser des échantillons de formulaires comme celui-ci pour effectuer l’apprentissage d’un modèle de test : 

![Image d’une facture utilisée dans ce projet.](./media/Form_1.jpg)

1. Dans la ligne de commande, exécutez pour `ls ./sample-forms`répertorier le contenu dans le **dossier sample-forms**. Notez que les fichiers se terminent par **.json** et **.jpg** dans le dossier.

    Vous utiliserez les fichiers **.jpg** pour entraîner votre modèle.  

    Les fichiers **.json** ont été générés pour vous et contiennent des informations d’étiquette. Les fichiers seront chargés dans votre conteneur de stockage d’objets blob en même temps que les formulaires.

1. Dans le **portail Azure**, accédez à la page **Présentation** de votre ressource si vous n'y êtes pas déjà. Dans la section *Essentials*, notez le **groupe de ressources**, **l’ID d’abonnement** et **l’emplacement**. Vous aurez besoin de ces valeurs dans les étapes suivantes.
1. Exécutez la commande `code setup.sh`pour ouvrir**setup.sh** dans un éditeur de code. Vous allez utiliser ce script pour exécuter les commandes d’interface de ligne de commande (CLI) Azure requises pour créer les autres ressources Azure dont vous avez besoin.

1. Dans le script **setup.cmd**, passez en revue les commandes. Le programme va :
    - Créer un compte de stockage dans votre groupe de ressources Azure
    - Télécharger des fichiers de votre dossier *sampleforms* local vers un conteneur appelé *sampleforms* dans le compte de stockage
    - Imprimer un URI de signature d’accès partagé

1. Modifiez les déclarations de variable **subscription_id**, **resource_group** et **location** avec les valeurs appropriées pour le nom de l’abonnement, du groupe de ressources et de l’emplacement où vous avez déployé la ressource Intelligence documentaire.

    > **Important** : Pour votre chaîne d’**emplacement**, veillez à utiliser la version de code de votre emplacement. Par exemple, si votre emplacement est « East US », la chaîne dans votre script doit être `eastus`. Vous pouvez voir que cette version est le bouton **Affichage JSON** situé à droite de l’onglet **Essentiels** de votre groupe de ressources dans Portail Azure.

    Si la variable **expiry_date** est passée, mettez-la à jour à une date ultérieure. Cette variable est utilisée lors de la génération de l’URI SAP (Shared Access Signature). Dans la pratique, vous devez définir une date d’expiration appropriée pour votre SAP. Vous en découvrirez davantage sur les SAP [ici](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).  

1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.

1. Entrez les commandes suivantes pour rendre le script exécutable et le lancer :

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. Une fois le script terminé, passez en revue la sortie affichée.

1. Dans le Portail Azure, actualisez votre groupe de ressources et vérifiez qu’il contient le compte Stockage Azure qui vient d’être créé. Ouvrez le compte de stockage et, dans le volet de gauche, sélectionnez **Navigateur de stockage **. Ensuite, dans le navigateur de stockage, développez **Conteneurs Blob** et sélectionnez le conteneur **sampleforms** pour vérifier que les fichiers ont bien été téléchargés depuis votre dossier local **custom-doc-intelligence/sample-forms**.

## Effectuer l’apprentissage du modèle à l’aide de Studio Intelligence documentaire

Vous allez maintenant effectuer l’apprentissage du modèle à l’aide des fichiers chargés sur le compte de stockage.

1. Ouvrez un nouvel onglet dans votre navigateur et accédez à Document Intelligence Studio sur `https://documentintelligence.ai.azure.com/studio`. 
1. Faites défiler vers le bas jusqu’à la section **Modèles personnalisés** et sélectionnez la vignette **Modèle d’extraction personnalisé**.
1. Si vous y êtes invité, connectez-vous à l'aide de vos informations d'identification Azure.
1. Si vous êtes invité à choisir la ressource Azure AI Intelligence documentaire à utiliser, sélectionnez l’abonnement et le nom de la ressource que vous avez utilisés lors de la création de la ressource Azure AI Intelligence documentaire.
1. Sous **Mes projets**, créez un projet avec la configuration suivante :

    - **Saisir les détails du projet** :
        - **Nom du projet** : Un nom valide pour votre projet
    - **Configurer la ressource de service** :
        - **Abonnement** : votre abonnement Azure.
        - **Groupe de ressources** : Le groupe de ressources dans lequel vous avez déployé votre ressource Document Intelligence
        - **Ressource Document Intelligence** Votre ressource Document Intelligence (sélectionnez l’option *Définir comme option par défaut* et utilisez la version de l’API par défaut)
    - **Connecter la source de données d’entraînement** :
        - **Abonnement** : votre abonnement Azure.
        - **Groupe de ressources** : Le groupe de ressources dans lequel vous avez déployé votre ressource Document Intelligence
        - **Compte de stockage** : Le compte de stockage créé par le script d’installation (sélectionnez l’option *Définir comme option par défaut* , sélectionnez le conteneur d’objets blob et laissez le `sampleforms` chemin du dossier vide)

1. Une fois votre projet créé, en haut à droite de l’écran, sélectionnez **Effectuer l’apprentissage** pour entraîner votre modèle. Utilisez les configurations suivantes :
    - **ID de modèle** : Nom valide pour votre modèle (*vous aurez besoin du nom d’ID de modèle à l’étape suivante*)
    - **Mode de génération** : modèle.
1. Cliquez sur **Accéder aux modèles**.
1. L’apprentissage peut prendre un certain temps. Attendez que le statut soit **réussi**.

## Tester votre modèle Intelligence documentaire personnalisé

1. Revenez à l'onglet du navigateur contenant le portail Azure et le Cloud Shell. Dans la ligne de commande, exécutez la commande suivante pour passer au dossier contenant les fichiers de code d’application :

    ```
    cd Python
    ```

1. Installez le package Document Intelligence en exécutant la commande suivante :

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

1. Dans le volet contenant le Portail Azure, dans la page **Vue d’ensemble** de votre ressource Document Intelligence, sélectionnez **Cliquez ici pour gérer les clés** pour afficher le point de terminaison et les clés de votre ressource. Modifiez ensuite le fichier de configuration avec les valeurs suivantes :
    - Votre point de terminaison Document Intelligence
    - Votre clé Document Intelligence
    - L'ID du modèle que vous avez spécifié lors de l'entraînement de votre modèle

1. Une fois les espaces réservés remplacés, utilisez la commande **CTRL+S** dans l’éditeur de code pour enregistrer vos modifications, puis la commande **CTRL+Q** pour fermer l’éditeur tout en gardant la ligne de commande Cloud Shell ouverte.

1. Ouvrez le fichier de code de votre application cliente ( `code Program.cs`pour C#, `code test-model.py` pour Python) et vérifiez le code qu'il contient, en particulier que l'image dans l'URL renvoie au fichier dans ce référentiel GitHub sur le Web. Fermez le fichier sans apporter de modifications.

1. Dans la ligne de commande, entrez la commande suivante pour exécuter le programme :

    ```
   python test-model.py
    ```

1. Affichez la sortie et observez comment la sortie du modèle fournit des noms de champs tels que `Merchant` et `CompanyPhoneNumber`.

## Nettoyage

Si vous en avez terminé avec votre ressource Azure, n’oubliez pas de la supprimer dans le [Portail Azure](https://portal.azure.com/?azure-portal=true) pour éviter les frais supplémentaires.

## Plus d’informations

Pour plus d’informations sur le service Intelligence documentaire, consultez la documentation [Intelligence documentaire](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true).

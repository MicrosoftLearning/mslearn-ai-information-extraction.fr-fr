---
lab:
  title: Développer une application cliente Compréhension de contenu
  description: Utilisez l'API REST Azure AI Content Understanding pour développer une application cliente pour un analyseur.
---

# Développer une application cliente Compréhension de contenu

Dans cet exercice, vous utilisez Azure AI Content Understanding pour créer un analyseur qui extrait des informations à partir de cartes de visite. Vous développerez ensuite une application client qui utilise l'analyseur pour extraire les coordonnées des cartes de visite numérisées.

Cet exercice prend environ **30** minutes.

## Créer un hub et un projet Azure AI Foundry

Les fonctionnalités d’Azure AI Foundry que nous allons utiliser dans cet exercice nécessitent un projet basé sur une ressource de *hub* Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./media/ai-foundry-home.png)

1. Dans le navigateur, accédez à `https://ai.azure.com/managementCenter/allResources` et sélectionnez **Créer un nouveau**. Choisissez ensuite l’option permettant de créer une **ressource de hub AI**.
1. Dans l’assistant **Créer un projet**, saisissez un nom valide pour votre projet et sélectionnez l’option permettant de créer un nouveau hub. Utilisez ensuite le lien **Renommer le hub** pour spécifier un nom valide pour votre nouveau hub, développez les **Options avancées** et définissez les paramètres suivants pour votre projet :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Nom du hub** : un nom valide pour votre hub
    - **Emplacement** : Choisissez l'un des emplacements suivants :\*
        - Australie Est
        - Suède Centre
        - USA Ouest

    > \*Au moment de l’écriture, la compréhension du contenu d’IA Azure n’est disponible que dans ces régions.

    > **Conseil** : si le bouton **Créer** est toujours désactivé, veillez à renommer votre hub en une valeur alphanumérique unique.

1. Attendez que votre projet soit créé, puis accédez à la page d'aperçu de votre projet.

## Utilisez l'API REST pour créer un analyseur de compréhension du contenu

Vous allez utiliser l'API REST pour créer un analyseur capable d'extraire des informations à partir d'images de cartes de visite.

1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

    Fermez les notifications de bienvenue pour afficher la page d’accueil du portail Azure.

1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure. Vous pouvez redimensionner ou agrandir ce volet pour faciliter le travail.

    > **Conseil** : Redimensionnez le volet afin de pouvoir travailler principalement dans le shell cloud tout en conservant les clés et le point de terminaison visibles dans la page du portail Azure. Vous devrez les copier dans votre code.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Conseil** : lorsque vous saisissez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le dépôt cloné, accédez au dossier contenant les fichiers de code de votre application :

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    Le dossier contient deux images de cartes de visite numérisées ainsi que les fichiers de code Python dont vous avez besoin pour créer votre application.

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous allez utiliser :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez les espaces réservés **YOUR_ENDPOINT** et **YOUR_KEY** par le point de terminaison de vos services Azure AI et l’une de ses clés (copiées à partir du Portail Azure) et vérifiez que **ANALYZER_NAME** est défini sur `business-card-analyzer`.
1. Une fois les espaces réservés remplacés, utilisez la commande **CTRL+S** dans l’éditeur de code pour enregistrer vos modifications, puis la commande **CTRL+Q** pour fermer l’éditeur tout en gardant la ligne de commande Cloud Shell ouverte.

    > **Conseil** : Vous pouvez désormais agrandir le volet Cloud Shell.

1. Dans la ligne de commande cloud shell, entrez la commande suivante pour afficher le fichier JSON **biz-card.json** fourni :

    ```
   cat biz-card.json
    ```

    Faites défiler le volet Cloud Shell pour afficher le JSON dans le fichier, qui définit un schéma d'analyse pour une carte de visite.

1. Lorsque vous avez consulté le fichier JSON de l’analyseur, entrez la commande suivante pour modifier le fichier de code Python **create-analyzer.py** fourni :

    ```
   code create-analyzer.py
    ```

    Le fichier de code Python est ouvert dans un éditeur de code.

1. Passez en revue le code, qui :
    - Charge le schéma de l'analyseur à partir du fichier **biz-card.json**.
    - Récupère le nom du point de terminaison, de la clé et de l'analyseur à partir du fichier de configuration de l'environnement.
    - Appelle une fonction nommée **create_analyzer**, qui n’est actuellement pas implémentée

1. Dans la fonction **create_analyzer** , recherchez le commentaire **Créer un analyseur Content Understanding** et ajoutez le code suivant (veillez à conserver la mise en retrait correcte) :

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. Vérifiez le code que vous avez ajouté, qui :
    - Crée des en-têtes appropriés pour les requêtes REST
    - Envoie une requête HTTP *DELETE* pour supprimer l’analyseur s’il existe déjà.
    - Envoie une requête HTTP *PUT* pour lancer la création de l’analyseur.
    - Vérifie la réponse pour récupérer l’URL de *rappel Operation-Location* .
    - Envoie à plusieurs reprises une requête HTTP *GET* à l’URL de rappel pour vérifier l’état de l’opération jusqu’à ce qu’elle ne soit plus en cours d’exécution.
    - Confirme la réussite (ou l’échec) de l’opération à l’utilisateur.

    > **Remarque** : Le code inclut des retards de temps délibérés pour éviter de dépasser la limite de débit de requête du service.

1. Utilisez la commande **CTRL+S** pour enregistrer les modifications de code, mais laissez le volet de l’éditeur de code ouvert si vous devez corriger les erreurs dans le code. Redimensionnez les volets afin de voir clairement le volet de ligne de commande.
1. Dans le volet de ligne de commande Cloud Shell, entrez la commande suivante pour exécuter le code Python :

    ```
   python create-analyzer.py
    ```

1. Vérifiez le résultat du programme, qui devrait indiquer que l'analyseur a bien été créé.

## Utilisez l'API REST pour analyser le contenu

Maintenant que vous avez créé un analyseur, vous pouvez l’utiliser à partir d’une application cliente via l’API REST Compréhension de contenu.

1. Dans la ligne de commande du cloud shell, entrez la commande suivante pour modifier le fichier de code Python **read-card.py** qui vous a été fourni :

    ```
   code read-card.py
    ```

    Le fichier de code Python est ouvert dans un éditeur de code :

1. Passez en revue le code, qui :
    - Identifie le fichier image à analyser, avec une valeur par défaut de **biz-card-1.png**.
    - Récupère le point de terminaison et la clé de votre ressource Azure AI Services à partir du projet (à l’aide des informations d’identification Azure de la session Cloud Shell actuelle pour s’authentifier).
    - Appelle une fonction nommée **analyze_card**, qui n’est actuellement pas implémentée

1. Dans la fonction **analyze_card** , recherchez le commentaire **Utiliser la compréhension du contenu** et ajoutez le code suivant (veillez à conserver la mise en retrait correcte) :

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. Vérifiez le code que vous avez ajouté, qui :
    - Lit le contenu du fichier image
    - Définit la version de l’API REST Content Understanding à utiliser
    - Envoie une requête HTTP *POST* à votre point de terminaison Content Understanding, lui demandant d'analyser l'image.
    - Vérifie la réponse de l'opération afin de récupérer un identifiant pour l'opération d'analyse.
    - Envoie de manière répétée une requête HTTP *GET* à votre point de terminaison Content Understanding afin de vérifier l'état de fonctionnement jusqu'à ce qu'il ne soit plus en cours d'exécution.
    - Si l'opération a réussi, enregistre la réponse JSON, puis analyse le JSON et affiche les valeurs récupérées pour chaque champ spécifique au type.

    > **Remarque** : Dans notre schéma de carte de visite simple, tous les champs sont des chaînes. Le code ci-dessous illustre la nécessité de vérifier le type de chaque champ afin de pouvoir extraire des valeurs de différents types à partir d'un schéma plus complexe.

1. Utilisez la commande **CTRL+S** pour enregistrer les modifications de code, mais laissez le volet de l’éditeur de code ouvert si vous devez corriger les erreurs dans le code. Redimensionnez les volets afin de voir clairement le volet de ligne de commande.
1. Dans le volet de ligne de commande Cloud Shell, entrez la commande suivante pour exécuter le code Python :

    ```
   python read-card.py biz-card-1.png
    ```

1. Vérifiez le résultat du programme, qui devrait afficher les valeurs des champs de la carte de visite suivante :

    ![Carte de visite de Roberto Tamburello, employé chez Adventure Works Cycles.](./media/biz-card-1.png)

1. Utilisez la commande suivante pour exécuter le programme avec une autre carte de visite :

    ```
   python read-card.py biz-card-2.png
    ```

1. Vérifiez les résultats, qui devraient refléter les valeurs indiquées sur cette carte de visite :

    ![Carte de visite de Mary Duartes, employée chez Contoso.](./media/biz-card-2.png)

1. Dans le volet de ligne de commande du shell cloud, utilisez la commande suivante pour afficher la réponse JSON complète qui a été renvoyée :

    ```
   cat results.json
    ```

    Faites défiler pour afficher le JSON.

## Nettoyage

Si vous avez terminé d’utiliser le service Compréhension de contenu, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Dans le portail Azure, supprimez les ressources que vous avez créées dans cet exercice.

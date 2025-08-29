---
lab:
  title: "Analyser des formulaires à l’aide des modèles prédéfinis d’Azure\_AI\_Document\_Intelligence"
  description: Utilisez des modèles Azure AI Document Intelligence prédéfinis pour traiter les champs de texte à partir de documents.
---

# Analyser des formulaires à l’aide des modèles prédéfinis d’Azure AI Document Intelligence

Dans cet exercice, vous allez configurer un projet Azure AI Foundry avec toutes les ressources nécessaires pour l’analyse des documents. Vous allez utiliser le portail Azure AI Foundry et le Kit de développement logiciel (SDK) Python pour envoyer des formulaires à cette ressource à des fins d’analyse.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications similaires à l'aide de plusieurs SDK spécifiques à un langage, notamment :

- [Bibliothèque cliente Azure AI Document Intelligence pour Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Bibliothèque cliente Azure AI Document Intelligence pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Bibliothèque cliente Azure AI Document Intelligence pour JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Cet exercice prend environ **30** minutes.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./media/ai-foundry-home.png)

1. Dans le navigateur, accédez à `https://ai.azure.com/managementCenter/allResources` et sélectionnez **Créer un nouveau**. Choisissez ensuite l’option permettant de créer une **ressource de hub AI**.
1. Dans l’assistant **Créer un projet**, saisissez un nom valide pour votre projet et sélectionnez l’option permettant de créer un nouveau hub. Utilisez ensuite le lien **Renommer le hub** pour spécifier un nom valide pour votre nouveau hub, développez les **Options avancées** et définissez les paramètres suivants pour votre projet :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** :  *Toute région disponible*

    > **Note** : si vous travaillez dans un abonnement Azure dans lequel les stratégies sont utilisées pour restreindre les noms de ressources autorisés, vous devrez peut-être utiliser le lien en bas de la boîte de dialogue **Créer un projet** pour créer le hub à l’aide du portail Azure.

    > **Conseil** : si le bouton **Créer** est toujours désactivé, veillez à renommer votre hub en une valeur alphanumérique unique.

1. Attendez que votre projet soit créé.
1. Une fois votre projet créé, fermez les conseils affichés et passez en revue la page du projet dans le portail Azure AI Foundry, qui doit ressembler à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](./media/ai-foundry-project.png)

## Utiliser le modèle Lecture

Commençons par utiliser le portail **Azure AI Foundry** et le modèle Read pour analyser un document contenant plusieurs langues :

1. Dans le panneau de navigation à gauche, sélectionnez **Services IA**.
1. Dans la page **Azure AI Services** , sélectionnez la mosaïque **Vision + Document** .
1. Dans la page **Vision + Document** , vérifiez que l’onglet **Document** est sélectionné, puis sélectionnez la mosaïque **OCR/Lecture** .

    Dans la page **Lecture** , la ressource Azure AI Services créée avec votre projet doit déjà être connectée.

1. Dans la liste des documents de gauche, sélectionnez **read-german.pdf**.

    ![Capture d’écran montrant la page Lecture dans Azure AI Document Intelligence Studio.](./media/read-german-sample.png#lightbox)

1. En haut à gauche, sélectionnez **Analyser les options**, puis activez la case à cocher **Langue** (sous **Détection facultative**) dans le volet **Analyser les options**, puis cliquez sur **Enregistrer**. 
1. En haut à gauche, sélectionnez **Exécuter l’analyse**.
1. Une fois l’analyse terminée, le texte extrait de l’image s’affiche à droite sous l’onglet **Contenu**. Passez en revue ce texte et comparez sa justesse par rapport au texte de l’image d’origine.
1. Sélectionnez l’onglet **Résultat**. Cet onglet affiche le code JSON extrait. 

## Préparer le développement d’une application dans Cloud Shell

Examinons maintenant l’application qui utilise le Kit de développement logiciel (SDK) du service Azure Intelligence documentaire. Vous allez développer votre application à l’aide de Cloud Shell. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

Il s’agit de la facture que votre code va analyser.

![Capture d’écran montrant un échantillon de document de facture.](./media/sample-invoice.png)

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Points de terminaison et clés**, sélectionnez l’onglet **Azure AI Services**, puis notez la **clé API** et le **point de terminaison Azure AI Services**. Vous utiliserez ces informations d’identification pour vous connecter à vos services Azure AI dans une application cliente.
1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

    ***Suivez maintenant les étapes de votre langage de programmation choisi.***

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code :

    ```
   cd mslearn-ai-info/Labfiles/prebuilt-doc-intelligence/Python
    ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous utiliserez, c’est-à-dire :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez les espaces réservés **YOUR_ENDPOINT** et **YOUR_KEY** par votre point de terminaison Azure AI Services et sa clé API (copiée depuis le portail Azure AI Foundry).
1. Une fois les espaces réservés remplacés, utilisez la commande **CTRL+S** dans l’éditeur de code pour enregistrer vos modifications, puis la commande **CTRL+Q** pour fermer l’éditeur tout en gardant la ligne de commande Cloud Shell ouverte.

## Ajouter du code pour utiliser le service Azure Intelligence documentaire

Vous êtes maintenant prêt à utiliser le Kit de développement logiciel (SDK) pour évaluer le fichier pdf.

1. Entrez la commande suivante pour modifier le fichier de l'application qui vous a été fourni :

    ```
   code document-analysis.py
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, recherchez le commentaire **Importer les bibliothèques requises** et ajoutez le code suivant :

    ```python
   # Add references
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.formrecognizer import DocumentAnalysisClient
    ```

1. Recherchez le commentaire **Analyser la facture** et ajoutez le code suivant (veillez à maintenir le niveau de retrait correct) :

    ```python
   # Create the client
   document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
   )
    ```

1. Recherchez le commentaire **Analyser la facture** et ajoutez le code suivant :

    ```python
   # Analyse the invoice
   poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
   )
    ```

1. Recherchez les **informations de facture du commentaire à l’utilisateur** et ajoutez le code suivant :

    ```python
   # Display invoice information to the user
   receipts = poller.result()
    
   for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")

        customer_name = receipt.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: '{customer_name.value}, with confidence {customer_name.confidence}.")


        invoice_total = receipt.fields.get("InvoiceTotal")
        if invoice_total:
            print(f"Invoice Total: '{invoice_total.value.symbol}{invoice_total.value.amount}, with confidence {invoice_total.confidence}.")
    ```

1. Dans l'éditeur de code, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** pour enregistrer vos modifications. Gardez l'éditeur de code ouvert au cas où vous auriez besoin de corriger des erreurs dans le code, mais redimensionnez les volets afin de pouvoir voir clairement le volet de la ligne de commande.

1. Dans le volet de ligne de commande, entrez la commande suivante pour exécuter l'application.

    ```
    python document-analysis.py
    ```

Le programme affiche le nom du fournisseur, le nom du client et le total de la facture avec des niveaux de confiance. Comparez les valeurs qu’il indique avec l’exemple de facture que vous avez ouvert au début de cette section.

## Nettoyage

Si vous n'utilisez plus votre ressource Azure, n'oubliez pas de la supprimer dans le [portail Azure](https://portal.azure.com) (`https://portal.azure.com`) afin d'éviter des frais supplémentaires.

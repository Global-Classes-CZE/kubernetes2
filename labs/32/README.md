# Lab 32

Cílem tohoto labu je v Azure vytvořit service principal, který bude mít práva na pushování kontejnerů do Azure Container Registry.

> Tento lab může udělat jen ten, kdo má v Azure Active Directory práva na vytváření účtů (např. role Application Developer).

## Krok 1 - Propojení Azure DevOps s Azure Container Registry

1. V nastavení projektu v Azure DevOps a najděte sekci __Pipelines / Service Connections__.

2. Přidejte connection __Docker Registry__, vyberte subscription a příslušné registry. 

> Tím dojde k automatickému vytvoření Service principala, který bude mít práva na push kontejnerů do vybraného registry.

## Krok 2 - Push kontejneru do registry

1. Nahoře v pipeline si vytvořte proměnnou a umístěte do ní URL adresu Azure Container Registry:

```
variables:
  registryName: '<REGISTRY_NAME>.azurecr.io'
```

2. Před pushem musíme musíme image přejmenovat na cílový název. Přidejte krok __Docker__:

    * Jako command zadejte `tag`

    * Jako argumenty dejme `northwindstoreapp:latest $(registryName)/northwindstore/app:latest`

    * Odškrtněte checkboxy týkající se metadat - nepotřebujeme je.

3. Zkopírujte tento krok a přidejte ještě druhý tag s číslem buildu:

    * Argumenty budou `northwindstoreapp:latest $(registryName)/northwindstore/app:$(Build.BuildId)`

4. Přidejte druhý krok __Docker__.

    * Jako container registry vyberte nově vytvořené __Service connection__

    * Container repository nastavte na `northwindstore/app`

    * Do tagů nastavte následující hodnoty:

```
$(Build.BuildId)
latest
```

5. Uložte a vyzkoušejte, že se konfigurace provede a image se uloží do Azure Container Registry.
# Lab 33

Cílem tohoto labu je vytvořit pipeline, která bude nasazovat aplikaci automaticky do testovacího prostředí.

## Krok 1 - Uložení helm chartu jako build artifact

1. Do složky `ci` zkopírujte z původního repozitáře finální Helm chart (složku `app`), který jsme vytvářeli v labu 24.

> Soubor `Chart.yaml` by měl být na cestě `ci/app/Chart.yaml`.

2. Commitněte a pushněte změnu.

3. Do build pipeline přidejte krok __Publish build artifacts__, který tuto složku vezme a prohlásí ji za build artifact.

    * Cesta k publikování bude `ci/app`

    * Název artifactu dejte `app`

4. Spusťte pipeline a zkontrolujte, že je helm chart vypublikován jako artifact.

## Krok 2 - Vytvoření deployment konfigurace

1. V Azure DevOps přejděte do sekce __Releases__ a klikněte na __New => New release pipeline__.

2. Ve výběru templates vyberte __Empty job__.

3. Stage pojmenujte __Test__.

4. V levé části klikněte na __+ Add an artifact__.

5. Vyberte pipeline a zkontrolujte, že se u ní našel artifact s názvem `app`.

6. Podívejte se, jaký to vygenerovalo __Source alias__ a někam si jej uložte (např. do Notepadu) - budeme jej za chvíli potřebovat.

7. Rozklikněte odkaz __1 job, 0 task__. 

8. Přidejte krok __Package and deploy Helm charts__.

    * Vyberte Azure subscription, ve kterém se nachází cluster. Klikněte na __Authorize__, aby se automaticky vytvořil service principal a přidělila se mu práva.

    * Zvolte resource group a Kubernetes cluster.

    * Jako namespace zadejte `$(targetNamespace)`

    * Command bude `upgrade`

    * Chart type bude __File path__
    
    * Z dialogu vyberte cestu ke složce `app`

> Nevybírejte přímo soubor `Chart.yaml` - chce to jen cestu k adresáři.

    * Jako release name zadejte `northwindstore`

    * V sekci __Set Values__ nastavte následující hodnotu s tím, že nahradíte placeholder `<SOURCE_ALIAS>` za název artifact source aliasu, který jsme si pamatovali minule.

```
ingress.hosts[0].host=$(appUrl),ingress.hosts[0].paths[0].path=/,ingress.hosts[0].paths[0].pathType=Prefix,ingress.tls[0].hosts[0]=$(appUrl),image.tag=$(Release.Artifacts.<SOURCE_ALIAS>.BuildId)
``` 

9. Na záložce __Variables__ nadefinujte proměnné

    * `appUrl` s hodnotou `helm.northwindstore.local`

    * `targetNamespace` s hodnotou `northwindstore-helm`

10. Uložte release pipeline.

11. Před spuštěním pipeline pro jistotu odinstalujte helm release z našeho namespace, kdyby náhodou nastala nějaká odchylka mezi helm chartem chystaným ručně a vzorovým, který je v repozitáři.

```
helm uninstall northwindstore --namespace northwindstore-helm
```

> Pokud jste release pojmenovali jinak, název releasu můžete zobrazit pomocí příkazu `helm list --namespace northwindstore-helm`.

12. Spusťte pipeline a počkejte, až doběhne.

13. Pomocí `kubectl describe pod/<POD_NAME>` ověřte, že se do chartu dosadila správná verze image (že tam není 1.0).

14. Otestujte, jestli se web nasadil a jestli funguje:

    * https://helm.northwindstore.local/

## Krok 3 - Volitelný - Dvě prostředí

Můžete si vyzkoušet odstranit aplikace z namespaců `northwindstore-test` a `northwindstore-prod` a upravit release pipeline, aby měla dvě stage (__Test__ a __Production__) s tím, že každé nasazuje do správného namespace.

    * Stage __Test__ se může nasazovat automaticky po doběhnutí build pipeline.

    * Stage __Production__ se může nasazovat jen manuálně.

    * Pro produkční stage by bylo vhodné upravit Helm template tak, aby volitelně uměla přidat ingress zajišťující redirect na verzi s `www`.
      Pomocí __Set Values__ pak pro produkční stage bude možné tuto funkci zapnout a specifikovat doménu, ze které se redirectuje.
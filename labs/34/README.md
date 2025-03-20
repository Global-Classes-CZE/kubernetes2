# Lab 34

Cílem tohoto labu je nakonfigurovat službu __Azure Monitor__ a ověřit, že se sbírají jak logy z clusteru, tak z aplikace.

## Krok 1 - Založení monitoru pro cluster

1. V Azure portálu (https://portal.azure.com) najděte svůj Kubernetes cluster.

2. V levém menu vyberte sekci __Insights__.

3. Klikněte na tlačítko __Configure Azure Monitor__ a počkejte, než operace doběhne.

> Pokud v tomto kroku dostanete chybu _MissingSubscriptionRegistration_, je třeba v portálu otevřít příslušné _Subscription_ a na záložce __Resource providers__ zaregistrovat providera `Microsoft.OperationsManagement`.

4. Po vytvoření prozkoumejte sekci __Insights__.

5. Klikněte na tlačítko __Recommended alerts__ a prozkoumejte alerty, které Azure doporučuje.

> Zajímavé jsou třeba __Failed Pod counts__ nebo __Restarting container count__.

## Krok 2 - Založení Application Insights pro aplikaci

1. V Azure portálu klikněte na možnost vytvoření nového resource a vyhledejte službu __Application Insights__.

2. Vyberte subscription a resource group.

3. Insights pojmenujte například `northwindstore-insights`.

4. Ponechte defaultní hodnoty a založte službu.

5. Po založení zkopírujte z hlavní obrazovky vygenerovaný __Instrumentation key__.

6. Otevřete helm chart (ve složce `ci/app`) a doplňte do kontejneru environment proměnnou `ApplicationInsights__InstrumentationKey`. Navažme ji na secret set s názvem `app` (stejně jako connection string) a klíčem `ApplicationInsights__InstrumentationKey` (stejný jako proměnná, ať se to neplete).

7. Pomocí `kubectl` přidáme instrumentation key do secret setu. Následujícím příkaz stáhne definici secretu a otevře ji v textovém editoru:

```
kubectl edit secret app --namespace northwindstore-helm
```

8. Hodnoty secretů jsou zakódované pomocí Base64. Hodnotu zaenkódujte a vložte ji do YAMLu:

```
# PowerShell
[System.Convert]::ToBase64String([System.Text.Encoding.ASCII]::GetBytes("<INSTRUMENTATION_KEY>"))

# Browser Dev Tools nebo "node -i"
btoa("<INSTRUMENTATION_KEY>")
```

9. Po uložení souboru v notepadu jej `kubectl` automaticky pošle do clusteru.

10. Přenasaďte aplikaci, aby začala logovat.

## Krok 3 - Test logů a vlastních metrik

1. V Azure portálu otevřete __Application Insights__ a ověřte v sekci __Live metrics__, že se Application Insights napojí.

> Také v grafu vidíte jeden HTTP request každých 10 sekund? Kdopak je asi dělá?

2. Zkuste do aplikace udělat pár requestů a ověřte, že jsou vidět.

3. Zkuste se přihlásit a ověřit, že vidíte trace __Successful login__.

4. Zkuste nasimulovat chybu (třeba pokusem o smazání nějakého regionu - kvůli cizím klíčům v databázi to nejde).

5. Vidíte v grafu __Outgoing requests / Dependency Call Rate__ počty dotazů do databáze?

6. V levém panelu vyberte sekci __Transaction search__ a zobrazte transakce za posledních 30 minut.

7. Ověřte, že sem padají logy, které jsou standardně vidět i na konzoli (např. __Successful login__).

8. Ověřte, že dostáváme custom event __Unsuccessful login__. 

> Data na této stránce mohou mít 1 - 2 minuty zpoždění.

9. Prohlédněte si sekci __Failures__ a zkontrolujte, že nám k exceptions chodí i stack trace. 

10. Zároveň můžete vidět, že u chyby v sekci __Dependencies__ vidíte i informace o HTTP requestu, tak informace o volání databáze.

11. Proklikejte všechny stránky v aplikaci a prozkoumejte sekci __Performance__.

12. V sekci __Events__ najděte ve filtrech vlastní událost __Unsuccessful login__ a vyneste ji do grafu.

13. Nastavte alert, který vás upozorní, pokud nastane více než 5 neplatných pokusů o přihlášení během 5 minut.

  * V signálech vyberte __Custom log search__

  * Jako dotaz zadejte `customMetrics | where name == "Unsuccessful login"

  * Nastavte agregaci na 5 minut a interval vyhodnocování také.

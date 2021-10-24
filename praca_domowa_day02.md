* Odpal deployment o następujących parametrach:
```yaml
name: ex-deployment
replicas: 10
image: nginx:1.20.1
strategy:
  type: Recreate
```
* Wykonaj upgrade werji nginx na: `1.20.0-alpine`
* Obseruj jak idzie upgrade, np poprzez: `kubectl get rs -w`
* Sprawdź czy wszystko działa logując się na jeden z podów i wykonując: `curl http://localhost`
* Teraz zmień strategię na:
```yaml
strategy:
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
  type: RollingUpdate
```
* Ponownie wykonaj upgrade poprzez zmianę:
```yaml
command:
  - /bin/sh
  - -c
  - while true; do echo $(date) - no nginx this time; sleep 10; done 
```
* Ponownie obseruj jak działa nowa strategia
* Sprawdz logi jednego z podów
* Na koniec zmień liczbę replik na 3 i zostaw zadnie do sprawdzenia na następny dzień
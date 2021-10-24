### InitConatiners
1. Utwórz definicję poda poprzez uruchomienie:
```shell
kubectl run init-container-test --image busybox -o yaml --dry-run=client > pod.yaml
```
2. Zmień plik `pod.yaml` następująco:
* dodaj do `spec`:
```yaml
  initContainers:
  - image: busybox
    name: my-init-container
    command:
    - /bin/sh
    - -c
    - "echo Running fake DB initialization; exit $(( RANDOM % 3 ));"
```
* w `spec.containers` dodaj:
```yaml
    command:
    - /bin/tail
    - -f
    - /dev/null
```
3. Uruchom poda i od razu sprawdzaj jak się uruchamia: `kubectl create -f ./pod.yaml && kubectl get pod init-container-test -w`
4. Jak pod wstanie poprawnie, sprawdź logi dla initContainera: `kubectl logs init-container-test -c my-init-container` i usuń poda
### Multi containers
1. Uruchom:
```shell
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  volumes:
  - name: html
    emptyDir: {}
  initContainers:
  - name: init
    image: busybox
    volumeMounts:
    - name: html
      mountPath: /a
    command: ["/bin/sh", "-c", "echo $(date) Initial page > /a/index.html"]
  containers:
  - name: 1st
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: 2nd
    image: debian
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 1;
        done
EOF

```
### Joby
1. Utwórz plik: `job.yaml` o następującej zawartości:
    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: my-job
    spec:
      parallelism: 10
      template:
        metadata:
        spec:
          containers:
          - command:
            - sh
            - -c
            - exit $(( RANDOM % 5))
            image: busybox
            name: my-job
          restartPolicy: Never
    ```
    i wrzuć go na kubernetesa
2. Uruchom `kubectl get pod` a następnie `kubectl get job my-job` i przeanalizuj otrzymane wyniki
### CronJob
1. Utwórz cronjob w następujący sposób:
```shell
k create cronjob db-backup --image=busybux --schedule='0 1 * * 0' -- /scripts/db_backup.sh
```
2. Sprawdź czy czy został poprawnie dodany, poprzez: `k get cronjob db-backup` 
3. Zobacz jego definicję YAML poprzez: `kubectl get cronjob db-backup -o ...` (usupełnij kropki, właściwym formatem) 
4. Wykonaj edycję tego joba poprzez `kubectl edit cronjob db-backup` tak żeby wykonywał się 30 min po 1 zamiast 1:00
5. Odpal 'ad hoc' db backup poprzez utworzenie joba z cronJoba:
```shell
kubectl create job --from=cronjob/db-backup
```
# 🚀 Інструкція з розгортання MVP у ArgoCD

Цей документ описує покрокове розгортання мінімального робочого продукту (MVP) на базі ArgoCD, з автоматичною синхронізацією з Git-репозиторієм [`https://github.com/den-vasyliev/go-demo-app`](https://github.com/den-vasyliev/go-demo-app).

---

## 🔧 Попередні вимоги

- Kubernetes кластер (`minikube`, `kind`, GKE, EKS тощо)
- Встановлений `kubectl`
- Встановлений ArgoCD
- Доступ до GitHub репозиторію або його форк

---

## Попередження
Нажаль гіт-репозітарій містить посилання на images котрі не працюють в кластері побудлованному за допомогою поточних версій вищевказаних інструментів.
WORKAROUND: встановити все річної давності. (дивись перший крок)

## Крок 1: Встановлення prerequisites and ArgoCD

```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | TAG=v5.6.3 bash
k3d --version
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.30.0+k3s1 sh -
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
sudo kubectl create namespace argocd
sudo kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.8.16/manifests/install.yaml
sudo kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Перевірка статусу:

```bash
kubectl get pods -n argocd
```

---

## Крок 2: Логін до ArgoCD UI

1. Отримати початковий пароль:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

2. Локальний доступ:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

3. Перейти до: [https://localhost:8080](https://localhost:8080)

- **Username**: `admin`
- **Password**: <виведений на попередньому кроці>

---

## Крок 3: Підготовка Git репозиторію

У репозиторії [`go-demo-app`](https://github.com/den-vasyliev/go-demo-app) вже є Kubernetes YAML-файли у директорії `k8s/`.

---

## Крок 4: Створення ArgoCD Application

### ВАРІАНТ 1: Через UI

- `+ NEW APP`
  - **Application Name**: `demo`
  - **Project**: `default`
  - **Sync Policy**: `Automatic`
  - **Repository URL**: `https://github.com/den-vasyliev/go-demo-app`
  - **Revision**: `HEAD`
  - **Path**: `helm`
  - **Cluster URL**: `https://kubernetes.default.svc`
  - **Namespace**: `default`

### ВАРІАНТ 2: Через YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/den-vasyliev/go-demo-app
    targetRevision: HEAD
    path: helm
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Зберегти як `go-demo-app.yaml` і застосувати:

```bash
kubectl apply -f go-demo-app.yaml
```

---

## Крок 5: Перевірка синхронізації

1. Перейти в UI ArgoCD.
2. Перевірити статус додатку — **Synced** та **Healthy**.
3. Модна змінити будь-який файл у `helm/` (наприклад, `deployment.yaml`) у репозиторії. Якщо нема доступа до репо, то можна спробувати смізнити змінну в ArgoCD і натиснуті **Sync**
4. ArgoCD автоматично застосує зміни в кластері.

---

## Крок 6: Повний цикл демонстрації

1. Змінити `image` в `k8s/deployment.yaml`:

```yaml
image: denvasyliev/go-demo-app:latest
```

2. Закомітити й запушити в Git.
3. ArgoCD автоматично:
   - Визначить зміни
   - Проведе синхронізацію
   - Оновить кластер

---

## Результат

- ArgoCD відслідковує зміни у Git
- Автоматично оновлює Kubernetes кластер
- Забезпечує повноцінний CI/CD для MVP

---

> **P.S.** Можна додатково інтегрувати GitHub Actions для CI, якщо потрібно.

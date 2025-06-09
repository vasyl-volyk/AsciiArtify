# POC: Розгортання Kubernetes з k3s та встановлення ArgoCD

## Мета
Налаштувати Kubernetes кластер за допомогою k3s, встановити ArgoCD та забезпечити доступ до його графічного інтерфейсу для команди.

---

## Передумови
- Linux-машина (або WSL, або VM)
- Права адміністратора (root або sudo)
- Встановлений `curl` та `kubectl`

---

## Крок 1: Встановлення k3s
```bash
curl -sfL https://get.k3s.io | sh -
```

**Примітка:** У деяких системах `k3s` встановлюється як `k3s.service`, а в інших — як `k3s-agent.service` або без `systemd` взагалі (наприклад, у WSL).

Перевірка статусу:
```bash
# Для systemd-сумісних систем
sudo systemctl status k3s

# Якщо сервіс не знайдено, перевірити запущений процес:
ps aux | grep k3s
```

Перевірка роботи кластеру:
```bash
sudo k3s kubectl get nodes
```

Щоб використовувати звичайну команду `kubectl`:
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

---

## Крок 2: Встановлення ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## Крок 3: Налаштування доступу до ArgoCD UI

### Відкриття доступу через port-forward:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Після цього UI буде доступний за адресою:
```
https://localhost:8080
```

### Отримання пароля адміністратора:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
```

Логін:
- Username: `admin`
- Password: (результат з попередньої команди)

---

## Крок 4: (Опційно) Встановлення CLI ArgoCD
```bash
VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep -Po '"tag_name": "\K.*?(?=")')
curl -sSL -o argocd "https://github.com/argoproj/argo-cd/releases/download/${VERSION}/argocd-linux-amd64"
sudo install -m 555 argocd /usr/local/bin/argocd
```

Логін через CLI:
```bash
argocd login localhost:8080 --username admin --password <your-password> --insecure
```

---

## Результат
Система ArgoCD розгорнута в кластері k3s. Доступ до графічного інтерфейсу працює локально через port-forwarding. Готово до налаштування застосунків CI/CD.

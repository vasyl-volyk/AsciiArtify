# kubeplugin

`kubeplugin` — це простий плагін для `kubectl`, який виводить статистику використання CPU та пам'яті для ресурсів Kubernetes у вигляді таблиці.

---

## Опис

Плагін використовує команду `kubectl top` для отримання статистики по ресурсах (pods, nodes тощо) у вказаному namespace та виводить дані у форматі CSV:

```
Resource, Namespace, Name, CPU, Memory
```

---

## Встановлення

1. Збережіть скрипт у файл, наприклад, `kubectl-kubeplugin` (назва повинна починатися з `kubectl-` щоб `kubectl` розпізнав його як плагін).

2. Зробіть скрипт виконуваним:
   ```bash
   chmod +x kubectl-kubeplugin
   ```

3. Помістіть файл у ваш `$PATH`, наприклад:
   ```bash
   mv kubectl-kubeplugin /usr/local/bin/
   ```

---

## Використання

```bash
kubectl kubeplugin <namespace> <resource_type>
```

### Аргументи

- `<namespace>` — Kubernetes namespace (наприклад, `default`, `kube-system`).
- `<resource_type>` — тип ресурсу (наприклад, `pods`, `nodes`).

### Приклади

- Вивести статистику CPU та пам'яті для всіх pod у namespace `default`:
  ```bash
  kubectl kubeplugin default pods
  ```

- Вивести статистику для node у namespace `kube-system`:
  ```bash
  kubectl kubeplugin kube-system nodes
  ```

---

## Довідка

Для перегляду довідки виконайте:

```bash
kubectl kubeplugin -h
```
або
```bash
kubectl kubeplugin --help
```

---

## Вимоги

- Kubernetes CLI (`kubectl`) має бути встановлений і налаштований.
- Повинна бути доступна команда `kubectl top` (метрики CPU і пам'яті повинні збиратись у кластері).

---

## Ліцензія

Цей проект відкритий і може бути використаний вільно.

---

Якщо потрібна допомога або є ідеї для покращень — звертайтеся!

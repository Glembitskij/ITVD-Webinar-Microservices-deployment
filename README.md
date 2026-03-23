# ☸️ ITVD Webinar: Microservices Deployment in Kubernetes

Цей репозиторій містить інструкції та Kubernetes-маніфести для розгортання мікросервісної архітектури, представленої на вебінарі:

> **"Мікросервісна архітектура на .NET: від коду до Kubernetes кластера"**

---

## 🏗️ Структура репозиторію

Репозиторій організований за типами ресурсів Kubernetes:

- 📂 **Namespace** — ізоляція середовища  
- 📂 **Secret** — конфіденційні дані (паролі, connection strings)  
- 📂 **Deployment** — запуск застосунків і SQL Server  
- 📂 **Service** — мережевий доступ до сервісів  

---

## 🚀 Крок 1: Підготовка кластера

Переконайтеся, що ваш Kubernetes контекст встановлено (наприклад, `docker-desktop` або інший локальний кластер).

### 🔹 Створення Namespace

```bash
kubectl apply -f ./Namespace/MainNamespace.yaml
```

---

### 🔹 Налаштування Secret

Перевірте файл:

```
./Secret/SqlSecret.yaml
```

Переконайтеся, що Base64 значення коректні.

```bash
kubectl apply -f ./Secret/SqlSecret.yaml
```

---

## 🗄️ Крок 2: База даних (SQL Server)

Розгортання SQL Server 2022 у кластері:

```bash
kubectl apply -f ./Deployment/Sql.yaml
kubectl apply -f ./Service/Sql.yaml
```

---

### 🔐 Доступ ззовні (SSMS)

Використовується NodePort:

- **Host:** localhost,31433  
- **User:** sa  
- **Auth:** SQL Server Authentication  

---

## 📦 Крок 3: Розгортання мікросервісів

Переконайтеся, що Docker-образи вже зібрані:

- `store-orders-api:v1`  
- `store-stock-api:v1`  

---

### 🔹 Deployment

```bash
kubectl apply -f ./Deployment/Stock.yaml
kubectl apply -f ./Deployment/Orders.yaml
```

---

### 🔹 Service

```bash
kubectl apply -f ./Service/Stock.yaml
kubectl apply -f ./Service/Orders.yaml
```

---

## 🔗 Взаємодія між сервісами

У Kubernetes сервіси спілкуються через внутрішній DNS:

```
http://stock-api-service
```

---

## 🌍 Доступ до Swagger (Port-Forward)

Оскільки використовується `ClusterIP`, потрібно прокинути порти:

| Сервіс      | Команда                                               | URL                          |
|------------|--------------------------------------------------------|------------------------------|
| Stock API  | kubectl port-forward svc/stock-api-service 5291:80 -n store-apps | http://localhost:5291/index.html |
| Orders API | kubectl port-forward svc/orders-api-service 7229:80 -n store-apps | http://localhost:7229/index.html |

---

## 🛠️ Корисні інструменти

Рекомендується використовувати **k9s** для управління кластером.

### 🔹 Основні команди k9s

- `:pods` — перегляд подів  
- `l` — логи контейнера  
- `s` — shell в контейнер  
- `Shift + F` — port-forward  

---

## ⚠️ Важливі зауваження

### ❗ Swagger у Release

У маніфестах встановлено:

```
ASPNETCORE_ENVIRONMENT=Development
```

### ❗ Health Checks / CrashLoopBackOff

При першому запуску:

- Orders API може перезапускатися (`CrashLoopBackOff`)
- Причина — SQL Server ще ініціалізується

✅ Це нормальна поведінка Kubernetes.

---

## ✅ Готово

Після виконання всіх кроків у вас буде працююча мікросервісна система в Kubernetes 🚀
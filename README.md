# 📖 Инструкция по запуску Java-приложения с Keycloak и Redis в Kubernetes

## 📌 1. Сборка образа

```bash
mvn clean package
docker build -t java-app:latest .
```

## 📌 2. hosts

Пропиши в hosts:
```
127.0.0.1 keycloak.local
127.0.0.1 keycloak
127.0.0.1 java-app.local
```

## 📌 3. Helm и Ingress

Скачай и установи helm, а затем ingress:

```bash
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
   helm install ingress-nginx ingress-nginx/ingress-nginx
```

## 📌 4. Настройка Keycloak вручную

Перейди на http://keycloak.local, войди с логином `admin` / `admin`, затем:

1. Создай Realm: `myrealm`
2. Создай клиента:
   - Client ID: `demo-client`
   - Client protocol: `openid-connect`
   - Client authentication: `true`
   - Valid redirect URIs: `http://java-app.local:8080/login/oauth2/code/keycloak`
   - Authentication flow: `Standart flow`
3. Создай scope если нет:
   ![Screenshot 2025-06-24 152505.png](screens%2FScreenshot%202025-06-24%20152505.png)
4. Добавь в Clients Scopes и назначь клиенту:
   ![Screenshot 2025-06-24 152525.png](screens%2FScreenshot%202025-06-24%20152525.png)
5. Пропиши client-secret из keycloak в параметре SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_KEYCLOAK_CLIENT_SECRET 
в ./java-app-chart/deployment.yaml

6. Создай пользователя и назначь ему пароль.

## 📌 5. Установка Redis, Keycloak и Java-приложения

```bash
kubectl apply -f redis-deployment.yaml
kubectl apply -f keycloak-deployment.yaml
kubectl apply -f ingress.yaml
helm install java-app ./java-app-chart
```
Или
```bash
kubectl apply -f k8s/
helm install java-app ./java-app-chart
```

## 📌 6. Проверка

1. Перейди на `/hello` — должно перекинуть на Keycloak.
2. Введи логин/пароль созданного пользователя.
3. Если все верно, должен увидеть `Hello, authenticated user!`

## 📌 7. Полезные команды

- Удаление приложения:
```bash
helm delete java-app 
```
- Удаление Keycloak:
```bash
kubectl delete -f .\k8s\keycloak-deployment.yaml
```
- Вывод всех сервисов k8s в namespace default:
```bash
kubectl get svc -n default
```
- Показать подробную информацию о сервисах
```bash
kubectl get svc -n default -o wide
```
- Пробросить порт для приложения из кластера в систему:
```bash
 kubectl port-forward svc/java-app 8081:8081
```
- Показать все поды кластера в namespace default
```bash
kubectl get pods -n default
```
- Запустить под для проверки доступности приложения в кластере:
```bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- curl http://java-app:8081
```
- Показать список доступных Ingress-контроллеров в кластере
```bash
kubectl get ingressclass
```
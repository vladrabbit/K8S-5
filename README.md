# Домашнее задание к занятию «Настройка приложений и управление доступом в Kubernetes»

------

## **Задание 1: Работа с ConfigMaps**
### **Задача**
Развернуть приложение (nginx + multitool), решить проблему конфигурации через ConfigMap и подключить веб-страницу.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами
   - `nginx`
   - `multitool`
3. **Подключить веб-страницу** через ConfigMap
4. **Проверить доступность**

### **Что сдать на проверку**
- Манифесты:
  - `deployment.yaml`
  - `configmap-web.yaml`
- Скриншот вывода `curl` или браузера

---

## **Решение 1: Работа с ConfigMaps**

- Манифесты:
    - `deployment.yaml`

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: web-app
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: web-app
    template:
        metadata:
        labels:
            app: web-app
        spec:
        containers:

        - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
            volumeMounts:
            - name: web-content
            mountPath: /usr/share/nginx/html

        - name: multitool
            image: wbitt/network-multitool
            command: ["sleep", "3600"]

        volumes:
        - name: web-content
            configMap:
            name: web-content

    ```

    - `configmap-web.yaml`

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: web-content
    data:
    index.html: |
        <!DOCTYPE html>
        <html>
        <head>
        <title>Страница из ConfigMap</title>
        </head>
        <body>
        <h1>Привет от Kubernetes!</h1>
        <p>Эта страница подключена через ConfigMap</p>
        </body>
        </html>
    ```
 
- Скриншот вывода `curl` или браузера

![SCR-1](https://github.com/vladrabbit/K8S-5/blob/main/scr/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-01%20%D0%B2%2022.34.40.png)

---
## **Задание 2: Настройка HTTPS с Secrets**  
### **Задача**  
Развернуть приложение с доступом по HTTPS, используя самоподписанный сертификат.

### **Шаги выполнения**  
1. **Сгенерировать SSL-сертификат**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```
2. **Создать Secret**
3. **Настроить Ingress**
4. **Проверить HTTPS-доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `secret-tls.yaml`
  - `ingress-tls.yaml`
- Скриншот вывода `curl -k`


---
## **Решение 2: Настройка HTTPS с Secrets** 

- Манифесты:
  - `secret-tls.yaml`

  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: tls-secret
  type: kubernetes.io/tls
  data:
    tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURHVENDQWdHZ0F3SUJBZ0lVV0FXdkU1eDBUbExoRC9rV2ZkYjUwZWxTcXVFd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0hERWFNQmdHQTFVRUF3d1JiWGxoY0hBdVpYaGhiWEJzWlM1amIyMHdIaGNOTWpZd01qQXlNRGt4TnpBMApXaGNOTWpjd01qQXlNRGt4TnpBMFdqQWNNUm93R0FZRFZRUUREQkZ0ZVdGd2NDNWxlR0Z0Y0d4bExtTnZiVENDCkFTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTGVzc1daNEZhL21qVDVFeVg5ZlhyaGoKcFh1aFBTVUVoL1BFQWVHczB1ZFhiT2tDQWpESU93aHRzaWdITHFKdmROUUJ2Y1RUN0tuV0RDT0JUQTVOQ05YNwphaEdCTVJZTXBmZEFUczl3c1NEK2l6QjI3R1JhMkU0VVNsL3BiRmVNYndJZUdScmR4eFBaQ1hXUm1yWDJSQk9ICi8weldzYXZRdWZmRWdYNk5uUWtUWXh4MkxBSGVmWFRpTzZqSWNNQmREQjZOMmNlUzArN3QzR1JXYWJRdldYNUwKVjRyVVBOcnMvSGV6ZTdENzFXdGQvSStEaWxwZVZ0TnVwT3dOanlIWmRmSnBNLzkrRS9FYnRIY2lWVjdpcmJ4dQpZQTNHeVQzd2RzRHFabkV5d2JpeEc2MmlPeU8wcUpvRHJJcHFyekQwaFYrR29YZ3h6Q0xCT2pRcGpmWThvanNDCkF3RUFBYU5UTUZFd0hRWURWUjBPQkJZRUZEZ2Ivc0JZZXcxbnNCTlRLTG5FN1NoN2dFRDJNQjhHQTFVZEl3UVkKTUJhQUZEZ2Ivc0JZZXcxbnNCTlRLTG5FN1NoN2dFRDJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdEUVlKS29aSQpodmNOQVFFTEJRQURnZ0VCQUgvQkV0R0NJUTU0QzZ6ZktZY0JjeEVYeUlpbFljbGhKRGQvZTMreHJpNWx3QWowCkJMbmJDWmFXZ2VNY0ZlbWpMTUJ0ZzV6SjI3MGZMQ2tBT1NtRVFZZ2FUS2Q0UkVsSTlIN2lsc2JZTVRPUW0zRFYKRm1PSmZKMlJVRG81OUtzTGg2d0xVVlpvRkU0d3p1YUkvb2Q0bnRPNFFoUkpRLzRzOVREeitaVVA0ZStCUWhTZgptRVhVRzFSc1pNMnh3ckZ4WWl0OWJQT2laNWJzaFBwVXdrak9DakE3UHBiSHJQL1V1MzJiakRKaS9EYzdSVVFECjlpWW1MdXYzcE5ML1d1VHJ0VHJwYTBJTFNPMS96Wms5L0dET3ZmL2QyTXU0dlFLN0kwNlZ0Z29DdzNzZDVPMUEKQ25hem90VXowOFFrd0VMRnR2WTFzWjMzcisxd3F0dHFYS1dL
    tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRQzNyTEZtZUJXdjVvMCsKUk1sL1gxNjRZNlY3b1QwbEJJZnp4QUhock5MblYyenBBZ0l3eURzSWJiSW9CeTZpYjNUVUFiM0UwK3lwMWd3agpnVXdPVFFqVisyb1JnVEVXREtYM1FFN1BjTEVnL29zd2R1eGtXdGhPRkVwZjZXeFhqRzhDSGhrYTNjY1QyUWwxCmtacTE5a1FUaC85TTFyR3IwTG4zeElGK2paMEpFMk1jZGl3QjNuMTA0anVveUhEQVhRd2VqZG5Ia3RQdTdkeGsKVm1tMEwxbCtTMWVLMUR6YTdQeDNzM3V3KzlWclhmeVBnNHBhWGxiVGJxVHNEWThoMlhYeWFUUC9maFB4RzdSMwpJbFZlNHEyOGJtQU54c2s5OEhiQTZtWnhNc0c0c1J1dG9qc2p0S2lhQTZ5S2FxOHc5SVZmaHFGNE1jd2l3VG8wCktZMzJQS0k3QWdNQkFBRUNnZ0VBQXJyUVVDT012dmFBTnVLeTArL0k2cGlnaHZ2WWVzcGNjdVVBMmlmRllxYkwKK1pLTUVjbUlCeElLU2NvQmlXeDZvZ1A4bkFaQ1NDdmtOa3JmcEg3RW1ObUp0QVRsZzl4Z1F4SnptV1dsWEVZcwpMMlkxRVREQWNqaWUrbG52d0VWWUNRSUZnWDYyVExjM0NzWkZORnNhbStlemhhTFROU2grK3cyeWx2em56cEN5CndBKzVjWUFNbnFhd2RIS2tML1B2eVZ2SjJ4cjFwYXJ6QkVhMVJFWmd5VGxocGUwa09LaS9kWTMyak9rc1hoZWkKOEpVdUFxUkN6cU9UbVZwTEdza1dsTUVlRVc3dnB1em9uWVRjdXFxOWt2NnBpVXdreDVOMzJPREYvcy9ValExbwo2TjNlTFJhYXRzaGUwbmNMdHdlTlprMUtUcXlaMHdTRnJ3bmF3VHBtOFFLQmdRRGM5RCtKczZ0dkVBM1N4S1AxCk1DU2hhSEN4elZ6eENSWmFKcDE4WHd3Z2N6aXo1QTlNTlM4SUd4L2RwcjJFTXZoYWFCQlA3L3F4bjJ4c1dXdU0KMUFZNXREeTdOcENiMmwrUlY2czIzQmxSd2ZOaU5pekNFWFpRL1QxZ1ZEak1NSUlocFpaekVndTNGUW13Y3g1WQppbmtydEt5bWsxM1BoWEpzaThwYzZmTnY1d0tCZ1FEVXpybW54QlVQeVdudktjU2s3MVZsbzlQRTZzQjIrSnZ0CnJrbWJFZklPa1ptQnFqM09BWHFqVkZGYkRQSENib29vcXRYbDFJQ3pyM0NNYjNFQWNZVVR6cys4K2hPT3VwaUQKTlNldG5id25lelR3TDNqa0J4Ynlyd01BOXl5WE5Kelo1Sjg0VDN2SnV0WFJTdGtyaHB2VUw2bjc5RndBNjRxTAppZ0VteHBnQWpRS0JnR0hEMlJGTzVINEI5bnZaOGtvZEFUaENCQXRJT09XV0JjUGg3akVIeFUvZWE2cDlNSitoCnNLdS9oTHdJZVRhemJ0eGh5MFh6ZzFOd25RTGNGaEI4Q0Qwa0dQTWxVNXNDWnVMaWphbDZmZUdGRmZIUTBzRVUKQk93VkFVRk1RczFtY1UzOS9MSHh2Q2xJTDc5WlVJWVF6MGlkYXY5Um1XS25RMWZ3Q3B4T0VCN2xBb0dBZWs1OQp3WUFlb3I0ZDFranBMZW9uNkl5cHo0a2tLTHhsMGNyVG52NUhZandvUDYrNmFjWEwyRWREb3RManQ5MlVKaDlaClpBZ29HQjJDMEJQVW5HNmlEMnBUVnNkYnFqSndLU2pKcnl6eTBMWXRETVliOHVKb08vNTkrWFlWK0tsU1pLRFQKS2FmMCsxSVlSWHVCS1ZUcUJwK0dVTHAyamtqUmpiVTVTREhuZHBrQ2dZRUFuUHQwV24rYWVWNnZRUTJLQnRsaQpQNStNbUdxY1hEdnNueWxuYVZWUWppWEU1TmtSV05LL0UxaWFyVHZFdHlxdnFXOGpodTZtdmZEU0hOTEZ5OEFiCkg3bVZsVGtsaGNReWtHaS9GZFg0ejVFdEh4R3NseTVaSUVCUk5QbTBRYndpQVB5WXpSNnp1WVVaQmVQbTM2M00KMVlmaVk4ZTVZUHBXa3NzOGIxYVVGWG89Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
  ```

  - `ingress-tls.yaml`

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: web-app-ingress
  spec:
    ingressClassName: nginx
    tls:
    - hosts:
      - myapp.example.com
      secretName: tls-secret
    rules:
    - host: myapp.example.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: web-app
              port:
                number: 80
  ```

- Скриншот вывода `kubectl get svc -n ingress-nginx`

![SCR-2](https://github.com/vladrabbit/K8S-5/blob/main/scr/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-02%20%D0%B2%2012.49.11.png)

- Скриншот вывода `curl -k`

![SCR-3](https://github.com/vladrabbit/K8S-5/blob/main/scr/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-02-02%20%D0%B2%2012.47.54.png)

---
## **Задание 3: Настройка RBAC**  
### **Задача**  
Создать пользователя с ограниченными правами (только просмотр логов и описания подов).

### **Шаги выполнения**  
1. **Включите RBAC в microk8s**
```bash
microk8s enable rbac
```
2. **Создать SSL-сертификат для пользователя**
```bash
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN={ИМЯ ПОЛЬЗОВАТЕЛЯ}"
openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```
3. **Создать Role (только просмотр логов и описания подов) и RoleBinding**
4. **Проверить доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `role-pod-reader.yaml`
  - `rolebinding-developer.yaml`
- Команды генерации сертификатов
- Скриншот проверки прав (`kubectl get pods --as=developer`)

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment с ConfigMap (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config # ПОДКЛЮЧЕНИЕ ConfigMap
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config # УКАЖИТЕ имя созданного ConfigMap
```
### **2. ConfigMap для веб-страницы**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-content # ИЗМЕНИТЕ: Укажите имя ConfigMap
  namespace: default # ОПЦИОНАЛЬНО: Укажите namespace, если не default
data:
  # КЛЮЧЕВОЙ МОМЕНТ: index.html будет подключен как файл
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Страница из ConfigMap</title> # ИЗМЕНИТЕ: Заголовок страницы
    </head>
    <body>
      <h1>Привет от Kubernetes!</h1> # ДОБАВЬТЕ: Свой контент страницы
    </body>
    </html>
```

### **3. Secret для TLS-сертификата**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret # ИЗМЕНИТЕ при необходимости
type: kubernetes.io/tls
data:
  tls.crt: # ЗАМЕНИТЕ на base64-код сертификата (cat tls.crt | base64 -w 0)
  tls.key: # ЗАМЕНИТЕ на base64-код ключа (cat tls.key | base64 -w 0)
```
### **4. Role для просмотра подов**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-viewer # ИЗМЕНИТЕ: Название роли
  namespace: default # ВАЖНО: Role работает только в указанном namespace
rules:
- apiGroups: [""] # КЛЮЧЕВОЙ МОМЕНТ: "" означает core API group
  resources: # РАЗРЕШЕННЫЕ РЕСУРСЫ:
    - pods # Доступ к просмотру подов
    - pods/log # Доступ к логам подов
  verbs: # РАЗРЕШЕННЫЕ ДЕЙСТВИЯ:
    - get # Просмотр отдельных подов
    - list # Список всех подов
    - watch # Мониторинг изменений
    - describe # Просмотр деталей
# ДОПОЛНИТЕЛЬНО: Можно добавить больше правил для других ресурсов
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать:
   - Скриншоты вывода команд `kubectl`
   - Скриншоты результатов выполнения
   - Тексты манифестов или ссылки на них
3. Для заданий с TLS приложите команды генерации сертификатов

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).

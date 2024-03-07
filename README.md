# Домашнее задание к занятию «Helm»

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение, например, MicroK8S.
2. Установленный локальный kubectl.
3. Установленный локальный Helm.
4. Редактор YAML-файлов с подключенным репозиторием GitHub.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://helm.sh/docs/intro/install/) по установке Helm. [Helm completion](https://helm.sh/docs/helm/helm_completion/).

------

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения.
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.


<p align="center">
    <img width="1200 height="600" src="/img/helm-create-chart.png">
</p>

| Backend |
| :---: |
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "netology-app.backend.deployment.name" . }}
  labels:
    {{- include "netology-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.backend.deployment.replicas }}
  selector:
    matchLabels:
      {{- include "netology-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "netology-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Values.backend.container.name }}
          image: "{{ .Values.backend.image.name }}:{{ .Values.backend.image.tag }}"
          imagePullPolicy: {{ .Values.backend.image.pullPolicy }}
          ports:
            - containerPort: 80
              protocol: TCP
```
| Backend service |
| :---: |
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "netology-app.backend.service.name" . }}
  labels:
    {{- include "netology-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.backend.service.type }}
  ports:
    - port: {{ .Values.backend.service.port }}
      protocol: TCP
      targetPort: 80
      name: multitool
  selector:
    {{- include "netology-app.selectorLabels" . | nindent 4 }}
```

| Frontend |
| :---: |
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "netology-app.frontend.deployment.name" . }}
  labels:
    {{- include "netology-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.frontend.deployment.replicas }}
  selector:
    matchLabels:
      {{- include "netology-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "netology-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Values.frontend.container.name }}
          image: "{{ .Values.frontend.image.name }}:{{ .Values.frontend.image.tag }}"
          imagePullPolicy: {{ .Values.frontend.image.pullPolicy }}
          ports:
            - containerPort: 8080
              protocol: TCP
```
| Frontend service |
| :---: |
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "netology-app.frontend.service.name" . }}
  labels:
    {{- include "netology-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.frontend.service.type }}
  ports:
    - port: {{ .Values.frontend.service.port }}
      protocol: TCP
      targetPort: 8080
      name: web
  selector:
    {{- include "netology-app.selectorLabels" . | nindent 4 }}
```

| Values |
| :---: |
```yaml
backend:
  deployment:
    name: 
    replicas: 2
  container:
    name: multitool
    resources: {}
#      limits:
#        memory: 1024Mi
#        cpu: 500m
#      requests:
#        memory: 512Mi
#        cpu: 100m
  service:
    name:
    port: 8080
    type: ClusterIP
  image:
    name: wbitt/network-multitool
    tag: latest
    pullPolicy: IfNotPresent

frontend:
  deployment:
    name: 
    replicas: 2
  container:
    name: nginx
    resources: {}
#      limits:
#        memory: 1024Mi
#        cpu: 500m
#      requests:
#        memory: 512Mi
#        cpu: 100m
  service:
    name:
    port: 80
#   Can be one of ClusterIP, NodePort or LoadBalancer
    type: ClusterIP
  image:
    name: nginx
    tag: 1.25.4-alpine
    pullPolicy: IfNotPresent
```

| _helpers.tpl |
| :---: |
```yaml
{{- define "netology-app.selectorLabels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}

{{- define "netology-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{- define "netology-app.labels" -}}
helm.sh/chart: {{ include "netology-app.chart" . }}
{{ include "netology-app.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end -}}

{{- define "netology-app.backend.defaultName" -}}
{{- printf "backend-%s" .Release.Name -}}
{{- end -}}

{{- define "netology-app.backend.deployment.name" -}}
{{- default (include "netology-app.backend.defaultName" .) .Values.backend.deployment.name | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{- define "netology-app.backend.service.name" -}}
{{- default (include "netology-app.backend.defaultName" .) .Values.backend.deployment.name | trunc 63 | trimSuffix "-" -}}
{{- end -}}


{{- define "netology-app.frontend.defaultName" -}}
{{- printf "frontend-%s" .Release.Name -}}
{{- end -}}

{{- define "netology-app.frontend.deployment.name" -}}
{{- default (include "netology-app.frontend.defaultName" .) .Values.frontend.deployment.name | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{- define "netology-app.frontend.service.name" -}}
{{- default (include "netology-app.frontend.defaultName" .) .Values.frontend.deployment.name | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```

| Chart |
| :---: |
```yaml
apiVersion: v2
name: netology-app
version: 1.0.0
description: Netology project
type: application
sources:
  - https://github.com/bosone87
maintainers:
  - name: Oleg Baranov
    email: bos.one@mail.ru
appVersion: 1.0.0
```

<p align="center">
    <img width="1200 height="600" src="/img/helm-install-chart-debug.png">
</p>

------
### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

<p align="center">
    <img width="1200 height="600" src="/img/helm-install-app1.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/helm-install-app2.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/helm-install-app3.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/helm-list1.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/helm-list2.png">
</p>

<br>
<br>

| Set version |
| :---: |
<p align="center">
    <img width="1200 height="600" src="/img/change_version.png">
</p>

| Set app-version |
| :---: |
<p align="center">
    <img width="1200 height="600" src="/img/change_app-version.png">
</p>

### Правила приёма работы

1. Домашняя работа оформляется в своём Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, `helm`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.


# Deploy to OpenShift

```bash
oc login

cd {{service_name}}/openshift
oc apply -f service.yml
oc apply -f route.yml
oc apply -f deploy.yml
```

Deployment Config:
```yaml
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: {{service_name}}
  namespace: digital-platform
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    app: {{service_name}}
    deploymentconfig: {{service_name}}
  strategy:
    activeDeadlineSeconds: 21600
    resources: {}
    rollingParams:
      intervalSeconds: 1
      maxSurge: 25%
      maxUnavailable: 25%
      timeoutSeconds: 600
      updatePeriodSeconds: 1
    type: Rolling
  template:
    metadata:
      labels:
        app: {{service_name}}
        deploymentconfig: {{service_name}}
    spec:
      containers:
      - image: nexus.dev.gazprombank.ru:60001/{{service_name}}:latest
        imagePullPolicy: Always
        name: {{service_name}}
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        readinessProbe:
          httpGet:
            path: /{{service_name}}/actuator/health
            port: 8080
          initialDelaySeconds: 60
          timeoutSeconds: 1
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30
  test: false
  triggers:
  - type: ConfigChange
```

Route:
```yaml
apiVersion: v1
kind: Route
metadata:
  name: {{service_name}}
  namespace: digital-platform
spec:
  host: {{service_name}}-service.openshift.dev.gazprombank.ru
  port:
    targetPort: 8080-tcp
  to:
    kind: Service
    name: {{service_name}}
    weight: 100
  wildcardPolicy: None
```

Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: {{service_name}}
  name: {{service_name}}
  namespace: digital-platform
spec:
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: {{service_name}}
    deploymentconfig: {{service_name}}
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

```
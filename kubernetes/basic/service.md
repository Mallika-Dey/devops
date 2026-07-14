```bash
apiVersion: v1
kind: Service
metadata:
  name: webapp-service # service identity. ex; kubectl describe svc webapp-service
  namespace: default
spec:
  ports:
  - nodePort: 30080
    port: 8080
    targetPort: 8080 
  selector:
    name: simple-webapp # label key. Send traffic to every Pod whose label is name=simple-webapp
  type: NodePort
  ```

# Visual Flow

## For example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: mypod

  labels:
    name: simple-webapp
```

```bash
Service
metadata.name = webapp-service
        |
        |
        v
selector:
  name = simple-webapp
        |
        |
        v
Pod
metadata.name = mypod
labels:
  name = simple-webapp
```
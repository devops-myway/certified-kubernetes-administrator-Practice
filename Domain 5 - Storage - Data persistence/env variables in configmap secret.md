
######  envFrom for configmaps and Secrets

envFrom:
In general if you have specific configmaps for specific applications where your application uses all or most of the keys in the configmaps, then envFrom is obviously easier the use and maintain.

evn in container:
In an other hand, if you organize your configmaps or secret more by theme or if multiple applications need specific keys from the same configmap or secret, then configMapKeyRef is better. You will take only the keys you need in your application and ensure that nothing will be overwritten accidentally.

```sh
envFrom:
    # This might contain environment-wide settings. Like the domain name that your application uses or a production only feature flag.
  - configMapRef:
      name: production-settings
    # Here you could store all the settings of this specific application.
  - configMapRef:
      name: my-app-settings
env:
  # This might be a bucket shared by multiple applications. So you might want to keep it a different configmap and let each aplication pick the keys they need.
  - name: S3_BUCKET
    valueFrom:
      configMapKeyRef:
        name: s3-settings
        key: bucket
```

```sh
envFrom:
  - secretRef:
      name: env-secrets

# Or for loading explicit values from a secret:
env:
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-credentials
      key: password
```

A single secret or configmap may package one or more key/value pairs. The commands used are as below:
--from-file=[key=]source
--from-literal=key1=value1
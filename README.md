# Java microservice en Kubernetes

Ejemplo mínimo de un servicio REST con Java 17, Spring Boot, Maven y Kubernetes.

## Ejecutar en local

```bash
mvn spring-boot:run
```

- `GET http://localhost:8080/hello` devuelve un saludo.
- `GET http://localhost:8080/health` devuelve el estado del servicio.
- `mvn test` ejecuta las pruebas.

## CI y CD

- **CI** (`.github/workflows/ci.yml`): ejecuta `mvn verify` en cada pull request y push a `main`.
- **CD** (`.github/workflows/cd.yml`): tras un CI correcto en `main`, construye la imagen, la publica en GHCR y actualiza Kubernetes.

Para habilitar el despliegue:

1. Configura el secreto del repositorio `KUBE_CONFIG_DATA` con el contenido de `base64 -w 0 ~/.kube/config` (en PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("$HOME/.kube/config"))`). El kubeconfig debe permitir aplicar recursos en el clúster.
2. Tras la primera publicación, cambia la visibilidad del paquete de GHCR a **Public** para que el clúster pueda descargar la imagen sin credenciales.
3. Usa `main` como rama de despliegue. El workflow aplica los manifiestos de `k8s/` y espera a que termine el rollout.

El Service es interno (`ClusterIP`); para probarlo con un clúster activo puedes ejecutar `kubectl port-forward service/java-microservice 8080:80`.
# Java microservice en Kubernetes

Ejemplo mínimo de un servicio REST con Java 17, Spring Boot, Maven y Kubernetes.

## Ejecutar en local

Maven resuelve `payment-utils` directamente desde GitHub Packages, según el repositorio y las coordenadas declaradas en `pom.xml`.

```bash
mvn -U clean verify
mvn spring-boot:run
```

- `GET http://localhost:8080/hello` devuelve `Hello from Kubernetes - PAYMENT_PROVIDER=LEGIT`.
- `GET http://localhost:8080/health` devuelve el estado del servicio.
- `mvn test` ejecuta las pruebas.

`test-dependency` publica `com.ekoparty.demo:payment-utils:1.0-SNAPSHOT` en `https://maven.pkg.github.com/drew2hip544/test-dependency`. Cada `mvn deploy` publica un nuevo snapshot bajo esa misma coordenada. El consumidor consulta metadata actualizada (`updatePolicy=always` y `-U`) para resolver el snapshot más reciente; los workflows no cachean el repositorio Maven.

Para ejecutar Maven localmente, configura `%USERPROFILE%\.m2\settings.xml` con un personal access token clásico que tenga permiso `read:packages` y un servidor cuyo `id` sea `github`. No guardes ese token en el repositorio:

```xml
<settings>
	<servers>
		<server>
			<id>github</id>
			<username>GITHUB_USERNAME</username>
			<password>CLASSIC_PAT_WITH_READ_PACKAGES</password>
		</server>
	</servers>
</settings>
```

En GitHub Actions, `setup-java` genera esa configuración usando `GITHUB_TOKEN`. Da acceso de lectura al repositorio `ms-kubernetes-lab` desde la configuración del paquete en `test-dependency`; el paquete debe haberse publicado antes de ejecutar el build.

## CI y CD

- **CI** (`.github/workflows/ci.yml`): ejecuta `mvn -U verify`; Maven descarga `payment-utils` desde GitHub Packages en cada pull request y push a `master`.
- **CD** (`.github/workflows/cd.yml`): tras un CI correcto en `master`, construye la imagen, la publica en GHCR y actualiza Kubernetes.

Para habilitar el despliegue:

1. Configura el secreto del repositorio `KUBE_CONFIG_DATA` con el contenido de `base64 -w 0 ~/.kube/config` (en PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("$HOME/.kube/config"))`). El kubeconfig debe permitir aplicar recursos en el clúster.
2. Tras la primera publicación, cambia la visibilidad del paquete de GHCR a **Public** para que el clúster pueda descargar la imagen sin credenciales.
3. Usa `master` como rama de despliegue. El workflow aplica los manifiestos de `k8s/` y espera a que termine el rollout.

El Service es interno (`ClusterIP`); para probarlo con un clúster activo puedes ejecutar `kubectl port-forward service/java-microservice 8080:80`.
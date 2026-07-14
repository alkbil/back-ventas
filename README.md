# Backend Ventas — Innovatech Chile

API REST para la gestión de ventas de Innovatech Chile.

## Tecnologías
- Java 17 + Spring Boot 3
- Maven
- MySQL 8.0
- Docker (multi-stage build)

## Endpoints principales
- `GET /api/v1/ventas` — Listar todas las ventas
- `GET /api/v1/ventas/{id}` — Obtener venta por ID
- `PUT /api/v1/ventas/{id}` — Actualizar venta
- Swagger UI: `/swagger-ui.html`

## Estructura del repositorio
back-Ventas_SpringBoot/
├── Springboot-API-REST/    # Código fuente Spring Boot
│   ├── src/
│   ├── Dockerfile          # Multi-stage build
│   └── pom.xml
├── k8s/                    # Manifiestos Kubernetes
│   ├── backend-ventas-deployment.yaml
│   ├── backend-ventas-service.yaml
│   └── backend-ventas-hpa.yaml
└── .github/workflows/
└── deploy.yml          # Pipeline CI/CD

## Variables de entorno
| Variable | Descripción |
|----------|-------------|
| `SPRING_DATASOURCE_URL` | URL de conexión MySQL |
| `DB_USERNAME` | Usuario de la base de datos |
| `DB_PASSWORD` | Contraseña de la base de datos |

## Pipeline CI/CD
El pipeline se activa con push a `main` o `deploy`:
1. **Build** → Compila JAR con Maven y construye imagen Docker
2. **Push** → Publica imagen en Amazon ECR
3. **Apply** → Aplica manifiestos Kubernetes en EKS
4. **Deploy** → Ejecuta rollout restart en el cluster

## Infraestructura AWS
- **Cluster EKS:** `innovatech-eks` (us-east-1)
- **Namespace:** `innovatech`
- **ECR:** `830985015694.dkr.ecr.us-east-1.amazonaws.com/backend-ventas-innovatech`
- **Service:** ClusterIP (accesible solo internamente)
- **Puerto:** 8080
- **HPA:** mín 2 réplicas, máx 5, umbral CPU 50%

## Secrets requeridos en GitHub
| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Credencial AWS |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS |
| `AWS_SESSION_TOKEN` | Token de sesión AWS |

## Despliegue local
```bash
docker-compose up --build
```
## Arquitectura de Despliegue

Este servicio (Backend Ventas) forma parte de la plataforma Innovatech, compuesta por un
Frontend (React/Vite + NGINX), dos microservicios Backend (Spring Boot: Ventas y
Despachos) y una base de datos MySQL, desplegados en un cluster Amazon EKS. El diagrama
completo de arquitectura y los manifiestos de Kubernetes se encuentran centralizados en
el repositorio front-despacho (carpeta k8s/).

API expuesta en el puerto 8080 (ver Swagger UI en /swagger-ui.html), consumida
internamente por el Frontend y conectada a MySQL mediante variables de entorno
inyectadas desde un Secret de Kubernetes (DB_ENDPOINT, DB_PORT, DB_NAME, DB_USERNAME,
DB_PASSWORD).

## Desarrollo local con Docker Compose

El archivo docker-compose.yml para levantar el stack completo (frontend, ambos backends
y mysql) se encuentra en el repositorio front-despacho.

## Despliegue en AWS EKS

- Cluster: Innovatech-eks (Kubernetes v1.36, EKS Auto Mode)
- Namespace: innovatech
- Imagen publicada en Amazon ECR: backend-ventas-innovatech
- Deployment con 1-3 replicas mediante Horizontal Pod Autoscaler (CPU 50%)

### Evidencia de funcionamiento

Pods corriendo:

NAME                                 READY   STATUS    RESTARTS   AGE
backend-ventas-6969747957-5jctj      1/1     Running   2          97s
backend-ventas-6969747957-m8hrc      1/1     Running   2          97s
backend-ventas-6969747957-wr47r      1/1     Running   2          97s

Prueba funcional via Swagger UI (POST /api/v1/ventas):
Respuesta 201 Created:

{
  "idVenta": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 25000,
  "fechaCompra": "2026-07-14",
  "despachoGenerado": false
}

## Observabilidad

Se verifico el funcionamiento mediante kubectl logs, detectando y corrigiendo un error de
conexion JDBC (Public Key Retrieval is not allowed) causado por el metodo de
autenticacion por defecto de MySQL 8. Metricas de escalado disponibles via
kubectl get hpa -n innovatech (backend-ventas-hpa: 1-3 replicas, target CPU 50%).

## CI/CD Pipeline

El workflow de GitHub Actions ejecuta build -> test -> build de imagen Docker -> push a
Amazon ECR (etiquetada con el SHA del commit) -> despliegue en EKS mediante kubectl apply.
Las credenciales se gestionan mediante GitHub Secrets.

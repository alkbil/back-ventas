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
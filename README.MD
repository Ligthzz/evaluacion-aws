# Evaluación Parcial N°3 — Innovatech Chile
## ISY1101 — Introducción a Herramientas DevOps

## Integrantes
- Joaquin Medina

## Descripción
Implementación de una arquitectura de orquestación de contenedores en AWS EKS con pipeline CI/CD automatizado usando GitHub Actions. El sistema despliega una aplicación de gestión de despachos y ventas de forma escalable, tolerante a fallos y automatizada.

## Arquitectura
Internet → ALB → Frontend (EKS) → Backend Despachos / Backend Ventas → MySQL
## Tecnologías utilizadas
- **AWS EKS** — Orquestación de contenedores
- **AWS ECR** — Registro de imágenes Docker
- **AWS VPC** — Red privada virtual
- **Docker** — Contenedorización
- **Kubernetes** — Orquestación
- **GitHub Actions** — Pipeline CI/CD
- **Spring Boot** — Backend (Java 17)
- **React + Vite** — Frontend
- **MySQL 8.0** — Base de datos

## Infraestructura AWS
| Recurso | Detalle |
|---------|---------|
| VPC | `10.0.0.0/16` |
| Subnets públicas | 2 (AZ-a y AZ-b) |
| Subnets privadas | 2 (AZ-a y AZ-b) |
| NAT Gateway | 1 (AZ-a) |
| Internet Gateway | 1 |
| Security Groups | ALB SG + EKS SG |
| EKS Cluster | `innovatech-cluster` |
| Node Group | `t3.medium` x2 |
| ECR Repositories | 3 (front, back-despachos, back-ventas) |

## Servicios desplegados
| Servicio | Puerto | Tipo |
|---------|--------|------|
| Frontend | 80 | LoadBalancer |
| Backend Despachos | 8081 | ClusterIP |
| Backend Ventas | 8080 | ClusterIP |
| MySQL | 3306 | ClusterIP |

## URL pública
http://a14d496b78d784a0eb8d76b8da82a504-360357626.us-east-1.elb.amazonaws.com
## Pipeline CI/CD
El pipeline se activa automáticamente con cada `git push` a la rama `main`:

1. **Build** — Construye la imagen Docker
2. **Push** — Sube la imagen a ECR
3. **Deploy** — Actualiza el deployment en EKS

### Workflows
- `deploy-back-despachos.yml` — Se activa cuando hay cambios en `back-Despachos_SpringBoot/`
- `deploy-back-ventas.yml` — Se activa cuando hay cambios en `back-Ventas_SpringBoot/`
- `deploy-frontend.yml` — Se activa cuando hay cambios en `front_despacho/`

## Autoscaling (HPA)
| Servicio | Min Pods | Max Pods | CPU Umbral |
|---------|----------|----------|------------|
| Backend Despachos | 1 | 4 | 70% |
| Backend Ventas | 1 | 4 | 70% |
| Frontend | 1 | 4 | 70% |

Se eligió el 70% de CPU como umbral porque permite escalar antes de saturar los nodos, manteniendo un margen de rendimiento adecuado.

## Manifiestos Kubernetes
Los manifiestos se encuentran en la carpeta `/k8s`:
- `namespace.yaml` — Namespace `innovatech`
- `mysql-deployment.yaml` — Base de datos MySQL
- `mysql-service.yaml` — Servicio MySQL
- `backend-despachos-deployment.yaml` — Deployment backend despachos
- `backend-despachos-service.yaml` — Servicio backend despachos
- `backend-ventas-deployment.yaml` — Deployment backend ventas
- `backend-ventas-service.yaml` — Servicio backend ventas
- `frontend-deployment.yaml` — Deployment frontend
- `frontend-service.yaml` — Servicio frontend (LoadBalancer)
- `backend-despachos-hpa.yaml` — HPA backend despachos
- `backend-ventas-hpa.yaml` — HPA backend ventas
- `frontend-hpa.yaml` — HPA frontend

## Cómo desplegar
### Requisitos
- AWS CLI configurado
- kubectl instalado
- eksctl instalado
- Docker Desktop

### Pasos
1. Configurar credenciales AWS:
```bash
aws configure
aws configure set aws_session_token TU_TOKEN
```

2. Conectar kubectl al clúster:
```bash
aws eks update-kubeconfig --region us-east-1 --name innovatech-cluster
```

3. Desplegar manifiestos:
```bash
cd k8s
kubectl apply -f namespace.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-service.yaml
kubectl apply -f backend-despachos-deployment.yaml
kubectl apply -f backend-despachos-service.yaml
kubectl apply -f backend-ventas-deployment.yaml
kubectl apply -f backend-ventas-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f backend-despachos-hpa.yaml
kubectl apply -f backend-ventas-hpa.yaml
kubectl apply -f frontend-hpa.yaml
```

4. Verificar pods:
```bash
kubectl get pods -n innovatech
```

5. Obtener URL pública:
```bash
kubectl get svc -n innovatech
```

## Secrets GitHub Actions
| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | ID de acceso AWS |
| `AWS_SECRET_ACCESS_KEY` | Clave secreta AWS |
| `AWS_SESSION_TOKEN` | Token de sesión AWS |
| `AWS_ACCOUNT_ID` | ID de cuenta AWS |
| `AWS_REGION` | Región AWS |
| `EKS_CLUSTER_NAME` | Nombre del clúster EKS |

## Decisiones técnicas
- **EKS sobre ECS** — Se eligió EKS por su mayor flexibilidad y soporte nativo de Kubernetes
- **t3.medium** — Se eligió este tipo de instancia para soportar los contenedores de CloudWatch y OpenTelemetry
- **1 NAT Gateway** — Se eligió un solo NAT Gateway para optimizar costos en el entorno de laboratorio. En producción real se usaría uno por AZ
- **70% CPU en HPA** — Permite escalar antes de saturar los recursos manteniendo margen de rendimiento
- **ClusterIP para backends** — Los backends no necesitan IP pública, solo el frontend. Esto mejora la seguridad
- **MySQL en EKS** — Se desplegó MySQL dentro del clúster para simplificar la arquitectura y evitar costos de RDS



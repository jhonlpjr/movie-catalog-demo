# 🎬 Movie Catalog — Monorepo (API + Web + Infra)

Complete application to **catalog, search and manage movies** with a **.NET REST API** and a **Next.js frontend**, orchestrated via **Docker Compose**. This monorepo also includes a **professional Terraform deployment plan** for AWS (ECS Fargate, ALB, ECR, ElastiCache for Redis, and MongoDB Atlas).

> Goal: demonstrate senior-level backend, architecture, DevOps, and IaC skills with a production‑ready structure, while keeping local setup simple.

---

## 📦 Modules

- **`/api`** — .NET Web API (MongoDB + Redis caching, Swagger, layered/clean architecture).
- **`/web`** — Next.js app (search, filters, CRUD, React Query, Tailwind).
- **`/docker`** — Compose files for local and production-like runs.
- **`/infra/terraform`** — IaC for AWS + MongoDB Atlas (modular structure, remote state, CI-friendly).

---

## ✨ Key Features

**Backend**
- REST endpoints for listing, searching, retrieving and managing movies.
- **Redis** caching for popular/recommended endpoints.
- **Swagger** for API documentation.
- Multi-environment ready (`local`, `dev`, `qa`, `prod`).

**Frontend**
- List, search (with debounce), filters and pagination.
- CRUD (create/edit/delete) with modals and confirmation flows.
- Responsive UI using **Tailwind** + **@tanstack/react-query**.

**Infra**
- Local development via **Docker Compose** (API + Web + Redis + Mongo).
- Cloud deployment with **Terraform** (AWS + MongoDB Atlas), following best practices: remote state, modules, per‑env configs, OIDC for CI, tagged resources, least-privileged IAM roles, and blue/green‑ready service config.

---

## 🧰 Tech Stack

- **API:** .NET Web API, MongoDB, Redis, Swagger
- **Web:** Next.js (App Router), React, TypeScript, Tailwind, React Query
- **Infra:** Docker/Compose, Terraform, AWS (ECR, ECS Fargate, ALB, VPC, ElastiCache Redis), MongoDB Atlas

---

## 🗂️ Repository Layout (suggested)

```
.
├── api/                       # Backend (.NET, Clean Architecture)
├── web/                       # Frontend (Next.js, Tailwind, React Query)
├── docker/
│   ├── local/
│   │   └── docker-compose.yml
│   └── prod/
│       └── docker-compose.yml
├── env/
│   ├── .env.local.example
│   ├── .env.dev.example
│   ├── .env.qa.example
│   └── .env.prod.example
└── infra/
    └── terraform/
        ├── modules/
        │   ├── networking/           # VPC, subnets, IGW, NAT, routes
        │   ├── ecr/                  # Registry for images
        │   ├── cluster/              # ECS cluster + capacity providers
        │   ├── api_service/          # Task def, service, autoscaling, ALB tg
        │   ├── alb/                  # Application Load Balancer + listeners
        │   ├── redis/                # ElastiCache Redis (single-node or cluster)
        │   └── dns_acm/              # (Optional) Route53 + ACM for HTTPS
        ├── envs/
        │   ├── dev/
        │   │   ├── main.tf
        │   │   ├── variables.tf
        │   │   ├── outputs.tf
        │   │   └── terraform.tfvars
        │   ├── qa/
        │   └── prod/
        └── README.md
```

> You can add a `mongodbatlas` module for Atlas projects, clusters and DB users, or keep Atlas as a pre-provisioned external dependency and only store connection strings in SSM/Secrets Manager.

---

## ⚙️ Environment Variables

Create your env files under `env/` based on these examples.

**`env/.env.local.example`**
```
# API
ENVIRONMENT=local
API_PORT=8080

# Mongo (local compose)
MONGO_URI=mongodb://mongo:27017
MONGO_DATABASE=movies_dev

# Redis (local compose)
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_INSTANCE_NAME=movieapi:
REDIS_DEFAULT_DB=0
REDIS_SSL=false
REDIS_ABORT_CONNECT=false

# Web
WEB_PORT=3000
API_BASE_URL=http://api:8080
```

**`env/.env.prod.example`**
```
# API
ENVIRONMENT=prod
API_PORT=8080

# Mongo (Atlas)
MONGO_URI=<mongodb+srv://...>
MONGO_DATABASE=movies_prod

# Redis (AWS ElastiCache)
REDIS_HOST=<redis-endpoint.amazonaws.com>
REDIS_PORT=6379
REDIS_INSTANCE_NAME=movieapi:
REDIS_DEFAULT_DB=0
REDIS_SSL=true
REDIS_ABORT_CONNECT=false

# Web
WEB_PORT=3000
API_BASE_URL=https://api.<your-domain.com>
```

---

## 🐳 Local Development with Docker Compose

**Prerequisites**
- Docker + Docker Compose

**Commands**
```bash
# 1) Build & up (detached)
ENV=local docker compose -f docker/local/docker-compose.yml up --build -d

# 2) Tail logs
docker compose -f docker/local/docker-compose.yml logs -f

# 3) Stop & remove
docker compose -f docker/local/docker-compose.yml down
```

**Services**
- API → http://localhost:8080 (Swagger at `/swagger` if enabled)
- Web → http://localhost:3000
- Redis → internal only
- Mongo → internal only

**Common gotchas**
- Frontend cannot reach API → check `API_BASE_URL` (use `http://api:8080` inside Compose; `http://localhost:8080` from host).
- Redis not caching → ensure `REDIS_HOST=redis` and service is healthy.

---

## 🔌 API — Main Endpoints

| Method | Route                           | Description                             |
|------:|----------------------------------|-----------------------------------------|
| GET   | `/api/movies`                    | List movies                             |
| GET   | `/api/movies/{id}`               | Movie details                           |
| GET   | `/api/movies/search?query=...`   | Search by title or genre                |
| GET   | `/api/movies/popular`            | Popular (Redis cached)                  |
| GET   | `/api/movies/recommendations`    | Recommendations (Redis cached)          |
| POST  | `/api/movies`                    | Create movie (demo)                     |
| PUT   | `/api/movies/{id}`               | Update movie                            |
| DELETE| `/api/movies/{id}`               | Delete movie                            |

---

## 🧪 Testing

**API**
```bash
cd api
dotnet test
```

**Web**
```bash
cd web
# Add tests under __tests__/ as needed
```

---

## 🚀 Cloud Deployment (Overview)

### Option A — Vercel for Web + AWS for API
- **Web**: deploy to Vercel, set `API_BASE_URL` to your API public URL.
- **API**: containerized service on **ECS Fargate** behind **ALB**; images stored in **ECR**; **ElastiCache Redis**; **MongoDB Atlas** for data.

### Option B — All on AWS
- **Web**: Next.js can be hosted as static on **S3 + CloudFront** (if static export) or as a Node service on **ECS Fargate** for SSR.
- **API**: ECS Fargate + ALB + ECR.
- **Stateful services**: ElastiCache (Redis), MongoDB Atlas (or EC2 self-managed Mongo — not recommended for prod).

---

## 🏗️ Terraform — Professional Setup (AWS + MongoDB Atlas)

This repo is prepared to host a **modular Terraform** stack with per‑environment configurations and a **remote backend**:

### Remote State (Backend)
We use an **S3** bucket for state and a **DynamoDB** table for state locking:

```hcl
# infra/terraform/envs/dev/backend.tf
terraform {
  backend "s3" {
    bucket         = "tfstate-movie-catalog-<account-id>"
    key            = "envs/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-locks-movie-catalog"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
}
```

> Create the S3 bucket and DynamoDB table once (manually or via a small bootstrap script). The backend itself cannot create its own state store.

### Example `main.tf` (dev environment)
```hcl
#########################################################
# Providers
#########################################################
provider "aws" {
  region = var.aws_region
}

# Optional: MongoDB Atlas provider if you manage Atlas via Terraform
# provider "mongodbatlas" {
#   public_key  = var.mongodbatlas_public_key
#   private_key = var.mongodbatlas_private_key
# }

#########################################################
# Networking (VPC, subnets, routes, nat, sg)
#########################################################
module "networking" {
  source                = "../../modules/networking"
  name_prefix           = var.name_prefix
  vpc_cidr              = "10.20.0.0/16"
  az_count              = 2
  public_subnet_cidrs   = ["10.20.1.0/24", "10.20.2.0/24"]
  private_subnet_cidrs  = ["10.20.11.0/24", "10.20.12.0/24"]
  create_nat_gateway    = true
  tags                  = var.tags
}

#########################################################
# ECR
#########################################################
module "ecr" {
  source      = "../../modules/ecr"
  name_prefix = var.name_prefix
  repos       = ["movies-api", "movies-web"]
  tags        = var.tags
}

#########################################################
# ECS Cluster
#########################################################
module "cluster" {
  source      = "../../modules/cluster"
  name_prefix = var.name_prefix
  tags        = var.tags
}

#########################################################
# ALB (public) + Listeners
#########################################################
module "alb" {
  source               = "../../modules/alb"
  name_prefix          = var.name_prefix
  vpc_id               = module.networking.vpc_id
  public_subnets       = module.networking.public_subnet_ids
  enable_https         = false # set true with dns_acm module
  tags                 = var.tags
}

#########################################################
# Redis (ElastiCache)
#########################################################
module "redis" {
  source              = "../../modules/redis"
  name_prefix         = var.name_prefix
  vpc_id              = module.networking.vpc_id
  subnet_ids          = module.networking.private_subnet_ids
  allowed_sg_ids      = [module.cluster.services_sg_id]
  node_type           = "cache.t4g.small"
  engine_version      = "7.1"
  tags                = var.tags
}

#########################################################
# API Service (ECS Fargate)
#########################################################
module "api_service" {
  source                = "../../modules/api_service"
  name_prefix           = var.name_prefix
  cluster_arn           = module.cluster.cluster_arn
  task_cpu              = 512
  task_memory           = 1024
  container_image       = "${module.ecr.repo_urls["movies-api"]}:latest"
  container_port        = 8080
  desired_count         = 1
  private_subnet_ids    = module.networking.private_subnet_ids
  lb_listener_arn_http  = module.alb.http_listener_arn
  health_check_path     = "/health"
  env_vars = {
    ENVIRONMENT           = "dev"
    MONGO_URI             = var.mongo_uri
    MONGO_DATABASE        = "movies_dev"
    REDIS_HOST            = module.redis.primary_endpoint
    REDIS_PORT            = "6379"
    REDIS_INSTANCE_NAME   = "movieapi:"
    REDIS_DEFAULT_DB      = "0"
    REDIS_SSL             = "true"
    REDIS_ABORT_CONNECT   = "false"
  }
  tags = var.tags
}
```

### Example `variables.tf`
```hcl
variable "aws_region" { type = string }
variable "name_prefix" { type = string }
variable "mongo_uri" { type = string } # from Atlas (Secret)
variable "tags" {
  type = map(string)
  default = {
    Project = "movie-catalog"
    Owner   = "platform"
    Env     = "dev"
  }
}
```

### Example `terraform.tfvars` (dev)
```hcl
aws_region  = "us-east-1"
name_prefix = "movies-dev"
mongo_uri   = "mongodb+srv://user:pass@cluster/url"
tags = {
  Project = "movie-catalog"
  Owner   = "jonathan"
  Env     = "dev"
}
```

### (Optional) DNS + ACM
Add the `dns_acm` module to request a certificate and attach HTTPS to ALB. Then set `enable_https = true` in the `alb` module and provide `certificate_arn`.

---

## 🔁 CI/CD (GitHub Actions — outline)

**Pipeline 1 — Build & Push images**
- Triggers on `main`/tags.
- `dotnet test` (API), optional web tests.
- Login to ECR via OIDC (`aws-actions/configure-aws-credentials`).
- Build & push `movies-api` and `movies-web` images with Git SHA tags.

**Pipeline 2 — Terraform Plan/Apply**
- OIDC to assume role with least‑privileged policy for Terraform.
- `terraform fmt -check`, `init`, `validate`, `plan` → upload artifact.
- **Manual approval** → `apply`.
- Store sensitive values in **GitHub Secrets** (e.g., `ATLAS_URI`) or **SSM Parameter Store/Secrets Manager** and reference them in module vars.

> Tip: pin runners to specific versions and enable drift detection (scheduled `terraform plan`).

---

## ▶️ Runbook (High-level)

1. **Local Dev**: `docker compose` up, iterate on API + Web.
2. **Images**: on merge to `main`, CI builds and pushes to ECR.
3. **Infra**: Terraform `plan`/`apply` per environment.
4. **Deploy**: ECS service picks latest image (via tag/pin). Consider blue/green or canary in production.
5. **Observability**: CloudWatch Logs for tasks and ALB access logs (S3). Add metrics/alarms (CPU, memory, 5xx).

---

## 🔒 Security Notes

- Use OIDC in GitHub Actions to avoid long‑lived AWS keys.
- Prefer **AWS Secrets Manager/SSM** for app secrets (Mongo URI, etc.).
- Restrict SG egress/ingress; keep tasks in **private subnets**; ALB in public subnets.
- Tag all resources and enable lifecycle policies on ECR.
- Enforce least privilege IAM for CI and at runtime (task role vs execution role).

---

## 🩺 Troubleshooting

- **Web cannot reach API**: check ALB target health; verify `API_BASE_URL` and security groups.
- **Redis connection errors**: verify ElastiCache endpoint, SG rules, in‑VPC connectivity.
- **Mongo timeouts**: ensure Atlas IP access list includes your NAT/Egress IPs or use VPC Peering/PrivateLink (preferred for prod).
- **Terraform locking**: if a stuck lock occurs, clear the row in DynamoDB with caution.

---

## 📜 License


All code, infrastructure, and documentation in this repository are provided for educational and professional demonstration purposes by Jonathan Reyna (jhonlpjr).

Feel free to use, adapt, or extend for your own projects. Attribution is appreciated.

---

## 🙌 Contributing


Contributions, feedback, and professional connections are welcome!

- Pull requests (PRs) are encouraged. Please use conventional commit messages.
- For infrastructure (Terraform), keep changes isolated per environment/module.
- For backend/frontend, follow clean code and architecture principles.

Contact: [LinkedIn](https://www.linkedin.com/in/jhonlpjr) · [GitHub](https://github.com/jhonlpjr)

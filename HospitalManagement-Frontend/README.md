                         HOST MACHINE
                    ┌───────────────────────┐
                    │                       │
                    │   Browser             │
                    │      │                │
                    │      ↓                │
                    │ localhost:3000        │
                    │      │                │
                    └──────┼────────────────┘
                           │
                    Docker Network
                           │
              ┌────────────┴────────────┐
              │                         │
              ↓                         │
       ┌──────────────┐                 │
       │   frontend   │                 │
       │    Nginx     │                 │
       │     :80      │                 │
       └──────┬───────┘                 │
              │                         │
              │ backend:8080            │
              ↓                         │
       ┌──────────────┐                 │
       │   backend    │                 │
       │ Spring Boot  │                 │
       │    :8080     │                 │
       └──────┬───────┘                 │
              │                         │
              │ mysql:3306              │
              ↓                         │
       ┌──────────────┐                 │
       │    mysql     │                 │
       │    :3306     │                 │
       └──────────────┘                 │
              │
              ↓
        mysql-data volume


Full Multi-Container Architecture




              Jib approach
                  ❌
              temporarily
               disabled
                  │
                  ↓
         mvn clean package
                  │
                  ↓
              target/*.jar
                  │
                  ↓
          Docker Compose
            /          \
           ↓            ↓
      Backend          Frontend
      Dockerfile       Dockerfile
           \            /
            ↓          ↓
             Docker Images
                  ↓
              Docker Hub
That way you can clearly learn Dockerfile + Compose + dynamic tagging + Docker Hub push without Jib unexpectedly executing in the middle.



                  compose.yaml
                       │
       ┌───────────────┼────────────────┐
       │               │                │
     MySQL           Backend          Frontend
       │               │                │
mysql:8.4        Dockerfile        Dockerfile
│               │                │
│               └───────┬────────┘
│                       │
│                    Build
│                       ↓
│                 Docker Images
│                       ↓
│                   Docker Hub
│
Official image



                    ┌──────────────────────┐
                    │       pom.xml        │
                    │                      │
                    │       Jib            │
                    │   <executions>       │
                    │      DISABLED        │
                    │   </executions>      │
                    └──────────┬───────────┘
                               │
                               X
                               │
                         NOT USED
                               │


     ┌──────────────────────────────────────────────────┐
     │             Docker Compose Approach              │
     │                                                  │
     │  compose.yaml                                   │
     │       │                                          │
     │       ├── Backend Dockerfile                     │
     │       │       ↓                                  │
     │       │   Backend Image                          │
     │       │                                          │
     │       └── Frontend Dockerfile                    │
     │               ↓                                  │
     │           Frontend Image                         │
     │                                                  │
     │              ↓                                   │
     │     docker compose build                         │
     │              ↓                                   │
     │        Local Docker Images                       │
     │              ↓                                   │
     │     docker compose push                          │
     │              ↓                                   │
     │          Docker Hub                              │
     └──────────────────────────────────────────────────┘




Jib approach:
Maven → Jib → Docker Hub

Compose approach:
Dockerfile → Docker Compose → Local Image → docker compose push → Docker Hub

Jenkins learning path

                    GitHub
                       │
                       │ git push
                       ▼
              ┌─────────────────┐
              │     Jenkins     │
              │   Controller    │
              └────────┬────────┘
                       │
                       │ Pipeline
                       ▼
              ┌─────────────────┐
              │ Jenkins Agent   │
              │                 │
              │ Git             │
              │ Java 21         │
              │ Maven           │
              │ Docker          │
              └────────┬────────┘
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Maven Build          Docker Build
             │                   │
             ▼                   ▼
          Tests             Docker Image
                                 │
                                 ▼
                            Docker Hub



Developer
│
│ git push
▼
GitHub
│
│ webhook
▼
Jenkins
│
├── Checkout
├── Compile
├── Unit Test
├── Package
├── Docker Build
├── Trivy Scan
├── Docker Push
└── Deploy
│
▼
AWS / ECS

That will connect nicely with what you've already been doing with Docker, Docker Compose, Docker Hub and Trivy.
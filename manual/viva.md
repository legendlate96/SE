# Software Engineering & DevOps Lab — Complete Viva Revision Guide

> **How to use this guide:** Designed for a **15-minute pre-viva sprint** with deep architectural explanations so you can handle both basic definitions and tough *"Why?" / "How does it work under the hood?"* follow-up questions.

---

## Quick Navigation
1. [SRS, Design, Testing & Risk Foundations (Week 1 + Units 3–5)](#1-srs-design-testing--risk-foundations)
2. [Git & Collaborative Coding (Weeks 2–3)](#2-git--collaborative-coding)
3. [Maven Build Automation (Week 4 + Unit 3)](#3-maven-build-automation)
4. [Docker & Containerization Architecture (Weeks 5–6 + Unit 4)](#4-docker--containerization-architecture)
5. [Docker Compose — Multi-Container Apps (Week 7 + Unit 4)](#5-docker-compose--multi-container-apps)
6. [Jenkins CI/CD, Pipelines, Webhooks & Ngrok (Weeks 8, 9, 11 + Unit 3)](#6-jenkins-cicd-pipelines-webhooks--ngrok)
7. [Kubernetes & Minikube Architecture (Week 10 + Unit 4)](#7-kubernetes--minikube-architecture)
8. [Continuous Monitoring & Nagios (Week 10 + Unit 4)](#8-continuous-monitoring--nagios)
9. [Cloud Computing & AWS EC2 Deployment (Weeks 10, 12 + Unit 5)](#9-cloud-computing--aws-ec2-deployment)
10. [20 Rapid-Fire Examiner Traps (Cheat Sheet)](#10-20-rapid-fire-examiner-traps)

---

## 1. SRS, Design, Testing & Risk Foundations

### 1.1 SRS (Software Requirements Specification)
An SRS is the formal contract between customer and developer defining **what** the system must do (without prescribing implementation code).
- **Functional Requirements (FR):** *What* the system does (core features/behaviors). *Ex: Detect ransomware, classify family, output confidence score.*
- **Non-Functional Requirements (NFR):** *How well* the system performs (quality attributes & constraints). *Ex: Low background CPU/RAM overhead, real-time detection latency, security, scalability.*

### 1.2 Design Engineering & Quality (Unit 3)
Translates the **Analysis Model** (*what*) into the **Design Model** (*how*) across 4 layers: **Data/Class -> Architecture -> Interface -> Component**.
- **FURPS Quality Model:** **F**unctionality, **U**sability, **R**eliability, **P**erformance, **S**upportability.
- **Core Design Principles:**
  - **Abstraction:** Hide complexity; focus on essential behavior (Procedural, Data, Control).
  - **Modularity & Information Hiding:** Split into independent modules; keep internal details private so changes don't ripple across the codebase.
  - **Architectural Style vs. Pattern:** A **Style** is the broad macro-layout (*Layered, Data-Centered, Pipes & Filters*). A **Pattern** is a reusable solution to a recurring design problem (*MVC, Microservices, Event-Driven*).
- **UML Quick Map:**
  - **Structural (Static):** Class, Object, Component, Deployment.
  - **Behavioral (Dynamic):** Use Case, Sequence (time-ordered messages), Activity (flowchart), State-Chart.
  - **High-Yield UML Traps:**
    - `<<include>>` = **Mandatory** sub-step (e.g., *Withdraw* always includes *Authenticate*).
    - `<<extend>>` = **Optional/Conditional** behavior (e.g., *Apply Discount* or *Invalid PIN*).
    - **Aggregation (hollow diamond):** Weak "has-a"; part survives without whole (*Classroom & Students*).
    - **Composition (filled diamond):** Strong "owns-a"; part dies with whole (*House & Rooms*).

### 1.3 Testing Strategies & Debugging (Unit 4)
- **Testing vs. Debugging:** Testing *finds* defects (QA/testers: "Does it work?"); Debugging *locates root cause & fixes* code (developers: "Why did it fail?").
- **Verification vs. Validation:**
  - **Verification:** *"Are we building the product **right**?"* (Static checks: design/code reviews against SRS).
  - **Validation:** *"Are we building the **right** product?"* (Dynamic execution: does it satisfy user needs?).
- **Black-Box vs. White-Box:** Black-Box tests inputs/outputs from specs without reading code; White-Box inspects internal paths, loops, and branches.
- **Testing Spiral:** **Unit** (isolated module) -> **Integration** (module interactions) -> **Validation** (Alpha = internal controlled env; Beta = real users) -> **System** (full app: performance, security, compatibility).
- **Drivers vs. Stubs:**
  - **Driver (Bottom-Up):** Dummy *caller* simulating a higher-level module to invoke a lower module.
  - **Stub (Top-Down):** Dummy *callee* returning fake responses for an unfinished lower module.

### 1.4 Risk Management (Unit 5)
- **Reactive ("Firefighting"):** Respond *after* a problem occurs.
- **Proactive ("Fire Prevention"):** Identify risks early, estimate **Probability** & **Impact** (Risk Table), and prepare plans before issues hit.
- **RMMM Plan:**
  - **Mitigation:** Steps to *prevent* or reduce risk likelihood/impact.
  - **Monitoring:** Continuously tracking risk indicators during the project.
  - **Management (Contingency):** Backup plan executed if the risk actually materializes.

---

## 2. Git & Collaborative Coding

### 2.1 Git's 4-Zone Architecture
Git is a **Distributed Version Control System (DVCS)**—every developer has a full local copy of the repository and commit history (unlike centralized VCS).

```text
+-------------------+   git add    +------------------+  git commit  +------------------+   git push   +------------------+
| Working Directory | -----------> |   Staging Area   | -----------> | Local Repository | -----------> | Remote Repo      |
| (Edited files)    | <----------- |     (Index)      | <----------- |  (.git objects)  | <----------- | (GitHub)         |
+-------------------+  git restore +------------------+  git reset   +------------------+  git fetch   +------------------+
       ^                                                                                         |
       +--------------------------------------- git pull (fetch + merge) ------------------------+
```

### 2.2 High-Yield Viva Comparisons

| Pair | Key Difference (Say This in Viva!) |
| :--- | :--- |
| **`git fetch` vs. `git pull`** | `fetch` downloads remote commits/refs into `origin/<branch>` **without touching** your working code. `pull` = `fetch` + `merge` (automatically integrates changes, which may cause conflicts). |
| **`git merge` vs. `git rebase`** | `merge` preserves full branching history by creating a new **merge commit**. `rebase` replays your commits on top of the target branch for a **clean linear history** (never rebase shared public branches—it rewrites commit hashes!). |
| **`git reset` vs. `git revert`** | `reset` moves the branch pointer backward, **rewriting history** (`--soft`: keeps staged; `--mixed` default: keeps unstaged; `--hard`: wipes working tree). `revert` creates a **new inverse commit**, keeping history intact (safe for pushed/shared branches). |
| **`git branch -d` vs. `-D`** | `-d` safely deletes only if already merged. `-D` force-deletes unmerged branches. |
| **Clone vs. Fork** | `Clone` creates a **local** copy on your machine. `Fork` creates a **remote** copy under your GitHub account so you can contribute to someone else's repo via Pull Request. |
| **`git apply` vs. `git am`** | Both apply `.patch` files (created by `git format-patch`). `git apply` only modifies working files (no commit). `git am` applies the patch **and preserves commit metadata** (author, message) as a new commit. |

### 2.3 Deep Mechanisms & Scenario Commands
- **Merge Conflicts:** Happen when two branches modify the **same lines** of the same file (or one edits and one deletes). Git pauses and inserts conflict markers (`<<<<<<< HEAD`, `=======`, `>>>>>>> branch`). Fix: edit file -> remove markers -> `git add <file>` -> `git commit` (or `git merge --abort` to cancel).
- **Non-Fast-Forward Push Error:** Remote has commits your local branch lacks. Fix: `git pull origin main` (resolve conflicts if any) -> `git push`.
- **`git stash`:** Temporarily shelves uncommitted changes onto a stack so you can switch branches cleanly (`git stash apply` keeps it on stack; `git stash pop` applies & removes it).
- **`git reflog`:** Local safety net recording every movement of `HEAD` (even deleted branches or hard resets). Recover a deleted branch with `git checkout -b <branch> <commit-hash>`.
- **`.gitignore`:** Prevents untracked files (`*.log`, `node_modules/`, `.env`) from being staged. *(Note: does NOT untrack files already committed—use `git rm --cached <file>` first).*
- **Removing Sensitive Data from History:** Simply deleting a file in a new commit leaves it visible in old commits. Use `git filter-branch` (or modern `git filter-repo`) + force push to purge it across all history.
- **Diagnostics:** `git blame <file>` shows who last edited each line; `git fetch --prune` / `git remote prune origin` cleans up stale references to deleted remote branches.

---

## 3. Maven Build Automation

### 3.1 Why Maven & Core Concepts
Maven is a Java **build automation and dependency management** tool built on **Convention over Configuration** (standardized folder layout means less boilerplate setup).
- **`pom.xml` (Project Object Model):** The XML recipe defining project metadata, dependencies, plugins, and build settings.
- **GAV Coordinates:** Uniquely identify any artifact in Maven:
  - `groupId` (organization/package, e.g., `com.example`)
  - `artifactId` (project name, e.g., `my-web-app`)
  - `version` (e.g., `1.0-SNAPSHOT` — `SNAPSHOT` means active development build, checked for updates).
- **Standard Directory Structure:**
  - `src/main/java` (app code) | `src/main/resources` (config files like `config.properties`, copied to `target/classes`)
  - `src/test/java` (JUnit tests) | `src/main/webapp/WEB-INF/web.xml` (web apps)
  - `target/` (generated `.class`, `surefire-reports`, `.jar`/`.war`).

### 3.2 Build Lifecycles & Phases
Maven has **3 built-in lifecycles**: `clean` (deletes `target/`), `default` (builds/deploys project), `site` (generates project documentation).

```text
Default Lifecycle Order (running any phase automatically runs all preceding phases):
validate ---> compile ---> test ---> package ---> verify ---> install ---> deploy
                                      |                         |            |
                              Creates JAR/WAR           Installs to     Uploads to
                                in target/             local ~/.m2     remote repo
```

### 3.3 Repository Lookup & Dependency Management
- **Lookup Order:** **Local Repository** (`~/.m2/repository`) -> **Maven Central** (online public repo) -> **Custom Remote Repository** (company Nexus/Artifactory).
- **Transitive Dependency Mediation:** If Library A needs Gson `2.8` and Library B needs Gson `2.10`, Maven resolves conflicts using the **"Nearest Definition"** rule (shortest path in dependency tree; if equal depth, first declared in `pom.xml` wins). Inspect with `mvn dependency:tree`.
- **Dependency Scopes (Frequent Viva Question!):**
  - `compile` (default): Available everywhere & packaged inside final artifact.
  - `test`: Only during test compile/execution (e.g., JUnit)—not bundled in final JAR/WAR.
  - `provided`: Needed at compile time, **NOT bundled** in WAR because the runtime container (e.g., Apache Tomcat) already provides it (e.g., `javax.servlet-api`—bundling it causes classloader conflicts!).

### 3.4 Key Plugins, Packaging & Multi-Module Projects
- **JAR vs. WAR:**
  - **JAR (Java ARchive):** Standalone Java app/library executed directly by JVM (`java -jar app.jar`). To make it executable, `maven-jar-plugin` writes `<mainClass>` into `META-INF/MANIFEST.MF`.
  - **WAR (Web Application ARchive):** Web app (`<packaging>war</packaging>`) containing JSPs, servlets, `WEB-INF/web.xml`; deployed into a Servlet Container like **Tomcat** (`webapps/` folder auto-extracts and deploys it).
- **Core Plugins:** `maven-compiler-plugin` (sets Java `<source>`/`<target>` to prevent *"Unsupported class version error"*), `maven-surefire-plugin` (runs JUnit tests & writes `target/surefire-reports/`), `maven-jar-plugin`, `maven-war-plugin`.
- **Multi-Module / Parent POM:** Root `parent/pom.xml` uses `<packaging>pom</packaging>` and lists `<modules><module>core</module><module>web</module></modules>`. Centralizes shared dependency versions and builds modules in dependency order via the Maven Reactor.
- **Useful Flags:** `mvn clean install -DskipTests` (skip test execution), `mvn clean compile -U` (force update snapshots/plugins), `mvn -Dtest=AppTest test` (run one test class).

---

## 4. Docker & Containerization Architecture

### 4.1 Containerization vs. Virtual Machines (VMs)
Containers solve *"it works on my machine"* by bundling app code, runtime, and libraries into a portable unit.

```text
        VIRTUAL MACHINES (Hypervisor)                 DOCKER CONTAINERS (Container Engine)
+-----------+  +-----------+  +-----------+        +-----------+  +-----------+  +-----------+
|   App 1   |  |   App 2   |  |   App 3   |        |   App 1   |  |   App 2   |  |   App 3   |
| Bins/Libs |  | Bins/Libs |  | Bins/Libs |        | Bins/Libs |  | Bins/Libs |  | Bins/Libs |
| Guest OS  |  | Guest OS  |  | Guest OS  |        +-----------+--+-----------+--+-----------+
+-----------+--+-----------+--+-----------+        |     Docker Engine (Container Runtime)   |
|        Hypervisor (VirtualBox/VMware)   |        +-----------------------------------------+
+-----------------------------------------+        |           Host OS (Shared Kernel)       |
|              Host OS / Hardware         |        +-----------------------------------------+
```

| Aspect | Virtual Machines (VMs) | Docker Containers |
| :--- | :--- | :--- |
| **OS Kernel** | Each VM runs a full separate **Guest OS** | **Shares the Host OS kernel** (isolated via Linux *Namespaces* & *cgroups*) |
| **Size & Boot** | Gigabytes; boots in minutes | Megabytes; starts in milliseconds/seconds |
| **Isolation** | Hardware-level (stronger) | Process-level (lightweight, high density) |

> **Deep Viva Bonus:** How does Docker isolate containers if they share the host kernel? Using **Linux Namespaces** (isolates PIDs, filesystem, network, users so each container thinks it's its own machine) and **cgroups (Control Groups)** (limits CPU/RAM per container, e.g., `--memory="512m"`).

### 4.2 Docker Client-Server Architecture

```text
+-------------------+   REST API    +-----------------------------------+   Pull / Push   +------------------+
|   Docker Client   | ------------> |            Docker Host            | <-------------> | Docker Registry  |
|  (docker CLI:     | (Unix socket  |  +-----------------------------+  |                 |  (Docker Hub)    |
|  build/pull/run)  |  or TCP)      |  |    Docker Daemon (dockerd)  |  |                 |  Stores versioned|
+-------------------+               |  +--------------+--------------+  |                 |  Docker Images   |
                                    |        |                |         |                 +------------------+
                                    |  +-----v-----+    +-----v------+  |
                                    |  |  Images   |--->| Containers |  |
                                    |  |(Read-Only)|    |(Running +  |  |
                                    |  |  Layers)  |    | R/W Layer) |  |
                                    |  +-----------+    +------------+  |
                                    +-----------------------------------+
```

- **Docker Image:** Immutable, **read-only** blueprint built in stacked layers (Union File System). Shared base layers are cached—making rebuilds and pulls fast.
- **Docker Container:** A runnable instance of an image. Docker adds a thin **writable top layer** (Copy-on-Write). When the container is deleted (`docker rm`), that writable layer is lost unless you use **Volumes**.
- **Docker Registry:** Central storage for distributing images (default: **Docker Hub**).

### 4.3 Dockerfile Instructions & Traps
| Instruction | Role & Viva Nuance |
| :--- | :--- |
| `FROM` | Sets base image (must be first instruction, e.g., `python:3.10-slim`, `nginx:alpine`). |
| `WORKDIR` | Sets working directory inside container for subsequent commands. |
| `COPY` vs `ADD` | `COPY` strictly copies local files/folders (preferred). `ADD` also auto-extracts `.tar` archives and supports remote URLs. |
| `RUN` | Executes at **image build time** (creates a new image layer, e.g., `RUN pip install flask`). |
| `CMD` | Default command at **container runtime**. Easily overridden by arguments passed to `docker run`. |
| `ENTRYPOINT` | Configures container as a fixed executable; `docker run` arguments are *appended* to it rather than replacing it. |
| `EXPOSE` | **Documentation only!** Declares which port the app listens on internally—does *not* publish it to the host. You still need `-p <host>:<container>` at runtime. |

### 4.4 Essential Operations & Storage
- **Port Mapping:** `-p 8080:5000` maps Host port `8080` -> Container port `5000`. Two containers *cannot* bind to the same host port (`8080`), even if they use the same internal port internally.
- **Persistent Storage:** Containers are **ephemeral**. Use **Docker Volumes** (managed by Docker, ideal for databases) or **Bind Mounts** (maps host path directly, ideal for live dev code) to persist data across restarts/removals.
- **`docker commit` vs. `Dockerfile`:** `docker commit` snapshots a running container into an image (quick & ad-hoc, not reproducible). `Dockerfile` is declarative, version-controllable, and industry standard.
- **Troubleshooting Commands:** `docker ps -a` (all containers incl. stopped), `docker logs -f <id>` (crash logs), `docker exec -it <id> /bin/bash` (interactive shell), `docker stats` (live CPU/RAM), `docker save -o img.tar img` (offline export), `--restart=always` (auto-restart on crash).

---

## 5. Docker Compose — Multi-Container Apps

### 5.1 What It Solves
While `Dockerfile` builds a **single image**, **Docker Compose** defines and orchestrates **multiple interdependent containers** (e.g., Web + API + Postgres + Redis) on a **single host** using one declarative YAML file (`docker-compose.yml`).

### 5.2 Anatomy of `docker-compose.yml`
- **`services`:** Defines each container (`image` or `build: .`, `ports`, `environment`, `volumes`, `networks`, `depends_on`).
- **`networks`:** Compose automatically creates a default isolated **bridge network** for the project. Containers reach each other via **built-in DNS using their service names** (e.g., WordPress connects to `db:3306`—never hardcode container IPs!).
- **`volumes`:** Named persistent volumes (e.g., `db-data:/var/lib/postgresql/data`) ensure DB data survives `docker compose down`.

### 5.3 Critical Viva Traps
- **`depends_on` Trap:** `depends_on: [db]` guarantees the `db` container **starts** before `web`, **NOT** that Postgres is finished initializing and ready to accept queries! (Production fix: add `healthcheck` + `condition: service_healthy`).
- **One Process Per Container Rule:** Don't stuff Node.js + Python cron jobs in one container—split them into separate services in Compose for independent scaling, clean logging, and fault isolation.
- **Command Distinctions:**
  - `docker compose up -d`: Creates network/volumes and starts all containers in detached mode.
  - `docker compose up --build`: Rebuilds images from updated Dockerfiles/code before starting.
  - `docker compose up -d --no-deps --build frontend`: Rebuilds/restarts **only** one service without touching DB/backend.
  - `docker compose down`: Stops & **removes** containers and default networks (**keeps** named volumes; add `-v` to wipe volumes too).
  - `docker compose up --scale web=3 -d`: Runs 3 replicas of a service (avoid hardcoded single host ports when scaling!).

---

## 6. Jenkins CI/CD, Pipelines, Webhooks & Ngrok

### 6.1 CI vs. Continuous Delivery vs. Continuous Deployment
| Stage | Meaning | Human Gate? |
| :--- | :--- | :--- |
| **Continuous Integration (CI)** | Developers merge code frequently; Jenkins auto-triggers **Build + Unit Tests** to catch bugs early. | No (fully automated) |
| **Continuous Delivery (CD)** | Every passing build is automatically packaged and **staged/ready** for production release. | **Yes** (manual approval to deploy to prod) |
| **Continuous Deployment (CD)** | Every passing build is **automatically deployed directly to production**. | **No** (zero manual intervention) |

### 6.2 Jenkins Distributed Architecture (Controller/Master – Agent/Slave)

```text
+-----------------------------------+           SSH / Inbound TCP           +---------------------------+
|     Jenkins Controller (Master)   | ------------------------------------> |  Agent / Slave (Linux)    |
|  - Web UI (Default Port: 8080)    |                                       |  (Executes builds/tests)  |
|  - Schedules jobs & plugins       | ------------------------------------> +---------------------------+
|  - Monitors agents & stores logs  |                                       |  Agent / Slave (Windows)  |
+-----------------------------------+                                       +---------------------------+
```
- **Why Master-Agent?** Offloads heavy compilation/testing from the Master, enables parallel builds, and lets you build OS-specific targets (e.g., Windows `.bat` vs Linux `.sh` agents).

### 6.3 Freestyle Jobs vs. Pipelines (Declarative vs. Scripted)
- **Freestyle Project:** GUI-configured jobs. Good for simple tasks; multi-stage pipelines require chaining separate jobs (`Build` -> `Test` -> `Deploy`) via *Upstream/Downstream triggers*, *Archive Artifacts*, and *Copy Artifacts* plugins.
- **Pipeline as Code (`Jenkinsfile`):** Entire workflow defined in a Groovy-based script stored in Git (versioned, auditable, portable).
  - **Declarative Pipeline (Preferred):** Rigid, readable structure starting with `pipeline { agent any ... stages { stage(...) { steps { ... } } } }`. Supports built-in `tools`, `triggers`, `post { success/failure }`.
  - **Scripted Pipeline:** Starts with `node { ... }`. Full Groovy imperative flexibility (`if/else`, `try/catch`, loops) for complex custom logic.
- **Execution Rules:**
  - `tools { maven 'MAVEN_HOME' }` injects the globally configured Maven binary into `PATH`.
  - **Fail-Fast:** If any stage fails (e.g., `mvn test`), Jenkins immediately aborts and skips all downstream stages unless wrapped in `catchError`/`try-catch`.
  - **OS Portability:** Windows agents run `bat '...'`; Linux/macOS agents run `sh '...'`.

### 6.4 Build Triggers: Webhook vs. Poll SCM vs. Cron
| Trigger | Mechanism | Pros / Cons |
| :--- | :--- | :--- |
| **GitHub Webhook** | **Push-based (Event-driven):** GitHub sends an HTTP `POST` (`application/json`) to `http://<jenkins>/github-webhook/` immediately on `git push`. | **Best:** Instant trigger, zero wasted CPU/network. |
| **Poll SCM** | **Pull-based:** Jenkins periodically asks GitHub *"Any new commits?"* on a schedule. | Wastes resources polling idle repos; introduces delay. |
| **Build Periodically (Cron)** | Runs unconditionally on a timer (e.g., `0 18 * * *` = daily at 6 PM). | Good for nightly regression suites. |

- **Why Ngrok is Needed:** GitHub servers on the public internet **cannot reach** `http://localhost:8080` on your laptop behind a router/NAT. **Ngrok** (`ngrok http 8080`) opens a secure reverse tunnel providing a public HTTPS URL that forwards GitHub's webhook payload to your local Jenkins.
- **Email Notifications:** Requires **Email Extension Plugin** + SMTP config (`smtp.gmail.com`, SSL enabled, Port `465`). **Why Gmail App Password?** Google blocks standard account passwords for third-party SMTP logins; enabling 2-Step Verification and generating a 16-char **App Password** authenticates Jenkins securely without exposing your main password.

---

## 7. Kubernetes & Minikube Architecture

### 7.1 Why Kubernetes (K8s)?
Docker alone only creates/runs containers on one host—if a container crashes, traffic spikes, or the host dies, Docker won't automatically recover it across servers. **Kubernetes** automates **orchestration**: self-healing, auto-scaling, service discovery, load balancing, and zero-downtime rolling updates/rollbacks.

| Feature | Docker | Docker Swarm | Kubernetes (K8s) |
| :--- | :--- | :--- | :--- |
| **Scope** | Single container runtime | Lightweight built-in clustering | Enterprise-grade container orchestration |
| **Self-Healing & Scaling** | Basic local `--restart` | Basic scaling & recovery | Advanced HPA auto-scaling, health probes, self-healing |
| **Rollouts** | Manual | Basic | Built-in Rolling Updates & Instant Rollbacks |

### 7.2 Kubernetes Master-Worker Architecture (MUST KNOW FOR VIVA!)

```text
+--------------------------------------------------+        +----------------------------------------+
|            CONTROL PLANE (Master Node)           |        |             WORKER NODE(S)             |
|                                                  |        |                                        |
|  kubectl ---> [ kube-apiserver ] <---------------+--------+---> [ kubelet ] (Node Agent)           |
|                 /      |      \                  |        |          |                             |
|                v       v       v                 |        |   +------v--------------------------+  |
|          [ etcd ] [Scheduler] [Controller Mgr]   |        |   | Container Runtime (containerd)  |  |
|       (Cluster DB) (Picks Node) (Desired=Actual) |        |   |  +-----------+   +-----------+  |  |
+--------------------------------------------------+        |   |  |  Pod 1    |   |  Pod 2    |  |  |
                                                            |   |  |(Container)|   |(Container)|  |  |
                                                            |   |  +-----------+   +-----------+  |  |
                                                            |   +---------------------------------+  |
                                                            |   [ kube-proxy ] <--- External Traffic |
                                                            +----------------------------------------+
```

- **Control Plane (Brain):**
  1. **`kube-apiserver`:** The single front door (REST API) for `kubectl` and all internal components. Validates & updates state in `etcd`.
  2. **`etcd`:** Distributed, consistent **key-value store** holding the entire cluster's configuration and current state (single source of truth).
  3. **`kube-scheduler`:** Watches for newly created unassigned Pods and assigns them to the best Worker Node based on CPU/RAM availability and constraints.
  4. **`kube-controller-manager`:** Runs reconciliation loops (Node, Replication, Deployment controllers) to ensure **Actual State == Desired State** (e.g., if 1 of 4 replicas dies, it spawns a replacement).
- **Worker Nodes (Muscle):**
  1. **`kubelet`:** Agent on every worker node that receives Pod specs from API Server and ensures containers are running & healthy via the runtime.
  2. **`kube-proxy`:** Maintains OS network rules (iptables/IPVS) to route traffic across Services and Pods.
  3. **Container Runtime:** Runs the actual containers (`containerd`, CRI-O, or Docker shim).

### 7.3 Core Objects & Minikube
- **Pod:** Smallest deployable unit in K8s (wraps 1 or more tightly coupled containers sharing one IP address, network namespace, and storage volumes). Pods are **mortal/ephemeral**—if recreated, they get a new IP.
- **Deployment:** Declarative manager for Pods/ReplicaSets. Handles scaling (`kubectl scale --replicas=4`), rolling updates (`kubectl set image`), and rollbacks (`kubectl rollout undo deployment/<name>`).
- **Service:** Provides a **stable IP and DNS name** in front of ephemeral Pods and load-balances traffic across them using label selectors:
  - `ClusterIP` (default): Internal cluster access only.
  - `NodePort`: Exposes app on a static port (`30000–32767`) on every node's IP (used in your lab!).
  - `LoadBalancer`: Provisions a cloud load balancer (AWS ELB) for external traffic.
- **Minikube:** Spins up a lightweight **single-node Kubernetes cluster** inside a local VM/Docker container where **one node acts as both Master and Worker** for local testing. Access apps via `kubectl port-forward svc/mynginx 8081:80` or `minikube tunnel` / `minikube service mynginx --url`.

### 7.4 K8s Troubleshooting Playbook
- **Pod stuck in `Pending`:** Scheduler cannot place it—usually insufficient CPU/RAM, node taint, or unbound storage. Check with `kubectl describe pod <pod>` and `kubectl get nodes`.
- **Pod in `CrashLoopBackOff`:** Container starts and immediately crashes repeatedly. Check previous crash logs with `kubectl logs <pod> --previous` and events via `kubectl describe pod <pod>`.
- **Probes:** **Liveness Probe** checks if container is alive (restarts if failed); **Readiness Probe** checks if container is ready for traffic (removes from Service endpoints if failed).

---

## 8. Continuous Monitoring & Nagios

### 8.1 Continuous Monitoring in DevOps
Continuous Monitoring sits in the **Operate -> Monitor** phase of DevOps. It continuously tracks infrastructure, application health, logs, and security metrics to detect anomalies **proactively** before users face downtime.
- **Tool Categories:** Infrastructure (**Nagios**), Logs (**ELK Stack**), APM (**New Relic**), Security/SIEM (**Splunk**).

### 8.2 Nagios Architecture

```text
+-----------------------------------------+            +--------------------+
|              Nagios Server              |            |   Remote Hosts /   |
|  +-----------------------------------+  |  Active    |   Services         |
|  | Nagios Core (Process Scheduler)   |  |  Checks    | (CPU, RAM, HTTP,   |
|  +-----------------+-----------------+  | ---------> |  Disk, Network)    |
|                    | invokes            |            +---------+----------+
|  +-----------------v-----------------+  |                      |
|  |        Plugins (Workers)          | <-----------------------+ (Returns status)
|  +-----------------+-----------------+  |
+--------------------|--------------------+
                     +-------------> [ Web UI Dashboard (Port 8888:80) ] + [ Email/SMS Alerts ]
```

- **Nagios Core (Brain):** Open-source central engine/scheduler that decides *what* to check and *when*, evaluates thresholds, updates state, and triggers alerts. (**Nagios XI** is the paid enterprise version with advanced UI/reporting).
- **Plugins (Workers):** Modular scripts/binaries that actually execute checks (ping, HTTP, disk, CPU load) and return standard exit codes (`0=OK`, `1=WARNING`, `2=CRITICAL`, `3=UNKNOWN`).
- **Hosts vs. Services:** A **Host** is a physical/virtual server (`UP`/`DOWN`); **Services** are metrics/daemons running on that host (CPU, RAM, Apache, SSH).
- **Lab Setup & Debugging:** Run via Docker (`docker run --name nagiosdemo -p 8888:80 jasonrivers/nagios:latest`, login `nagiosadmin`/`nagios`).
  - *"Unable to connect to Nagios process"*: Container/daemon crashed or stopped -> check `docker ps -a`, inspect `docker logs -f nagiosdemo`, and `docker restart nagiosdemo`.
  - *New host not showing*: Syntax error in Nagios `.cfg` file or Nagios service wasn't reloaded after config edit.

---

## 9. Cloud Computing & AWS EC2 Deployment

### 9.1 Cloud Computing Essentials
On-demand delivery of IT resources over the internet with **Availability**, **Rapid Elasticity/Scalability**, **Resource Pooling**, and **Pay-as-you-go** pricing.

| Model Category | Types & Clear Distinction |
| :--- | :--- |
| **Deployment Models** | **Public** (shared multi-tenant cloud: AWS, Azure, GCP) \| **Private** (dedicated to one org for strict security/compliance) \| **Hybrid** (combines private on-prem + public cloud bursting). |
| **Service Models** *(Control decreases ->)* | **IaaS** (Virtual hardware: VMs, storage, networks — *AWS EC2*) -> **PaaS** (Managed runtime/platform for devs — *Elastic Beanstalk, Heroku*) -> **SaaS** (Ready-to-use end-user apps — *Gmail, Google Docs*). |

### 9.2 Key AWS Services (1-Line Definitions)
- **EC2 (Elastic Compute Cloud):** Resizable virtual servers (VMs) in the cloud (IaaS).
- **S3 (Simple Storage Service):** Infinitely scalable **object storage** (buckets) for files, backups, static sites.
- **VPC (Virtual Private Cloud):** Your isolated private virtual network in AWS (subnets, route tables, gateways).
- **IAM (Identity & Access Management):** Manages users, roles, groups, and least-privilege permissions.
- **RDS:** Managed relational databases (MySQL, Postgres).
- **CloudWatch:** Native monitoring, logs, metrics, and alarms for AWS resources.
- **Lambda:** Serverless compute—runs event-driven code without managing servers.
- **CloudFront:** Global CDN (Content Delivery Network) caching content at edge locations for low latency.

### 9.3 EC2 Deep Dive & Cloud Web Deployment (Week 12)
When launching an Ubuntu EC2 instance (`t2.micro`: 1 vCPU, 1 GB RAM, 8 GB EBS root volume):
1. **AMI (Amazon Machine Image):** OS + software template used to boot the EC2 instance (e.g., Ubuntu Server LTS).
2. **Key Pair (`.pem`):** Public key lives on EC2; private `.pem` key stays on your machine for passwordless asymmetric **SSH authentication** (`ssh -i key.pem ubuntu@<public-ip>`).
3. **Security Group (CRITICAL VIVA TOPIC!):** Acts as a **stateful virtual firewall** controlling **Inbound** and **Outbound** traffic by protocol/port/IP.
   - *Common Ports:* `22` (SSH), `80` (HTTP), `443` (HTTPS), `8080`/`6060` (Tomcat/Docker web apps).
   - *Classic Trap:* If `curl localhost:8080` works inside EC2 but the browser times out externally -> **Port `8080` is blocked in the Security Group Inbound Rules!**
4. **Public IP vs. Elastic IP:** A default EC2 Public IP is **dynamic** (changes whenever the instance is stopped and restarted). Associate an **Elastic IP (EIP)** for a permanent static public IPv4 address.
5. **EC2 User Data:** Shell script (`#!/bin/bash`) passed at launch to automate software installation and service startup (e.g., installing & enabling `httpd`/Docker on boot).
6. **NAT Gateway:** Allows an EC2 instance inside a **private subnet** to initiate **outbound** internet connections (e.g., downloading Maven/apt dependencies) while blocking unsolicited **inbound** internet traffic.
7. **End-to-End Cloud Deployment Flow:**
   `Push code to GitHub` -> `Launch EC2 + open ports in Security Group` -> `SSH via .pem` -> `Install git & docker.io` -> `git clone` -> `Write Dockerfile` -> `docker build -t app .` -> `docker run -d -p 80:8080 app` -> `Access via http://<EC2-Public-IP>`.

---

## 10. 20 Rapid-Fire Examiner Traps

1. **Docker vs. Jenkins — aren't both DevOps tools?**
   *Jenkins automates the CI/CD pipeline (build/test/deploy); Docker packages apps and dependencies into portable containers so they run identically everywhere.*
2. **Why use `scope: provided` for `javax.servlet-api` in Maven?**
   *It's needed to compile the code, but Tomcat already ships with it at runtime. Bundling it in the WAR bloats the package and causes runtime classloader conflicts.*
3. **What is the difference between `mvn package` and `mvn install`?**
   *`package` builds the JAR/WAR into `target/`. `install` runs `package` AND copies the artifact into your local `~/.m2/repository` so other local projects can depend on it.*
4. **Why did `docker run` fail with "port already allocated" when running two web containers?**
   *Both tried to bind to the same **host port** (e.g., `8080`). Map them to different host ports (`-p 8081:8080` and `-p 8082:8080`).*
5. **If I delete a running database container, what happens to the data?**
   *It is permanently lost unless mounted to a **Docker Volume** or **Bind Mount**, because the container's writable layer is ephemeral.*
6. **Why does `EXPOSE 80` in a Dockerfile not let me open `localhost:80`?**
   *`EXPOSE` is only metadata/documentation. You must publish the port using `-p 80:80` during `docker run`.*
7. **How do two containers in Docker Compose talk to each other without knowing IP addresses?**
   *Docker Compose creates a shared bridge network with an internal DNS resolver, so containers address each other by **service name** (e.g., `db:5432`).*
8. **Why is `git revert` safer than `git reset --hard` on `main`?**
   *`reset --hard` rewrites commit history and breaks collaborators' local repos. `revert` adds a new undo commit, preserving shared history.*
9. **You accidentally committed `secrets.txt` and deleted it in the next commit. Is it safe?**
   *No! It still exists in Git's object history. You must rewrite history (`git filter-branch`/`git filter-repo`), force-push, and rotate the compromised keys.*
10. **Why do we use Ngrok with Jenkins and GitHub Webhooks?**
    *Local Jenkins (`localhost:8080`) isn't publicly reachable. Ngrok creates a secure public HTTPS tunnel forwarding GitHub's webhook HTTP POSTs to local Jenkins.*
11. **What's the difference between Declarative and Scripted Jenkins Pipelines?**
    *Declarative (`pipeline {}`) is structured, readable, and validated before running. Scripted (`node {}`) uses raw Groovy for complex imperative logic.*
12. **If `mvn test` fails in a Jenkins pipeline, does the `Deploy` stage run?**
    *No. Jenkins fails fast and halts subsequent stages by default unless error handling (`catchError`/`try-catch`) explicitly allows continuation.*
13. **Why can't Kubernetes run containers by itself without a Container Runtime?**
    *K8s is an orchestrator (scheduler/control plane), not a container runtime. `kubelet` delegates actual container execution to a CRI runtime like `containerd` or Docker.*
14. **Why do we need a Kubernetes Service if every Pod already gets an IP?**
    *Pods are ephemeral—when a Pod crashes or scales, the replacement gets a **new IP**. A Service provides a permanent IP/DNS endpoint and load-balances traffic to healthy Pods.*
15. **What is the role of `etcd` in Kubernetes?**
    *It is the distributed key-value database storing the entire cluster's state and configuration. If `etcd` is lost without backup, the cluster state is lost.*
16. **A Pod is stuck in `Pending` vs `CrashLoopBackOff` — difference?**
    *`Pending` = Scheduler can't place it on any node (low CPU/RAM, scheduling constraints). `CrashLoopBackOff` = Placed and started, but app inside keeps crashing (`kubectl logs --previous`).*
17. **How does Nagios actually monitor a remote server?**
    *Nagios Core's scheduler periodically invokes modular **Plugins** which query the target host/service and return status codes (`OK`/`WARNING`/`CRITICAL`) to update the UI and fire alerts.*
18. **Why did Apache (`httpd`) stop working on EC2 after a reboot?**
    *It was started (`systemctl start httpd`) but not enabled on boot. Fix: `sudo systemctl enable httpd`.*
19. **My EC2 web app works via `curl localhost:8080` on SSH, but not from my laptop browser. Why?**
    *AWS **Security Group** inbound rules are blocking port `8080`. Add a Custom TCP inbound rule for port `8080`.*
20. **How do you keep an EC2 instance's public IP from changing after stop/start?**
    *Allocate and associate an **AWS Elastic IP (EIP)** with the instance.*

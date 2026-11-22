# Software Engineering Lab — Revision Manual

**Subject:** Software Engineering Lab · B.Tech III Year I Sem (KR24)
**Source:** `SE_KR24.pdf` lab record
**Purpose:** Last-minute revision — steps, commands and a short "why" for every experiment.
**Companion file:** [`sbq.md`](sbq.md) — all scenario-based questions with explained answers.

> **Conventions**
> - Commands are for **Git Bash** (Windows) unless marked Linux/PowerShell.
> - Replace `<...>` placeholders with your own values.
> - `docker compose` = Compose v2. On older installs use `docker-compose`.
> - "Why" lines tell you what the command actually does — enough to answer viva.

---

## Contents

| Week | Experiment | Main tools |
|---|---|---|
| [1](#week-1-software-installation-and-srs-document) | Software installation and SRS document | Word/LaTeX, SRDC case study |
| [2](#week-2-git-local-and-remote-commands) | Git local and remote commands on a multi-folder project | Git, GitHub |
| [3](#week-3-collaborative-coding-using-git) | Collaborative coding: organization, conflicts, patches | Git, GitHub |
| [4](#week-4-build-and-package-java-and-web-apps-using-maven) | Build and package Java/Web apps using Maven | Maven, Eclipse, Tomcat |
| [5 and 6](#week-5-and-6-docker-cli-and-dockerfile) | Docker CLI commands and Dockerfile | Docker, Docker Hub |
| [7](#week-7-multi-container-apps-with-docker-compose) | Multi-container apps with Docker Compose | Docker Compose |
| [8](#week-8-jenkins-automation) | Jenkins automation (freestyle jobs + pipeline view) | Jenkins, Maven, Git |
| [9](#week-9-pipeline-creation-using-script) | Scripted/declarative Jenkins pipeline | Jenkins |
| [10](#week-10-minikube-and-nagios) | Minikube, Nagios and AWS free tier | Minikube, kubectl, Nagios |
| [11](#week-11-jenkins-cicd) | Jenkins CI/CD: webhooks and email notifications | Jenkins, ngrok, Gmail |
| [12](#week-12-ubuntu-vm-and-cloud-deployment) | Ubuntu VM and public deployment of a web app | AWS EC2, Docker |
| [Appendix](#appendix-command-index) | One-page command index | — |

**Lab at a glance**

| # | One-line takeaway |
|---|---|
| 1 | An SRS fixes *what* the product must do (FR/NFR/users) before *how* it is built. |
| 2 | Git = local snapshots + remotes; push/pull/fetch move commits between them. |
| 3 | Collaboration = branches + pull requests; conflicts are resolved by editing markers. |
| 4 | Maven = standard layout + lifecycle (`clean → compile → test → package → install`). |
| 5/6 | Image = blueprint, container = running instance; Dockerfile automates image builds. |
| 7 | Compose = one YAML file that starts a whole multi-container stack. |
| 8 | Jenkins freestyle jobs automate Maven build → test → deploy through upstream/downstream jobs. |
| 9 | A Jenkinsfile is pipeline-as-code: stages, triggers and post-actions in Git. |
| 10 | Minikube runs a one-node Kubernetes cluster; Nagios watches hosts/services. |
| 11 | Webhooks make GitHub push events to Jenkins instantly; email reports build results. |
| 12 | EC2 VM + Docker + security-group port = app reachable on a public IP. |

---

## Week 1: Software Installation and SRS Document

**Goal:** Install the lab toolchain and write a complete SRS for the case study **SRDC — Semantics-based Ransomware Detection and Classification**.

### 1.1 Toolchain checklist

| Tool | Verify | Typical install |
|---|---|---|
| Git | `git --version` | `winget install Git.Git` |
| JDK 17 | `java -version`, `javac -version` | `winget install EclipseAdoptium.Temurin.17.JDK` |
| Maven | `mvn -v` | `winget install Apache.Maven` |
| Eclipse IDE | Help → About | eclipse.org installer (Enterprise Java) |
| Docker Desktop | `docker --version`, `docker run hello-world` | docker.com |
| Jenkins | open `http://localhost:8080` | `java -jar jenkins.war` or Windows installer |
| Minikube + kubectl | `minikube version`, `kubectl version --client` | `winget install Kubernetes.minikube` |
| ngrok | `ngrok version` | ngrok.com download |

```bash
git --version
java -version
mvn -v
docker --version
```

> **Why:** each lab week depends on the previous tool — Git for source, Maven for builds, Docker for packaging, Jenkins for automation, Minikube for orchestration.

### 1.2 What an SRS contains

| Section | What to write |
|---|---|
| 1. Introduction | Purpose of the document, product scope, intended audience |
| 1.1 Purpose | Why the SRS exists and what product it describes |
| 1.2 Intended audience | Who should read it (companies, individual users, developers) |
| 2. Specific requirements | Functional + non-functional requirements |
| 2.1 Functional requirements | What the system must *do* |
| 2.2 Non-functional requirements | Quality attributes: performance, security, usability |
| 3. SDLC / architecture | Development approach and system design |
| 4. Users | User types and their workflows |

### 1.3 SRDC case study (memorise this)

**Abstract in one line:** Detect and classify *zero-day ransomware* from system logs using the **SRDC** (System-call Relation and Dependency Classification) method, so malicious behaviour is caught the moment it appears.

**Functional requirements (FR)**

1. Detect ransomware.
2. Classify the identified ransomware (family).
3. Give a confidence percentage for the detection.
4. Explain the identified ransomware (why it was flagged).

**Non-functional requirements (NFR)**

1. Detect automatically in the background.
2. Detect ransomware at the moment of entry (early detection).
3. Consume minimal system resources while running in the background.

**Users**

1. Large companies managing many systems that need ransomware protection.
2. Individual users who care about data security.

**Workflow of each user**

| User | Workflow |
|---|---|
| Company admin | Install/deploy the agent on the fleet → monitoring runs in the background → alert on suspicious behaviour → review the classification, confidence and explanation → quarantine/clean the host → feed the new family back into the model |
| Individual user | Install on one machine → background monitoring → receive an alert with the family and confidence → review the explanation → remove the threat |

**Use cases**

1. **Detect** — identify ransomware behaviour from system logs.
2. **Classify** — name the ransomware family.
3. **Explain** — show the confidence % and the evidence behind the decision.
4. **Alert** — notify the user/admin at the moment of entry.
5. **Learn** — update the model when a new family is discovered.

**Architecture pipeline**

```text
Raw system logs
   ↓  Data ingestion (parse + normalise logs)
   ↓  Feature extraction (API calls, registry keys, strings)
   ↓  GPT-2 (pre-trained language model → embeddings)
   ↓  MLP (Multi-Layer Perceptron classifier)
   ↓  Ransomware family prediction + confidence + explanation
```

> **Why:** logs are text, GPT-2 turns behaviour into vectors, and the MLP maps those vectors to a ransomware family. The telemetry loop feeds newly discovered strains back into the model.

**Revision points**

- SRS is a contract between customer and developer — it states *what*, not *how*.
- FR = behaviour; NFR = quality constraint.
- Use cases: *detect*, *classify*, *explain*, *alert* — one actor (user/admin) per workflow.
- Zero-day ransomware has no signature, so **behaviour** (system calls) is used instead of signatures.

---

## Week 2: Git Local and Remote Commands

**Goal:** Push a multi-folder project to a **private** GitHub repository and practise the full set of local and remote Git commands.

### 2.1 Push a multi-folder project to a private repo

```bash
# 1. Create an empty PRIVATE repository on github.com (New repository → Private → Create)
# 2. Initialise Git inside the project root
cd /path/to/project
git init

# 3. Connect the local repo to GitHub
git remote add origin https://github.com/<username>/<repo>.git

# 4. Stage everything and take the first snapshot
git add .
git commit -m "Initial commit"

# 5. Upload and remember the upstream branch
git push -u origin main        # use master if that is your branch name
```

> **Why:** `git init` creates the hidden `.git` database, `add` moves files to the staging area, `commit` records a snapshot, `push` copies commits to GitHub. `-u` links the local branch to `origin/main`, so later you can just type `git push`.

### 2.2 Setup and configuration

| Command | What it does |
|---|---|
| `git --version` | Shows the installed Git version |
| `git config --global user.name "Your Name"` | Sets the author name for all repos |
| `git config --global user.email "you@example.com"` | Sets the author email |
| `git config --list` | Prints all active settings |
| `git config --global --edit` | Opens the global config file |

### 2.3 Repository and remotes

| Command | What it does |
|---|---|
| `git init` | Turns the current folder into a Git repository |
| `git clone <url>` | Downloads a remote repo with full history |
| `git remote -v` | Lists remotes and their URLs |
| `git remote add origin <url>` | Adds a remote named `origin` |
| `git remote remove origin` | Removes the remote (local commits stay) |
| `git remote rename origin upstream` | Renames a remote |
| `git remote set-url origin <new-url>` | Points the remote at a new URL |
| `git remote show origin` | Detailed remote info: branches, tracking, HEAD |
| `git remote prune origin` | Deletes stale `origin/*` tracking branches |
| `git fetch origin` | Downloads remote commits **without** merging |
| `git pull origin main` | Fetch **and** merge into the current branch |
| `git push origin main` | Uploads local commits |
| `git push -u origin main` | First push + sets upstream |
| `git branch -r` | Lists remote-tracking branches |
| `git branch -a` | Lists local **and** remote branches |

### 2.4 Staging and committing

| Command | What it does |
|---|---|
| `git status` | Shows untracked / modified / staged files |
| `git add file.txt` | Stages one file |
| `git add .` | Stages all changes in the current tree |
| `git commit -m "msg"` | Records the staged snapshot |
| `git commit -am "msg"` | Stages tracked files and commits in one step |
| `git diff` | Unstaged changes (working tree vs index) |
| `git diff --staged` | Staged changes (index vs last commit) |
| `git log --oneline --graph --decorate` | Compact visual history |

### 2.5 Branching and merging

| Command | What it does |
|---|---|
| `git branch` | Lists local branches |
| `git branch <name>` | Creates a branch (does not switch) |
| `git switch <name>` / `git checkout <name>` | Switches branch |
| `git switch -c <name>` | Creates **and** switches |
| `git branch -d <name>` | Deletes a merged branch |
| `git branch -D <name>` | Force-deletes an unmerged branch |
| `git merge <name>` | Merges `<name>` into the current branch |
| `git rebase origin/main` | Replays local commits on top of remote (linear history) |
| `git push --set-upstream origin <branch>` | Pushes a new branch and tracks it |
| `git fetch origin <branch>` | Fetches one specific branch |
| `git remote prune origin` | Cleans deleted remote branches |

### 2.6 Undo, recover and inspect

| Command | What it does |
|---|---|
| `git restore <file>` | Discards unstaged changes in a file |
| `git restore --staged <file>` | Unstages a file, keeps the edits |
| `git reset <file>` | Older form of unstaging |
| `git reset --soft HEAD~1` | Undoes the last commit, keeps changes staged |
| `git reset --hard HEAD~1` | Undoes the last commit **and deletes** changes |
| `git revert <hash>` | New commit that reverses an old commit (safe on shared branches) |
| `git commit --amend -m "new msg"` | Rewrites the last commit message/content |
| `git reflog` | Every position of HEAD — the recovery log |
| `git checkout -b <branch> <hash>` | Recreates a deleted branch at a commit |
| `git show <hash>` | Details and diff of one commit |
| `git blame script.py` | Last author/commit per line |
| `git blame -L 25,25 script.py` | Blame only line 25 |
| `git stash` | Shelves uncommitted work |
| `git stash list` | Shows saved stashes |
| `git stash apply` | Restores the latest stash (keeps it) |
| `git stash pop` | Restores and deletes the stash |
| `git branch --merged` | Branches already merged into HEAD |
| `git branch -d b1 b2 b3` | Deletes several merged branches at once |

### 2.7 Remove a sensitive file from history

```bash
# Modern, recommended
pip install git-filter-repo
git filter-repo --path secrets.txt --invert-paths

# Legacy (as in the record)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch secrets.txt" \
  --prune-empty --tag-name-filter cat -- --all

git push --force --all
git push --force --tags
```

> **Why:** a normal `git rm` only deletes the file in a *new* commit — old commits still contain it. History rewriting removes every trace. Always **rotate the leaked secret** afterwards.

### 2.8 `.gitignore`

```bash
touch .gitignore
```

```gitignore
*.log
secret_file.txt
temp/
node_modules/
target/
*.bak

# keep one important file inside an ignored folder
folder/*
!folder/important_file.txt
```

> **Why:** ignored files are never staged, so build output, logs and secrets stay out of the repo. Already-tracked files must be untracked first: `git rm --cached <file>`.

### 2.9 Resolve a merge conflict

```bash
git status                    # shows "both modified: app.js"
# open app.js and edit the markers:
#   <<<<<<< HEAD
#   your version
#   =======
#   their version
#   >>>>>>> feature/x
git add app.js
git commit                    # only if Git did not auto-create the merge commit
```

Abort if it gets messy:

```bash
git merge --abort
```

> **Why:** Git cannot choose between two edits to the same lines, so it writes both versions with markers. You keep the correct code, delete the markers, stage the file and finish the merge.

### 2.10 Typical workflows to remember

```bash
# Start a feature
git switch main
git pull origin main
git switch -c feature/login

# Finish a feature
git add .
git commit -m "Add login page"
git push -u origin feature/login
# then open a Pull Request on GitHub
```

```bash
# Recover a deleted branch
git reflog
git branch feature-ui <commit-hash>
```

```bash
# Keep a branch linear with remote
git fetch origin
git rebase origin/main
```

**Gotchas**

- `git pull` = `fetch` + `merge`; use `git pull --rebase` to avoid noise commits.
- Never `push --force` a shared branch; use `--force-with-lease` if unavoidable.
- `git reset --hard` destroys uncommitted work — stash first.
- `git branch -d` refuses unmerged branches; that is a safety feature, not a bug.

---

## Week 3: Collaborative Coding Using Git

**Goal:** Collaborate through a GitHub **organization** and a **shared repository**, resolve conflicts, and exchange work as **patches**.

### 3.1 Create an organization

1. GitHub → **New organization** → **Create a free organization**.
2. Enter organization name, contact email, verify the account → **Next**.
3. Add members (invitations must be accepted) → **Complete setup**.
4. **Settings → Member privileges → Base permissions → Write**.
5. **Projects base permissions → Write**.
6. Create the project repository **inside the organization** (private if needed).

> **Why:** an organization is a shared account that owns repositories; base permissions decide what every member can do without being added repo-by-repo.

### 3.2 Collaborator workflow (shared repository)

**Owner (Collaborator 1)**

1. Repository → **Settings → Collaborators → Add people**.
2. Enter the teammate's GitHub username → invite.
3. Teammate accepts the email/GitHub invitation.

**Collaborator 2**

```bash
# 1. Identity
git config --global user.name "Your Name"
git config --global user.email you@example.com

# 2. Clone the shared repo
git clone https://github.com/<owner>/<repo>.git
cd <repo>

# 3. Work on a branch, never directly on main
git switch -c feature/my-feature

# 4. Edit files, then commit
git add .
git commit -m "Add a clear and descriptive message"

# 5. Publish the branch
git push origin feature/my-feature
```

```bash
# 6. Stay up to date before new work
git switch main
git pull origin main
git switch feature/my-feature
git merge main
```

> **Why:** branches isolate unfinished work; the pull request is where the team reviews it before it reaches `main`.

### 3.3 Fork and pull request (public repository)

1. Open `https://github.com/Person1/awesome-project`.
2. Click **Fork** → GitHub creates `https://github.com/Person2/awesome-project`.
3. Clone **your fork**, make changes, commit, push.
4. Click **Contribute → Open pull request** (or **Pull requests → New**).
5. Add a title and description → **Create pull request**.
6. Owner reviews **Files changed** → **Review changes** → **Submit review** → **Merge pull request** → **Confirm merge**.
7. Both sides sync:

```bash
git switch main
git pull origin main
```

> **Why:** you cannot push to someone else's public repo, but you can push to your fork and ask the owner to merge your commits.

### 3.4 Resolve a conflict (two collaborators, same lines)

```bash
git status                     # both modified: f1.txt
# edit f1.txt, keep one or both versions, delete all markers
git add f1.txt
git commit                     # completes the merge
# or, while rebasing:
git rebase --continue
```

Conflict markers:

```text
def greet():
<<<<<<< HEAD
    print("Hello from First collaborator")
=======
    print("Hello from second collaborator")
>>>>>>> feature/second-collaborator
```

> **Why:** `HEAD` is your current branch; the part below `=======` is the incoming branch. Git stops the merge until a human decides.

Abort:

```bash
git merge --abort
```

### 3.5 Create and apply a patch

```bash
# 1. Clone the project and make a change
git clone https://github.com/<user>/<repo>.git
cd <repo>
# edit a file
git add .
git commit -m "Fix: small change"

# 2. Find the commit
git log --oneline

# 3. Export it as a patch
git format-patch -1 <commit-hash>
# creates 0001-Fix-small-change.patch

# 4. Send the .patch file to the owner (email/chat)

# 5. Owner applies it
git am 0001-Fix-small-change.patch     # keeps author + commit message
# or, if it is a plain diff:
git apply my-changes.patch
```

| Command | Use |
|---|---|
| `git format-patch -1 <hash>` | One commit → mailbox patch (author preserved) |
| `git format-patch -3` | Last three commits |
| `git format-patch <base>..HEAD` | A range of commits |
| `git am file.patch` | Apply a `format-patch` file as a commit |
| `git apply file.patch` | Apply a raw diff without committing |

> **Why:** a patch is a portable diff. `format-patch` + `am` preserves authorship and commit messages, which is why it is preferred for email-based contribution.

**Gotchas**

- Always `git pull` before starting work; most conflicts come from stale branches.
- A conflict is not an error — it is Git asking for a decision.
- `git am` fails on an already-applied patch; check with `git am --abort` / `git apply --check`.

---

## Week 4: Build and Package Java and Web Apps Using Maven

**Goal:** Understand Maven's standard structure and lifecycle, add dependencies, run tests, build executable JARs and deployable WARs, and handle multi-module projects.

### 4.1 Standard project structure

```text
my-project/
├── pom.xml                      ← project object model (build recipe)
└── src/
    ├── main/
    │   ├── java/                ← application source
    │   ├── resources/           ← config.properties, etc.
    │   └── webapp/              ← only for web apps (WAR)
    │       └── WEB-INF/web.xml
    └── test/
        ├── java/                ← unit tests
        └── resources/           ← test resources
```

After a build, everything generated lands in `target/`.

> **Why:** convention over configuration — Maven knows where sources and tests are, so no paths need to be configured.

### 4.2 Lifecycle (memorise the order)

| Phase | What happens |
|---|---|
| `validate` | Project is correct and all info is available |
| `compile` | `src/main/java` → `target/classes` |
| `test` | Runs JUnit tests via Surefire |
| `package` | Produces the JAR/WAR in `target/` |
| `verify` | Runs integration checks |
| `install` | Copies the artifact to `~/.m2/repository` |
| `deploy` | Uploads to a remote repository (Nexus/Artifactory) |

```bash
mvn clean            # delete target/
mvn compile          # compile main sources
mvn test             # compile + run tests
mvn package          # build JAR/WAR
mvn install          # package + copy to local .m2
mvn clean install    # the full everyday build
mvn clean install -DskipTests           # skip running tests (still compiles them)
mvn clean install -Dmaven.test.skip=true # skip compiling and running tests
```

> **Why:** `clean` prevents stale classes from hiding errors; `install` makes the artifact available to other local projects.

### 4.3 Create a Maven Java project (Eclipse)

1. **File → New → Maven Project**.
2. Choose archetype `maven-archetype-quickstart` (1.4).
3. Group Id `com.example`, Artifact Id `my-maven-project`, Version default → **Finish**.
4. Answer **Y** when Maven asks to download archetype artifacts.
5. Right-click project → **Maven → Update Project** if dependencies look stale.
6. Right-click `App.java` → **Run As → Maven Clean / Install / Test**, or **Maven Build** with goals `clean install test`.
7. Check console for `BUILD SUCCESS`.
8. **Run As → Java Application** → prints `Hello World`.

Equivalent from the terminal:

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app \
  -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
cd my-app
mvn clean install
```

### 4.4 Create a Maven web project (Eclipse + Tomcat)

1. **File → New → Maven Project** → archetype `maven-archetype-webapp` (1.4).
2. Group Id `com.example`, Artifact Id `my-web-app` → **Finish**.
3. Add the Servlet API dependency in `pom.xml` (from mvnrepository.com):

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>
```

4. **Window → Show View → Servers** → add **Tomcat v9.0** → configure port (8080).
5. Edit `tomcat-users.xml` to enable the manager:

```xml
<role rolename="manager-gui"/>
<user username="admin" password="1234" roles="manager-gui,manager-script"/>
```

6. Build with goals `clean install test`.
7. Right-click `index.jsp` → **Run As → Run on Server** → choose Tomcat → **Finish**.
8. Open `http://localhost:8080/my-web-app/` → "Hello World" page.

> **Why:** a web app needs a servlet container. `provided` means Tomcat supplies the API at runtime, so it must not be bundled into the WAR.

### 4.5 `pom.xml` essentials

**Packaging**

```xml
<packaging>jar</packaging>   <!-- default -->
<packaging>war</packaging>   <!-- web app -->
```

**Dependency (Gson example)**

```xml
<dependency>
  <groupId>com.google.code.gson</groupId>
  <artifactId>gson</artifactId>
  <version>2.10</version>
</dependency>
```

> **Why:** Maven downloads the JAR and its transitive dependencies into `~/.m2/repository/...` and adds them to the classpath. A wrong version → `BUILD FAILURE` with a dependency resolution error.

**Compiler plugin (fix "unsupported class version")**

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
  <configuration>
    <source>17</source>
    <target>17</target>
  </configuration>
</plugin>
```

**Executable JAR**

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.example.Main</mainClass>
          </manifest>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```

```bash
mvn package
java -jar target/my-app-1.0-SNAPSHOT.jar
jar tf target/my-app-1.0-SNAPSHOT.jar    # inspect contents
```

> **Why:** the manifest's `Main-Class` tells the JVM which class to run. A plain JAR does **not** include dependencies — use the Shade or Assembly plugin for a fat JAR.

**Custom output directory**

```xml
<build>
  <directory>build_output</directory>
</build>
```

**WAR structure**

```text
src/main/webapp/
├── index.jsp
└── WEB-INF/
    └── web.xml
```

```bash
mvn package
# target/my-web-app.war → copy to Tomcat webapps/ or deploy from Eclipse
```

### 4.6 JUnit tests and reports

```java
public class AppTest {
    @Test
    public void testSum() {
        assertEquals(5, 2 + 3);
    }
}
```

```bash
mvn test
```

| Output | Meaning |
|---|---|
| `target/test-classes/` | Compiled test classes |
| `target/surefire-reports/` | `.txt` and `.xml` results |
| `target/surefire-reports/*.txt` | Failure trace for a failed test |

> **Why:** Surefire is the plugin bound to the `test` phase; its reports are what CI tools read.

### 4.7 Resource files

```text
src/main/resources/config.properties
```

```java
InputStream input =
    getClass().getClassLoader().getResourceAsStream("config.properties");
```

After `mvn package`, confirm `target/classes/config.properties` exists.

> **Why:** Maven copies `src/main/resources` into `target/classes`, which is on the runtime classpath — this is why the file is read with the classloader, not a file path.

### 4.8 Multi-module / parent project

```text
parent/
├── pom.xml          <packaging>pom</packaging> + <modules>
├── core/
│   └── pom.xml
└── web/
    └── pom.xml
```

```xml
<!-- parent pom.xml -->
<packaging>pom</packaging>
<modules>
  <module>core</module>
  <module>web</module>
</modules>
```

```xml
<!-- web/pom.xml depends on core -->
<dependency>
  <groupId>com.example</groupId>
  <artifactId>core</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

```bash
cd parent
mvn clean install
```

> **Why:** the parent centralises versions and plugins; Maven builds modules in dependency order (core before web). The parent itself produces no JAR.

### 4.9 Common Maven errors

| Error | Cause | Fix |
|---|---|---|
| `BUILD FAILURE` + dependency not found | Wrong version / missing repo | Correct the `<version>`, run `mvn -U clean install` |
| `Unsupported class version` | Compiling with a newer JDK than the target | Set `source`/`target`/`release`, align `JAVA_HOME` |
| `maven-compiler-plugin ... compile` | Real error is above the `[Help 1]` line | Re-run with `mvn -e -X` and read the first `ERROR` |
| Tests fail | Assertion or environment | Read `target/surefire-reports` |
| Dependency conflict | Two versions of the same library | `mvn dependency:tree`, add `<exclusions>` or `<dependencyManagement>` |

```bash
mvn dependency:tree
mvn dependency:tree -Dverbose
mvn dependency:resolve -U
mvn help:effective-pom
```

> **Why:** Maven uses *nearest-wins* and *first-declared-wins* for transitive versions; the dependency tree shows exactly which version was chosen and who pulled it in.

**Gotchas**

- `-DskipTests` still compiles tests; `-Dmaven.test.skip=true` skips them entirely.
- A WAR is deployed to a container; a JAR is run by the JVM (`java -jar`).
- `provided` scope = the server already has it (Servlet API, JSP API).
- Always run from the folder containing `pom.xml`, or use `-f path/to/pom.xml`.

## Week 5 and 6: Docker CLI and Dockerfile

**Goal:** Pull, run, inspect and remove containers/images; write a Dockerfile, build a custom image, publish it to Docker Hub and run it elsewhere.

### 5.1 Core vocabulary

| Term | Meaning |
|---|---|
| **Image** | Read-only blueprint: app code + runtime + libraries |
| **Container** | Running instance of an image |
| **Dockerfile** | Text recipe used to build an image |
| **Registry** | Image store (Docker Hub) |
| **Volume** | Persistent storage that survives container deletion |
| **Network** | Virtual network that lets containers talk by name |

> **Why Docker:** the image carries its own dependencies, so the app behaves the same on every machine.

### 5.2 Redis CLI walkthrough

```bash
# 1. Download the image
docker pull redis

# 2. Start a container in the background
docker run --name my-redis -d redis

# 3. Confirm it is running
docker ps

# 4. Open the Redis shell inside the container
docker exec -it my-redis redis-cli
```

```text
127.0.0.1:6379> SET name "Alice"
OK
127.0.0.1:6379> GET name
"Alice"
127.0.0.1:6379> exit
```

```bash
# 5. Stop, restart, remove
docker stop my-redis
docker start my-redis
docker rm my-redis          # container must be stopped first
docker rmi redis            # remove the image
```

> **Why:** `-d` detaches the container (runs in background); `exec -it` opens an interactive terminal inside a running container.

### 5.3 Dockerfile: build your own Redis image

```bash
mkdir -p ~/DockerProjects/Redis
cd ~/DockerProjects/Redis
```

```dockerfile
FROM redis:latest
CMD ["redis-server"]
```

```bash
docker build -t redisnew .              # . = build context (current folder)
docker run --name myredisnew -d redisnew
docker ps
docker stop myredisnew
```

> **Why:** `FROM` sets the base image, `CMD` is the default process. `-t` names/tags the image so it is easy to find.

### 5.4 Publish to Docker Hub and pull elsewhere

```bash
docker login                              # Docker Hub username + password/token
docker ps -a                              # get the container ID
docker commit <container-id> <dockerhub-user>/redis1
docker images
docker push <dockerhub-user>/redis1

# On another machine
docker pull <dockerhub-user>/redis1
docker run --name myredis -d <dockerhub-user>/redis1
docker exec -it myredis redis-cli
```

```text
SET name "Abcdef"
GET name
exit
```

```bash
docker stop myredis
docker rm <container-id>
docker rmi <dockerhub-user>/redis1
docker logout
```

> **Why:** `commit` snapshots a container into an image (fine for the lab, but a Dockerfile is the reproducible way). `push`/`pull` move the image through the registry.

### 5.5 Networks

```bash
docker network create app-net
docker network ls
docker network inspect app-net

docker run -d --name web --network app-net nginx
docker run -d --name api --network app-net <image>
docker network connect app-net <container>
docker network disconnect app-net <container>
docker network rm app-net
```

> **Why:** containers on the same user-defined network resolve each other by **container name** (DNS). The default bridge network does not provide this by name.

### 5.6 Volumes (persistent storage)

```bash
docker volume create db-data
docker volume ls
docker volume inspect db-data

docker run -d --name db \
  -v db-data:/var/lib/postgresql/data \
  postgres:15

docker volume rm db-data     # only when no container uses it
```

> **Why:** a container's writable layer disappears with the container; a named volume lives outside it and is re-attached on the next run.

### 5.7 Monitor and troubleshoot

| Command | What it shows |
|---|---|
| `docker ps` | Running containers |
| `docker ps -a` | All containers, including stopped |
| `docker logs <c>` | Container stdout/stderr |
| `docker logs -f --tail 100 <c>` | Follow the last 100 lines live |
| `docker inspect <c>` | Full JSON: IP, mounts, env, state |
| `docker stats` | Live CPU / memory / network per container |
| `docker top <c>` | Processes inside the container |
| `docker port <c>` | Port mappings |
| `docker exec -it <c> sh` | Shell inside the container |
| `docker cp <c>:/path ./local` | Copy files in/out |
| `docker diff <c>` | Files changed since the image was built |

```bash
docker run -d --name web_app -p 8080:5000 --restart unless-stopped <image>
docker update --restart=always web_app
docker run -d --name limited --memory=256m --cpus=1 nginx
```

> **Why:** `-p host:container` publishes a port; `--restart` controls what happens after a crash or reboot; `--memory/--cpus` cap resource usage.

### 5.8 Cleanup

```bash
docker stop $(docker ps -q)          # stop all running containers
docker rm $(docker ps -aq)           # remove all containers
docker rmi <image>                   # remove one image
docker system prune -a               # remove unused containers/images/networks/cache
docker system df                     # disk usage summary
```

**Gotchas**

- `docker rm` needs a stopped container (or `-f`).
- `docker rmi` fails while a container still uses the image.
- `-p 8080:80` means **host 8080 → container 80** — the app inside listens on 80.
- Containers are ephemeral: anything not in a volume is lost on removal.
- `docker commit` is a lab shortcut; real projects use a Dockerfile.

---

## Week 7: Multi-Container Apps with Docker Compose

**Goal:** Define several interdependent services (web server, database, cache) in one `docker-compose.yml`, run them together, add networking and persistent storage, and rebuild quickly during development.

### 7.1 Why Compose

Without Compose you would run one long `docker run` per service and wire networks/volumes manually. Compose lets you declare the whole stack once and start it with one command — identical on every machine.

### 7.2 File anatomy

```yaml
version: "3.9"          # compose file format (v2 ignores this field, harmless)

services:               # one entry per container
  web:
    image: nginx:latest
    ports:
      - "8080:80"       # host:container
    environment:
      - APP_MODE=dev
    depends_on:
      - db              # start db first (not a health guarantee)
    networks:
      - app-net
    volumes:
      - db-data:/var/lib/postgresql/data

networks:               # optional: custom bridge network
  app-net:

volumes:                # optional: named persistent volume
  db-data:
```

| Key | Purpose |
|---|---|
| `image` | Image to run |
| `build` | Build from a local Dockerfile instead of pulling |
| `ports` | Publish container ports on the host |
| `environment` | Environment variables |
| `depends_on` | Start order |
| `networks` | Attach services to a shared network |
| `volumes` | Persist data / bind-mount source |
| `restart` | Restart policy |

### 7.3 Essential commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Build if needed and start all services detached |
| `docker compose up --build` | Force a rebuild before starting |
| `docker compose ps` | Status of the stack |
| `docker compose logs -f` | Follow logs of all services |
| `docker compose logs -f web` | Logs of one service |
| `docker compose restart` | Restart all services |
| `docker compose stop` | Stop without removing |
| `docker compose start` | Start stopped services |
| `docker compose down` | Stop **and remove** containers + default network |
| `docker compose down -v` | Also remove named volumes (**data loss**) |
| `docker compose up --scale web=2 -d` | Run 2 replicas of `web` |
| `docker compose build web` | Rebuild only `web` |
| `docker compose exec web sh` | Shell inside a running service |
| `docker compose config` | Validate/render the final YAML |

> **Why `-d`:** detach mode returns your terminal; the stack keeps running in the background.

### 7.4 Task 1: two interdependent services

```yaml
version: "3.9"
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
```

```bash
docker compose up -d
docker compose ps
```

Open `http://localhost:8080` → Nginx welcome page; Postgres runs in the background.

### 7.5 Task 2: add a Redis cache and start order

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - redis
  redis:
    image: redis:alpine
```

```bash
docker compose up -d
docker compose ps
```

Expected: three services (`web`, `db`, `redis`) running.

> **Why `depends_on`:** Compose starts `redis`/`db` before `web`. It does **not** wait for the app inside to be ready — use a healthcheck for that.

### 7.6 Task 3: deploy on another machine

1. Zip the `compose-lab` folder (or push it to Git).
2. Copy it to any machine with Docker + Compose.
3. Run:

```bash
docker compose up -d
```

> **Why:** the YAML is the single source of truth — same images, same ports, same environment, no manual setup.

### 7.7 Task 4: custom network + persistent volume

```yaml
networks:
  app-net:

volumes:
  db-data:

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    networks:
      - app-net
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net
```

```bash
docker compose up -d
# insert data with psql (optional)
docker compose down
docker compose up -d
```

Expected: data survives `down` + `up`; services reach each other by name (`db:5432`) on `app-net`.

> **Why:** `docker compose down` removes containers but **not** named volumes; `-v` would delete them too.

### 7.8 Task 5: rebuild only what changed

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask + Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```

```yaml
services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - db
```

```bash
docker compose up --build
# edit the return text in app.py
docker compose up --build
```

Open `http://localhost:5000` → the new message appears.

> **Why:** `build: .` tells Compose to build the local Dockerfile; `--build` forces a fresh image so code changes are included.

**Gotchas**

- Two services cannot publish the same **host** port — change one host side (`8081:80`).
- `depends_on` controls order, not readiness; add `healthcheck` + `condition: service_healthy`.
- `docker compose down` keeps volumes unless you add `-v`.
- Service names are the DNS names inside the network (`db`, `redis`, `web`).

---

## Week 8: Jenkins Automation

**Goal:** Create Jenkins freestyle jobs that build and test the Maven Java and Maven Web projects from GitHub, chain them (build → test → deploy), and visualise them in a pipeline view.

### 8.1 Jenkins setup

```bash
# Standalone (Windows/Linux/macOS)
java -jar jenkins.war --httpPort=8080
```

- Open `http://localhost:8080`.
- Unlock with the initial admin password:
  - Windows: `C:\Program Files\Jenkins\secrets\initialAdminPassword`
  - Linux: `/var/lib/jenkins/secrets/initialAdminPassword`
- Install suggested plugins; create the admin user.

**Plugins used in this lab**

| Plugin | Why |
|---|---|
| Git | Clone GitHub repositories |
| Maven Integration | "Invoke top-level Maven targets" build step |
| Pipeline / Build Pipeline | Pipeline jobs and pipeline view |
| Copy Artifact | Copy build output into the test job |
| Deploy to container | Deploy a WAR to Tomcat |
| Email Extension | Build notifications (Week 11) |

**Global tools**

**Manage Jenkins → Tools** → add JDK and Maven. Note the Maven **name** (e.g. `MAVEN_HOME`) — the job must reference exactly that name.

> **Why:** Jenkins needs to know where Maven lives; the name is the handle used inside jobs and Jenkinsfiles.

### 8.2 Maven Java: `MavenJava_Build`

1. **New Item** → name `MavenJava_Build` → **Freestyle project** → OK.
2. Description: `Java Build demo`.
3. **Source Code Management → Git** → repository URL of the Maven Java project.
4. Branches to build: `*/main` (or `*/master`).
5. **Build Steps**:
   - *Invoke top-level Maven targets* → Maven `MAVEN_HOME` → Goals `clean`.
   - Add a second one → Goals `install`.
6. **Post-build Actions**:
   - *Archive the artifacts* → Files to archive `**/*`.
   - *Build other projects* → `MavenJava_Test` → trigger **only if build is stable**.
7. **Apply → Save**.

> **Why:** two Maven steps make the console easier to read (`clean` then `install`); archiving keeps the JAR available to downstream jobs.

### 8.3 Maven Java: `MavenJava_Test`

1. **New Item** → `MavenJava_Test` → Freestyle → OK.
2. Description: `Test demo`.
3. **Build Environment** → check **Delete the workspace before build starts**.
4. **Build Steps**:
   - *Copy artifacts from another project* → project `MavenJava_Build` → build **Stable build only** → artifacts `**/*`.
   - *Invoke top-level Maven targets* → `MAVEN_HOME` → Goals `test`.
5. **Post-build Actions** → *Archive the artifacts* → `**/*`.
6. **Apply → Save**.

> **Why:** a clean workspace guarantees the test runs on the freshly built artifact, not leftovers.

### 8.4 Pipeline view: `MavenJava_Pipeline`

1. Dashboard → **+** beside **All** → name `MavenJava_Pipeline`.
2. Select **Build pipeline view** → OK.
3. Layout: **Based on upstream/downstream relationship**.
4. Initial job: `MavenJava_Build` → **Apply → Save**.
5. Click **Run**; click the small console box to watch each stage.
6. Success = green boxes; `MavenJava_Test` triggers automatically.

### 8.5 Maven Web: `MavenWeb_Build` → `MavenWeb_Test` → `MavenWeb_Deploy`

**`MavenWeb_Build`**

1. Freestyle project, description `Web Build demo`.
2. Git URL of the Maven Web project; branch `*/main`.
3. Build steps: Maven `clean`, then `install` (Maven `MAVEN_HOME`).
4. Post-build: archive `**/*`; **Build other projects** → `MavenWeb_Test`, only if stable.

**`MavenWeb_Test`**

1. Freestyle project, description `Test demo`.
2. Check **Delete the workspace before build starts**.
3. Build steps: copy artifacts from `MavenWeb_Build` (stable only, `**/*`), then Maven goal `test`.
4. Post-build: archive `**/*`; **Build other projects** → `MavenWeb_Deploy`.

**`MavenWeb_Deploy`**

1. Freestyle project, description `Web Code Deployment`.
2. Check **Delete the workspace before build starts**.
3. Build step: copy artifacts from `MavenWeb_Test` (stable only, `**/*`).
4. Post-build: **Deploy WAR/EAR to a container**:
   - WAR/EAR file: `**/*.war`
   - Context path: `Webpath`
   - Container: **Tomcat 9.x remote**
   - Credentials: `admin` / `1234`
   - Tomcat URL: `http://localhost:8085/` (or your Tomcat port)
5. **Apply → Save**.

**`MavenWeb_Pipeline`**

1. **+** → `MavenWeb_Pipeline` → **Build pipeline view**.
2. Initial job `MavenWeb_Build` → Save → **Run**.
3. Open the Tomcat manager (`http://localhost:8085/manager/html`) → log in → click `/Webpath` → app page.

> **Why a deploy job:** the WAR is the artifact; Tomcat's manager API accepts it and deploys it under the context path, so no manual copy is needed.

**Gotchas**

- Jenkins on 8080 conflicts with Tomcat on 8080 — run Tomcat on 8085 (as above) or change one port.
- The Maven tool name in the job must match **Manage Jenkins → Tools** exactly.
- "Build other projects" only fires on the configured stability; a failed build stops the chain.
- Tomcat remote deploy needs a user with `manager-script` role in `tomcat-users.xml`.

---

## Week 9: Pipeline Creation Using Script

**Goal:** Replace clicking through freestyle jobs with **pipeline-as-code**: a Jenkinsfile that clones, cleans, installs, tests and packages a Maven project, plus build triggers.

### 9.1 Create the pipeline job

1. **New Item** → name (e.g. `MavenJava_Pipeline_Script`) → **Pipeline** → OK.
2. Description: what the pipeline does.
3. **Build Triggers**: choose one (see 9.3).
4. **Pipeline → Definition: Pipeline script**.
5. Paste the Jenkinsfile → **Apply → Save** → **Build Now**.

> **Why:** the script lives in the job (or in Git with "Pipeline script from SCM"), so the build definition is versioned and reviewable.

### 9.2 Declarative Jenkinsfile (PDF version, cleaned up)

```groovy
pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'          // must match Manage Jenkins → Tools
    }

    stages {
        stage('Checkout & Clean') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/<user>/<maven-java-repo>.git'
                bat 'mvn clean'                   // Windows
                // sh 'mvn clean'                 // Linux
            }
        }

        stage('Install') {
            steps {
                bat 'mvn install'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Package') {
            steps {
                bat 'mvn package'
            }
        }
    }

    post {
        success { echo 'Pipeline succeeded' }
        failure { echo 'Pipeline failed' }
    }
}
```

> **Why no `-f`:** the `git` step checks the repository out at the workspace root, which is where `pom.xml` lives. If your POM is inside a subfolder (the record uses `mavenjava`), either add `-f mavenjava` to each goal or wrap the steps in `dir('mavenjava') { ... }`.

### 9.3 Build triggers

| Trigger | Where | Meaning |
|---|---|---|
| `githubPush()` | `triggers` block + GitHub webhook | Build on every push (Week 11) |
| `pollSCM('* * * * *')` | `triggers` block | Check Git every minute |
| `cron('0 18 * * *')` | `triggers` block | Every day at 18:00 |
| Upstream job | Job config | Build after another job succeeds |
| Manual | Dashboard | **Build Now** |

```groovy
triggers {
    githubPush()
    // cron('0 18 * * *')
    // pollSCM('H/5 * * * *')
}
```

> **Why:** triggers decide *when* the pipeline runs; stages decide *what* runs.

### 9.4 Useful pipeline extras

```groovy
pipeline {
    agent any
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    environment {
        APP_ENV = 'staging'
    }
    stages {
        stage('Build') {
            steps {
                echo "Build number: ${env.BUILD_NUMBER}"
                bat 'mvn clean package'
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                echo 'Deploying...'
            }
        }
    }
    post {
        always  { junit 'target/surefire-reports/*.xml' }
        success { echo 'SUCCESS' }
        failure { echo 'FAILED' }
        unstable { echo 'UNSTABLE' }
    }
}
```

> **Why:** `post` always runs; `when` guards a stage; `junit` publishes test results in the Jenkins UI.

### 9.5 Credentials for a private repo

1. **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**.
2. Kind: *Username with password* (GitHub username + personal access token).
3. Note the **ID** (e.g. `github-creds`).

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            credentialsId: 'github-creds',
            url: 'https://github.com/<user>/<private-repo>.git'
    }
}
```

> **Why:** never hard-code tokens in a Jenkinsfile; Jenkins injects them only for that step and masks them in logs.

### 9.6 Windows vs Linux

| Windows agent | Linux agent |
|---|---|
| `bat 'mvn clean install'` | `sh 'mvn clean install'` |
| `bat 'dir'` | `sh 'ls -l'` |
| `bat 'rmdir /s /q folder'` | `sh 'rm -rf folder'` |

> **Why:** `bat` runs `cmd.exe`, `sh` runs the POSIX shell; using the wrong one gives "command not found".

**Gotchas**

- A failed stage stops the pipeline by default; `catchError` is what makes it continue.
- `tools { maven 'NAME' }` fails immediately if the name is not configured.
- `mvn test` failure returns a non-zero exit code → stage FAILED → later stages skipped.
- Put `post` actions at pipeline level, not inside a stage, for build-wide reporting.

---

## Week 10: Minikube and Nagios

**Goal:** Create, run and scale pods on a local Kubernetes cluster; expose Nginx; monitor infrastructure with Nagios in Docker; create an AWS free-tier account.

### 10.1 Kubernetes vocabulary

| Term | Meaning |
|---|---|
| **Cluster** | Set of machines managed by Kubernetes |
| **Node** | A machine (VM or physical) in the cluster |
| **Pod** | Smallest deployable unit — one or more containers |
| **Deployment** | Desired replica count + rolling updates for pods |
| **Service** | Stable network endpoint for a set of pods |
| **kubectl** | CLI that talks to the cluster API |

> **Why Kubernetes when Docker exists:** Docker runs a container but does not restart/reschedule it if it dies. Kubernetes maintains the desired state (self-healing, scaling, rolling updates).

### 10.2 Install and start Minikube

```bash
# Windows (PowerShell as admin)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe
.\minikube-installer.exe

# or a package manager
winget install Kubernetes.minikube
choco install minikube
```

```bash
minikube start --driver=docker      # or --driver=hyperv / virtualbox
minikube status
minikube dashboard                  # opens the Kubernetes dashboard
```

> **Why Minikube:** it creates a single-node cluster (usually inside a VM or Docker) so Kubernetes can be practised on a laptop.

Verify:

```bash
minikube version
kubectl version --client
minikube kubectl -- get pods -A
```

### 10.3 Create, run and scale a deployment

```bash
# Create a Deployment running Nginx
kubectl create deployment mynginx --image=nginx

# If it already exists, update the image
kubectl set image deployment/mynginx nginx=nginx:latest

# Inspect
kubectl get deployments
kubectl get pods
kubectl describe pods
```

```bash
# Expose it as a NodePort service
kubectl expose deployment mynginx --type=NodePort --port=80 --target-port=80

# Scale to 4 pods
kubectl scale deployment mynginx --replicas=4
kubectl get pods
kubectl get service mynginx
```

> **Why a Service:** pods get new IPs when recreated; a Service gives one stable virtual IP/DNS name and load-balances across the replicas.

### 10.4 Access the app

**Option A — port forwarding (simplest)**

```bash
kubectl port-forward svc/mynginx 8081:80
```

Open `http://localhost:8081`.

**Option B — Minikube tunnel**

```bash
minikube tunnel              # keep this terminal open
minikube service mynginx --url
```

Open the printed URL.

> **Why tunnel:** a NodePort is reachable on the node's IP, which on Windows/Docker may not be routable from the host; `minikube tunnel` bridges that.

### 10.5 Cleanup

```bash
kubectl delete deployment mynginx
kubectl delete service mynginx

minikube stop        # stop the cluster (state kept)
minikube delete      # delete the cluster completely
```

### 10.6 Nagios monitoring with Docker

```bash
docker pull jasonrivers/nagios:latest
docker run --name nagiosdemo -p 8888:80 jasonrivers/nagios:latest
```

Open `http://localhost:8888` → login `nagiosadmin` / `nagios`.

| Menu | What you see |
|---|---|
| **Hosts** | Machines being monitored (UP/DOWN) |
| **Services** | Checks such as CPU, disk, HTTP |
| **Alerts** | Recent notifications |
| **Host detail** | Status, metrics, reschedule check, disable notifications, schedule downtime |

```bash
docker ps
docker logs nagiosdemo
docker logs -f nagiosdemo          # live logs
docker stop nagiosdemo
docker rm nagiosdemo
docker rmi jasonrivers/nagios:latest
```

> **Why Nagios:** it polls hosts/services on a schedule, raises alerts when a check fails and notifies when it recovers — basic infrastructure monitoring.

### 10.7 AWS free-tier account (steps)

1. Open `https://aws.amazon.com` → **Create an AWS account**.
2. Enter email and account name → verify the email code.
3. Enter contact details → accept terms → **Create**.
4. Add billing details → verify (a small refundable charge is made).
5. Verify the phone number with the SMS code.
6. Choose the **Basic support plan** → **Complete sign up**.
7. Open the **AWS Management Console** → select role **Academic/Researcher**, interest **DevOps**.
8. Sign in as **root user** (secure it with MFA afterwards).

> **Why the free tier:** new accounts get limited free usage of services such as EC2 `t2.micro` for 12 months — enough for this lab.

**Gotchas**

- `kubectl` must point at the Minikube context: `kubectl config use-context minikube`.
- A `Pending` pod usually means insufficient CPU/RAM or an unschedulable node.
- `minikube stop` keeps the cluster; `minikube delete` removes it.
- Nagios container is not persistent by default — configuration changes vanish with `docker rm`.
- AWS free tier has limits; stop/terminate instances when done to avoid charges.

---

## Week 11: Jenkins CI/CD

**Goal:** Trigger Jenkins automatically from GitHub with a **webhook** (via ngrok) and send **email notifications** on build success/failure.

### 11.1 How the webhook flow works

```text
git push → GitHub webhook (HTTP POST) → ngrok public URL → local Jenkins
        → GitHub hook trigger → job starts automatically
```

### 11.2 Configure the GitHub webhook

1. GitHub repo → **Settings → Webhooks → Add webhook**.
2. **Payload URL**: `http://<jenkins-host>:8080/github-webhook/`
   - Local Jenkins: use the ngrok URL instead.
3. **Content type**: `application/json`.
4. **Which events**: *Just the push event*.
5. **Add webhook**.

> **Why:** a webhook is a push notification from GitHub; without it Jenkins must poll (slower, wasteful).

### 11.3 Expose local Jenkins with ngrok

```bash
ngrok config add-authtoken <your-token>
ngrok http 8080
```

Copy the generated HTTPS URL, e.g. `https://wintry-gloryingly-nam.ngrok-free.dev`, and use:

```text
https://<ngrok-id>.ngrok-free.dev/github-webhook/
```

> **Why ngrok:** GitHub cannot reach `localhost`; ngrok creates a public tunnel that forwards to port 8080. The `/github-webhook/` suffix is Jenkins' endpoint.

### 11.4 Enable the trigger in Jenkins

1. Open the job → **Configure**.
2. **Build Triggers** → check **GitHub hook trigger for GITScm polling**.
3. **Save**.

Test: push a commit → GitHub **Webhooks → Recent Deliveries** shows `200` → Jenkins build starts.

### 11.5 Gmail app password

1. `https://myaccount.google.com` → **Security → 2-Step Verification** → turn ON.
2. **Security → App passwords** → app **Other (Custom name)** → `Jenkins-Demo` → **Generate**.
3. Copy the 16-character password.

> **Why an app password:** Gmail blocks normal passwords for SMTP clients; the app password is a revocable credential just for Jenkins.

### 11.6 Install and configure email

**Plugin:** Manage Jenkins → Plugins → Available → **Email Extension Plugin** → Install.

**Manage Jenkins → Configure System**

| Field | Value |
|---|---|
| SMTP server | `smtp.gmail.com` |
| Use SMTP authentication | enabled |
| User name | your Gmail address |
| Password | the 16-digit app password |
| Use SSL | enabled |
| SMTP port | `465` |
| Reply-To | your Gmail address |
| Test configuration | send a test mail to yourself |

**Extended E-mail Notification** section: same SMTP host/port/SSL, add the Gmail credentials, default content type `text/html`.

### 11.7 Per-job notification

1. Job → **Configure → Post-build Actions**.
2. **Editable Email Notification**.
3. **Project Recipient List**: `you@example.com`.
4. **Triggers**: Failure, Success (and Unstable if needed).
5. Optional attachments: console log / test report.
6. **Save**.

```groovy
// Pipeline equivalent
post {
    success { mail to: 'team@example.com', subject: "OK: ${env.JOB_NAME}", body: 'Build passed' }
    failure { mail to: 'team@example.com', subject: "FAIL: ${env.JOB_NAME}", body: 'Build failed' }
}
```

> **Why:** notifications close the feedback loop — the team learns about a broken build without watching the dashboard.

**Gotchas**

- Webhook returns 404 → wrong URL; must end with `/github-webhook/`.
- Webhook returns 403/500 → Jenkins URL misconfigured or the job has no GitHub trigger.
- ngrok free URLs change on restart → update the webhook.
- Gmail "Less secure apps" is not the solution; use 2FA + app password.
- Port 465 = SSL; port 587 = STARTTLS. Do not mix them.

---

## Week 12: Ubuntu VM and Cloud Deployment

**Goal:** Launch an Ubuntu EC2 instance, install Docker, clone a Maven web app, containerise it and access it publicly through the instance's IP.

### 12.1 Launch the EC2 instance

| Step | Choice |
|---|---|
| 1. Name | `ubuntu` |
| 2. AMI | Ubuntu Server (**free tier eligible**) |
| 3. Architecture | 64-bit |
| 4. Instance type | `t2.micro` (1 vCPU, 1 GB RAM) |
| 5. Key pair | Create new → download `.pem` → store safely |
| 6. Network / Security group | Allow **SSH (22)**, **HTTP (80)**, **HTTPS (443)**; add app port later |
| 7. Storage | 8 GB (root volume) |
| 8. Launch | 1 instance |

> **Why a key pair:** AWS does not use passwords; the `.pem` private key proves your identity over SSH.

### 12.2 Connect over SSH

```bash
# From the folder containing the key
chmod 400 key.pem            # Linux/macOS
ssh -i key.pem ubuntu@<public-ip>
```

PowerShell/Git Bash on Windows work too; the key must be readable only by you.

> **Why:** `ubuntu` is the default user for Ubuntu AMIs; the public IP is on the EC2 console.

### 12.3 Install the software on the VM

```bash
sudo apt update
sudo apt install -y docker.io git nano
sudo systemctl enable --now docker
sudo usermod -aG docker $USER      # then log out/in, or use sudo for docker
```

> **Why:** the VM starts bare; Docker builds/runs the image, Git pulls the source, nano edits the Dockerfile.

### 12.4 Clone the app and write the Dockerfile

```bash
git clone https://github.com/<user>/<maven-web-app>.git
cd <maven-web-app>
nano Dockerfile
```

**Option A — deploy a pre-built WAR** (build it first):

```dockerfile
FROM tomcat:9-jdk17
COPY target/*.war /usr/local/tomcat/webapps/
EXPOSE 8080
```

**Option B — multi-stage build inside Docker** (no Maven on the VM):

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /src
COPY . .
RUN mvn clean package -DskipTests

FROM tomcat:9-jdk17
COPY --from=build /src/target/*.war /usr/local/tomcat/webapps/
EXPOSE 8080
```

> **Why:** Tomcat automatically deploys any `.war` placed in `webapps/`. The multi-stage build compiles inside a Maven image, so only the WAR is copied into the final image.

### 12.5 Build, run, open the port

```bash
docker build -t <imagename> .
docker images
docker run -d --name app-demo -p 6060:8080 <imagename>
docker ps
```

Open `http://<public-ip>:6060/<app-context>/`.

> **Why `-p 6060:8080`:** Tomcat listens on 8080 inside the container; 6060 is the host port. The security group must allow **6060** (or whichever host port you chose).

### 12.6 Security group (if the page does not load)

EC2 → instance → **Security → Security group → Edit inbound rules**:

| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |
| Custom TCP | 6060 (app port) | 0.0.0.0/0 |

> **Why:** a security group is a stateful firewall. If the port is not allowed, the browser times out even though the container is healthy.

### 12.7 Elastic IP (stable public address)

EC2 → **Elastic IPs → Allocate → Associate** with the instance.

> **Why:** a stopped/started instance may get a new public IP; an Elastic IP keeps the address stable.

### 12.8 User data (automate on boot)

```bash
#!/bin/bash
apt update -y
apt install -y docker.io git
systemctl enable --now docker
```

Paste into **Advanced details → User data** at launch.

> **Why:** user-data scripts run once on first boot, so a fresh instance is ready without manual setup.

**Gotchas**

- The app port must be open in the **security group**, not just published with `-p`.
- `docker build` needs the `.` build context — run it inside the project folder.
- A WAR's URL includes its context path (`/my-web-app/`), not just the IP.
- Stop or terminate the instance after the lab to avoid billing.
- `chmod 400 key.pem` prevents "UNPROTECTED PRIVATE KEY FILE" errors.

---

## Appendix: Command Index

### Git

| Command | Purpose |
|---|---|
| `git init` | Create a repository |
| `git clone <url>` | Copy a remote repository |
| `git remote -v` / `add` / `remove` / `rename` / `set-url` / `show` | Manage remotes |
| `git status` / `git diff` / `git diff --staged` | Inspect changes |
| `git add .` / `git commit -m` | Stage and snapshot |
| `git push -u origin main` | First push with upstream |
| `git fetch` / `git pull` | Download / download + merge |
| `git branch` / `switch -c` / `merge` / `rebase` | Branching |
| `git restore` / `restore --staged` | Discard / unstage |
| `git reset --soft/--hard` / `revert` / `commit --amend` | Undo |
| `git stash` / `stash apply` / `stash pop` | Shelve work |
| `git reflog` | Recover lost commits/branches |
| `git log --oneline --graph` / `show` / `blame` | History |
| `git format-patch -1 <hash>` / `git am` / `git apply` | Patches |
| `git branch -d/-D` / `push origin --delete` | Delete branches |
| `git remote prune origin` | Clean stale remote branches |
| `git filter-repo --path <file> --invert-paths` | Purge a file from history |

### Maven

| Command | Purpose |
|---|---|
| `mvn -v` | Version / Java info |
| `mvn archetype:generate ...` | Create a project |
| `mvn clean` | Delete `target/` |
| `mvn compile` / `test` / `package` / `install` | Lifecycle phases |
| `mvn clean install` | Full build |
| `mvn clean install -DskipTests` | Build without running tests |
| `mvn dependency:tree` | Show resolved dependency versions |
| `mvn dependency:resolve -U` | Force dependency download |
| `mvn site` | Generate project reports |
| `java -jar target/app.jar` | Run an executable JAR |
| `jar tf target/app.jar` | Inspect JAR contents |

### Docker

| Command | Purpose |
|---|---|
| `docker pull` / `push` / `login` / `logout` | Registry operations |
| `docker images` / `rmi` | List / remove images |
| `docker run -d --name x -p host:container image` | Start a container |
| `docker ps` / `ps -a` | Running / all containers |
| `docker stop` / `start` / `restart` / `rm` | Lifecycle |
| `docker exec -it x sh` | Shell inside |
| `docker logs -f x` / `inspect` / `stats` / `top` / `port` | Diagnose |
| `docker build -t name .` | Build an image |
| `docker commit <c> <image>` | Snapshot a container |
| `docker save` / `load` | Export / import a `.tar` image |
| `docker network create` / `connect` / `inspect` | Networking |
| `docker volume create` / `ls` / `rm` | Persistent storage |
| `docker system prune -a` | Clean unused objects |

### Docker Compose

| Command | Purpose |
|---|---|
| `docker compose up -d` | Start the stack |
| `docker compose up --build` | Rebuild then start |
| `docker compose ps` / `logs -f` | Status / logs |
| `docker compose restart` | Restart services |
| `docker compose down` / `down -v` | Remove stack (and volumes) |
| `docker compose up --scale web=2 -d` | Scale a service |
| `docker compose build <service>` | Rebuild one service |
| `docker compose exec <service> sh` | Shell into a service |

### Kubernetes / Minikube

| Command | Purpose |
|---|---|
| `minikube start --driver=docker` | Start the cluster |
| `minikube status` / `dashboard` / `stop` / `delete` | Cluster control |
| `minikube service <svc> --url` / `minikube tunnel` | Reach a service |
| `kubectl create deployment <name> --image=<img>` | Create a deployment |
| `kubectl get pods/deployments/services` | List resources |
| `kubectl describe pod <name>` | Events and state |
| `kubectl logs <name>` / `logs --previous` | Container logs |
| `kubectl exec -it <pod> -- sh` | Shell into a pod |
| `kubectl expose deployment <name> --type=NodePort --port=80` | Create a service |
| `kubectl scale deployment <name> --replicas=4` | Scale |
| `kubectl port-forward svc/<name> 8081:80` | Local port forward |
| `kubectl rollout undo deployment/<name>` | Roll back |
| `kubectl delete deployment/service <name>` | Cleanup |

### Jenkins / CI-CD

| Command / action | Purpose |
|---|---|
| `java -jar jenkins.war --httpPort=8080` | Run Jenkins |
| `http://localhost:8080` | Dashboard |
| Manage Jenkins → Tools | Configure JDK + Maven (`MAVEN_HOME`) |
| Manage Jenkins → Plugins | Install Git, Maven, Pipeline, Email Extension |
| New Item → Freestyle / Pipeline | Create a job |
| Build Triggers → GitHub hook trigger | Accept webhooks |
| `ngrok http 8080` | Public tunnel for webhooks |
| `ngrok config add-authtoken <token>` | Authenticate ngrok |
| Payload URL `.../github-webhook/` | GitHub webhook endpoint |
| Post-build → Editable Email Notification | Mail on success/failure |

### AWS EC2

| Command / action | Purpose |
|---|---|
| Launch Instance → Ubuntu AMI → `t2.micro` | Create the VM |
| Key pair `.pem` | SSH authentication |
| `ssh -i key.pem ubuntu@<ip>` | Connect |
| `sudo apt update && sudo apt install -y docker.io git` | Install tools |
| `docker build -t app .` / `docker run -d -p 6060:8080 app` | Deploy |
| Security group inbound rules | Open 22 / 80 / 443 / app port |
| Elastic IP | Stable public address |
| User data script | First-boot automation |

---

*End of manual. Scenario-based questions with explained answers: [`sbq.md`](sbq.md).*

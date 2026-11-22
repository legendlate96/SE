# Scenario-Based Questions — Answers and Explanations

**Companion to:** [`SE_Lab_Manual.md`](SE_Lab_Manual.md)
**Source:** `SE_KR24.pdf` — every scenario-based question from Weeks 2–12.
**How to use:** cover the answer, read the question, say the command out loud, then check the **Why** line. Question numbers follow the lab record; the one unnumbered Week 9 question is marked **Q15b**.

> **Golden rule for scenarios:** identify *which layer* the problem is in — working tree, staging area, local commits, remote, or the runtime (container/VM). The command family follows from that.

---

## Contents

| Section | Questions |
|---|---|
| [Week 2 — Basic Git](#week-2-basic-git-commands) | 25 |
| [Week 2 — Remote repositories](#week-2-remote-repository-commands) | 15 |
| [Week 3 — Collaboration and patches](#week-3-collaboration-conflicts-and-patches) | 15 |
| [Week 4 — Maven](#week-4-maven) | 19 |
| [Week 5 and 6 — Docker](#week-5-and-6-docker) | 15 |
| [Week 7 — Docker Compose](#week-7-docker-compose) | 10 |
| [Week 8 — Jenkins theory](#week-8-jenkins-theory-and-viva) | 15 |
| [Week 9 — Jenkins pipeline](#week-9-jenkins-pipeline-scenarios) | 24 |
| [Week 10 — Minikube and Nagios](#week-10-minikube-and-nagios) | 13 |
| [Week 11 — CI/CD viva](#week-11-cicd-theory-and-viva) | 10 |
| [Week 12 — EC2 deployment](#week-12-ec2-and-cloud-deployment) | 10 |
| [Pattern recognition](#pattern-recognition-cheat-sheet) | — |

---

## Week 2: Basic Git Commands

### Q1. You changed a file but have not staged it, and the change was a mistake. How do you discard it?

```bash
git restore file.txt
# older Git:
git checkout -- file.txt
```

**Why:** `git restore` copies the file from the index (which still matches the last commit) back into the working tree, so the bad edit disappears. Check with `git diff` first — this is destructive and cannot be undone by Git.

### Q2. You accidentally ran `git add file1.txt` but are not ready to commit. How do you unstage it without losing changes?

```bash
git restore --staged file1.txt
# older Git:
git reset HEAD file1.txt
```

**Why:** unstaging only moves the file out of the index; the working-tree content is untouched. `git status` then shows it as modified again.

### Q3. You made a commit with the wrong message and have not pushed it. How do you fix it?

```bash
git commit --amend -m "Corrected commit message"
```

**Why:** `--amend` replaces the last commit with a new one (new hash) that has the corrected message. Only do this on commits that are still local — amending a pushed commit rewrites shared history.

### Q4. You want a readable commit history of the current branch. Which command?

```bash
git log --oneline --graph --decorate
```

**Why:** `--oneline` gives one line per commit, `--graph` draws the branch/merge structure, `--decorate` shows branch and tag labels. Add `--all` to include every branch.

### Q5. After cloning, how do you set your name and email globally?

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --list
```

**Why:** `--global` writes to `~/.gitconfig`, so every repository on the machine uses the same author identity. Without this, commits may be attributed to the wrong person.

### Q6. You edited files but have not staged them. How do you view the changes?

```bash
git diff
git diff --stat          # summary: files and line counts
git diff --staged        # changes already staged
```

**Why:** plain `git diff` compares the working tree with the index, which is exactly the "not yet staged" set. `--staged` compares the index with the last commit.

### Q7. You are on `main` and need to switch to `feature/login`. What command?

```bash
git switch feature/login
# older Git:
git checkout feature/login
```

**Why:** switching updates the working tree to that branch's latest commit. If you have uncommitted changes that would be overwritten, Git refuses — commit or stash them first.

### Q8. You deleted `feature-ui` by mistake and have not pushed the deletion. How do you recover it?

```bash
git reflog
git branch feature-ui <commit-hash>
# or recreate and switch:
git switch -c feature-ui <commit-hash>
```

**Why:** a branch is only a pointer to a commit. `reflog` records where `HEAD` has been, so the commit still exists and the pointer can be recreated. If the branch also exists on the remote: `git fetch origin` then `git branch feature-ui origin/feature-ui`.

### Q9. You have local commits and want to upload them. What do you run?

```bash
git push origin main
# after the first push with -u:
git push
```

**Why:** push transfers commits that the remote does not have. It does not upload the working tree — only committed snapshots.

### Q10. How do you fetch remote changes without merging them automatically?

```bash
git fetch origin
git log HEAD..origin/main --oneline    # inspect what arrived
```

**Why:** `fetch` updates the remote-tracking branch `origin/main` only; your `main` and working tree stay unchanged. You can then merge or rebase deliberately.

### Q11. You are on `main` and want to start a feature called `search-filter`. What command?

```bash
git switch main
git pull origin main
git switch -c search-filter
```

**Why:** branch from an up-to-date `main` so the feature starts from the latest code. `-c` creates the branch and switches to it in one step.

### Q12. You committed a file containing an API key and want it completely removed from history. What do you do?

```bash
# recommended
pip install git-filter-repo
git filter-repo --path .env --invert-paths

# legacy
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all

git push --force --all
git push --force --tags
```

**Why:** deleting the file in a new commit leaves the old commits intact, so the key is still in history. A history rewrite removes it from every commit. **Rotate the key immediately** — assume it is compromised, and tell collaborators to re-clone because their hashes no longer match.

### Q13. You want to see all branches, local and remote. What command?

```bash
git branch -a        # all
git branch -r        # remote-tracking only
git branch -vv       # local + upstream + last commit
```

**Why:** `-a` combines local branches with `remotes/origin/*` tracking branches. Remote branches appear only after a `fetch`.

### Q14. You are on `main` and want to merge the completed `feature/signup` branch. What command?

```bash
git switch main
git pull origin main
git merge feature/signup
```

**Why:** `merge` integrates the feature's commits into the current branch. If `main` has not moved, Git fast-forwards; otherwise it creates a merge commit.

### Q15. A merge reported a conflict in `app.js`. What are the steps to resolve it?

```bash
git status                       # both modified: app.js
# open app.js; keep the correct code and delete the markers
git add app.js
git commit                       # completes the merge
# to cancel instead:
git merge --abort
```

**Why:** Git cannot decide between two edits to the same lines, so it writes both with `<<<<<<<`, `=======`, `>>>>>>>`. You choose the final content, stage the resolved file, and commit. `--abort` returns to the pre-merge state.

### Q16. You do not want Git to track `.log` files or `node_modules/`. How?

```bash
touch .gitignore
```

```gitignore
*.log
node_modules/
```

```bash
# if they were already tracked:
git rm -r --cached node_modules
git commit -m "Stop tracking node_modules"
```

**Why:** `.gitignore` only affects untracked files. Already-tracked files must be removed from the index (`--cached` keeps them on disk) and the removal committed.

### Q17. You want to know who last changed line 25 of `script.py`. What command?

```bash
git blame -L 25,25 script.py
git show <commit-hash>            # see the full change
```

**Why:** `blame` annotates each line with the commit, author and date that last touched it. `-L 25,25` limits the output to that line.

### Q18. You are mid-work and need to switch branches quickly. How do you save your work?

```bash
git stash push -m "wip: login form"
# or simply
git stash
```

**Why:** `stash` saves tracked modifications and restores a clean working tree, letting you switch branches. Use `-u` to include untracked files. Retrieve later with `git stash apply`/`pop`.

### Q19. You previously ran `git stash` and now want those changes back. What command?

```bash
git stash list
git stash apply        # restore, keep the stash entry
git stash pop          # restore and delete the entry
```

**Why:** `apply` is safer because the stash remains available if something goes wrong; `pop` applies and drops it.

### Q20. The `feature/test` branch is no longer needed. How do you delete it locally?

```bash
git branch -d feature/test
```

**Why:** `-d` deletes a branch that has already been merged. If Git says it is not fully merged, either merge it first or use `-D` if you are sure.

### Q21. You just merged `feature-ui` into `main` and want to clean up. How do you safely delete it?

```bash
git switch main
git branch --merged              # confirm feature-ui is listed
git branch -d feature-ui
```

**Why:** `--merged` proves the branch's commits are already in `main`; `-d` then deletes it safely. `-d` itself refuses to delete unmerged work, which is the safety net.

### Q22. `feature-experiment` was never merged but is no longer needed. How do you delete it?

```bash
git branch -D feature-experiment
```

**Why:** `-D` forces deletion regardless of merge status. The commits are not immediately lost — they remain in `reflog` for a while, so recovery is possible.

### Q23. You are on `feature-login` and want to delete `feature-ui`. What must you ensure first?

```bash
git switch main                 # or any branch other than feature-ui
git branch -d feature-ui        # or -D if unmerged
```

**Why:** Git cannot delete the branch you currently have checked out, because that would leave `HEAD` pointing at a non-existent branch. Switch away first.

### Q24. You want to delete `bugfix-footer` but are unsure whether it was merged. What do you check first?

```bash
git branch --merged             # merged into current branch
git branch --no-merged          # not merged
git log main..bugfix-footer     # commits not in main
```

**Why:** if it is merged, `git branch -d` is safe. If it is not, decide whether to merge it or accept losing the commits (`-D`). Checking first avoids accidental data loss.

### Q25. `feature-a`, `feature-b`, `feature-c` are all merged. How do you delete them in one command?

```bash
git branch -d feature-a feature-b feature-c
```

**Why:** `git branch -d` accepts multiple branch names. It still refuses any branch that is not merged, so the command fails safely rather than deleting unfinished work.

---

## Week 2: Remote Repository Commands

### Q1. You are starting work and need a local copy of a remote repository.

```bash
git clone https://github.com/<user>/<repo>.git
cd <repo>
```

**Why:** `clone` downloads the full history, creates `origin`, and checks out the default branch. It is `init` + `remote add` + `fetch` + `checkout` in one step.

### Q2. You want to see the remotes connected to your local repository.

```bash
git remote -v
```

**Why:** lists each remote name with its fetch and push URLs. No output means the repository has no remote configured.

### Q3. You want to add a new remote repository.

```bash
git remote add origin https://github.com/<user>/<repo>.git
git remote -v
```

**Why:** `origin` is only a conventional name; it is the default handle for the main remote. You can add more (e.g. `upstream` for a fork).

### Q4. You want to remove a remote from your local configuration.

```bash
git remote remove origin
```

**Why:** deletes the remote entry and its tracking branches locally. It does **not** delete anything on GitHub, and local commits are unaffected.

### Q5. You want to rename an existing remote.

```bash
git remote rename origin upstream
```

**Why:** renames the handle and updates the tracking configuration. Useful after forking, when `origin` should point to your fork and `upstream` to the original repo.

### Q6. You want to fetch updates without merging them.

```bash
git fetch origin
```

**Why:** downloads new commits and updates `origin/*` only. Your branch, index and working tree stay as they were; you can review before integrating.

### Q7. You want to pull the latest changes and merge them into your branch.

```bash
git pull origin main
# equivalent to:
git fetch origin
git merge origin/main
```

**Why:** `pull` is fetch + merge. It updates the current branch with the remote's commits and may create a merge commit or fast-forward.

### Q8. You want to push your local commits to the remote.

```bash
git push origin main
```

**Why:** uploads commits reachable from `main` that the remote does not have. A non-fast-forward rejection means the remote has commits you have not integrated yet.

### Q9. You are pushing for the first time and want to set the upstream branch.

```bash
git push --set-upstream origin main
# short form
git push -u origin main
```

**Why:** `-u` records `origin/main` as the upstream of local `main`, so later `git push` and `git pull` need no arguments. It also enables `git status` to show ahead/behind counts.

### Q10. You want to change a remote's URL.

```bash
git remote set-url origin https://github.com/<user>/<new-repo>.git
git remote -v
```

**Why:** updates the stored URL without removing and re-adding the remote, so tracking branches and configuration are preserved. Common after renaming a repository or switching HTTPS ↔ SSH.

### Q11. You want to list all branches on the remote.

```bash
git branch -r
# or, with more detail:
git ls-remote --heads origin
```

**Why:** `-r` lists remote-tracking branches (`origin/main`, `origin/dev`). They are refreshed by `git fetch`; `ls-remote` queries the server directly.

### Q12. A branch was deleted on the remote but still appears locally. What command fixes it?

```bash
git remote prune origin
# or prune while fetching:
git fetch --prune
```

**Why:** deleting a remote branch does not automatically remove your local `origin/<branch>` reference. Pruning removes those stale tracking branches. It does not touch your local branches.

### Q13. You want to fetch one specific branch from a remote.

```bash
git fetch origin feature-branch
# then inspect or switch:
git switch feature-branch
```

**Why:** fetches only that branch's commits into `origin/feature-branch`, which is faster than fetching everything on a large repository.

### Q14. You want detailed information about a remote repository.

```bash
git remote show origin
```

**Why:** shows the fetch/push URLs, `HEAD` branch, remote branches, and which local branches track which remote branches, plus branches to pull/push.

### Q15. You want to rebase your local branch onto a remote branch to keep history linear.

```bash
git fetch origin
git rebase origin/main
```

**Why:** rebase replays your local commits on top of the latest `origin/main`, producing a straight line instead of a merge commit. Never rebase commits that others have already pulled.

---

## Week 3: Collaboration, Conflicts and Patches

### Q1. Push is rejected with "non-fast-forward". How do you resolve it?

```bash
git pull --rebase origin main
# resolve any conflicts, then:
git rebase --continue
git push origin main
```

**Why:** the remote has commits you do not have, so Git refuses to overwrite them. Pull integrates them first; `--rebase` keeps history linear. Do **not** use `git push --force` on a shared branch — that discards teammates' commits.

### Q2. You have a feature branch and the remote already has `main`. How do you push without affecting `main`?

```bash
git push -u origin feature/my-feature
```

**Why:** a push updates only the branch you name. `feature/my-feature` becomes a new remote branch; `main` is untouched until a pull request is merged.

### Q3. The remote structure changed and new branches were added. How do you update your local copy?

```bash
git fetch --all --prune
git branch -a
git pull origin main
```

**Why:** `fetch --all` downloads every remote's updates, `--prune` removes deleted tracking branches, and `git branch -a` shows the new branches. `pull` updates your current branch.

### Q4. A colleague pushed to `main` and you have local changes on the same branch. How do you pull without conflicts?

```bash
git stash push -m "my local work"
git pull --rebase origin main
git stash pop
# resolve any conflicts, then commit
```

**Why:** stashing gives you a clean tree so the pull cannot collide with uncommitted edits. Rebasing puts your work on top of the colleague's commits; `stash pop` reapplies it, and any overlap is resolved once.

### Q5. You accidentally pushed a sensitive file (API keys). How do you fix it?

```bash
pip install git-filter-repo
git filter-repo --path .env --invert-paths
git push --force --all
git push --force --tags
```

Then: **revoke/rotate the key**, add the file to `.gitignore`, and tell the team to re-clone.

**Why:** the secret is in the commit history and possibly in forks/caches, so deleting it in a new commit is not enough. Rewriting history plus force-push removes it from the branch; rotation is mandatory because the old key may already be copied.

### Q6. Your manager wants the latest `main` integrated into your feature branch. What steps?

```bash
git switch feature/my-feature
git fetch origin
git merge origin/main
# or, for linear history:
git rebase origin/main
```

**Why:** merging brings `main`'s commits into the feature branch and produces a merge commit; rebasing replays your feature commits on top of `main`. Either way, test after integrating.

### Q7. You need to push to a different remote repository. How do you configure it?

```bash
git remote add neworigin https://github.com/<user>/<new-repo>.git
git push -u neworigin main
# or repoint the existing remote:
git remote set-url origin https://github.com/<user>/<new-repo>.git
```

**Why:** a repository can have several remotes. `neworigin` keeps the old one intact; `set-url` replaces the destination of `origin`. Use `-u` to track the new upstream.

### Q8. After `git pull`, your branch is behind the remote. How do you catch up without losing local changes?

```bash
git stash push -m "local work"
git pull --rebase origin main
git stash pop
```

**Why:** the stash protects uncommitted work; rebase replays it on the newest remote state. If the branch is merely behind and you have no local commits, `git pull --ff-only` is the cleanest option.

### Q9. Your local changes conflict with work pushed by others. How do you resolve it?

```bash
git fetch origin
git merge origin/main          # or git rebase origin/main
git status                     # list conflicted files
# edit each file, remove <<<<<<< ======= >>>>>>> markers
git add <files>
git commit                     # for merge; git rebase --continue for rebase
git push origin <branch>
```

**Why:** a conflict is Git asking which version to keep. You resolve it by editing the file to the intended final content, staging it, and completing the merge/rebase. Test before pushing.

### Q10. You pushed a feature branch but now need to delete it from the remote.

```bash
git push origin --delete feature/my-feature
# or
git push origin :feature/my-feature
```

**Why:** the first form is explicit; the second is the older refspec shorthand. Deleting the remote branch does not delete your local branch — use `git branch -d` for that.

### Scenario: team of four, `feature/ui-update`, upstream changes, and an emailed patch

### Q11. Which command brings local `main` up to date with the remote before merging it into your feature branch?

```bash
git switch main
git pull origin main
```

**Why:** you want the newest `main` locally first, so the merge into your feature includes the teammate's critical bug fix. `git fetch origin` + `git merge origin/main` is the explicit equivalent.

### Q12. How do you update `feature/ui-update` with the latest `main`?

```bash
git switch feature/ui-update
git merge main
# or keep history linear:
git rebase main
```

**Why:** merging `main` into the feature branch incorporates the bug fix and surfaces conflicts in your branch — the safest place to fix them, before the feature goes back to `main`.

### Q13. Which command retries the push, and what if it fails due to conflicts?

```bash
git push origin feature/ui-update
```

If rejected:

```bash
git pull --rebase origin feature/ui-update
# resolve conflicts
git add <files>
git rebase --continue
git push origin feature/ui-update
```

**Why:** the rejection means the remote branch has commits you lack. Integrate them, resolve conflicts locally, then push again. Avoid `--force` on a shared branch.

### Q14. How do you apply a `.patch` file from a teammate and include it in history?

```bash
git am 0001-fix-css.patch
# if it is a plain diff rather than a format-patch file:
git apply fix-css.patch
git add .
git commit -m "Apply teammate CSS fix"
```

**Why:** `git am` reads a mailbox-format patch and creates a commit with the original author and message. `git apply` only changes the working tree, so you must commit manually. Check first with `git apply --check`.

### Q15. After testing, how do you merge the feature into `main` and push?

```bash
git switch main
git pull origin main
git merge feature/ui-update
# run the tests
git push origin main
```

**Why:** merge into an up-to-date `main`, verify the combined result, then publish. If the team uses pull requests, push the feature branch and merge the PR on GitHub instead.

---

## Week 4: Maven

### Q1. The build fails with `maven-compiler-plugin ... compile` and `[Help 1]`. How do you resolve it?

```bash
mvn -e -X clean compile
mvn -v
```

**Why:** `[Help 1]` is only the footer — the real cause is the first `ERROR` line above it. Common causes: wrong `source`/`target` Java version, `JAVA_HOME` pointing to an older JDK, or a genuine syntax/type error. `-e` shows the stack trace, `-X` shows debug details.

Fix the compiler configuration:

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

### Q2. A dependency is not recognized by the compiler. How do you confirm it is in `.m2` and in the dependency tree?

```bash
mvn dependency:tree
mvn dependency:resolve -U
ls ~/.m2/repository/<group-path>/<artifact>/
mvn help:effective-pom
```

**Why:** `dependency:tree` shows whether the dependency was resolved and which version won; `-U` forces a re-download of snapshots/metadata. If the artifact is missing from `~/.m2`, the coordinates or repository are wrong.

### Q3. A teammate sends a `.patch` for a bug fix. How do you apply it and include it in the Maven build?

```bash
git am 0001-bugfix.patch        # creates a commit
# or
git apply bugfix.patch && git add . && git commit -m "Apply bugfix"
mvn clean install
```

**Why:** the patch changes source files, so it must be applied before Maven compiles. `git am` preserves authorship; then the normal build validates it.

### Q4. You have multiple JUnit failures and want to rerun only the failed tests.

```bash
mvn test -Dtest=AppTest#testSum
mvn test -Dtest='!PassingTest'
mvn surefire:test -Dtest=FailedTest
```

**Why:** Surefire accepts a `-Dtest=` filter (class, method, or exclusion with `!`). Alternatively, read `target/surefire-reports/*.txt` to identify the failing classes and rerun just those. Some teams add `-Dsurefire.rerunFailingTestsCount=2` for flaky tests.

### Q5. The build fails with "Unsupported class version error". Which plugin/configuration do you review?

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
  <configuration>
    <release>17</release>
  </configuration>
</plugin>
```

```bash
mvn -v            # check the JDK Maven is using
echo $JAVA_HOME   # Linux/Git Bash
```

**Why:** the class file was compiled for a newer Java than the runtime JVM. Align the compiler `release` (or `source`/`target`) with the JVM that runs the app, and make sure `JAVA_HOME` points to that JDK.

### Q6. You need to change a Java application from a WAR to a standalone JAR. What changes in `pom.xml`?

```xml
<packaging>jar</packaging>

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

**Why:** `packaging` selects the artifact type. A JAR needs a `Main-Class` manifest to be runnable with `java -jar`; remove web-only pieces (`webapp/`, servlet dependencies, `web.xml`). For dependencies inside the JAR, add the Shade or Assembly plugin.

### Q7. Change the default build output directory from `target/` to `build_output/`.

```xml
<build>
  <directory>build_output</directory>
</build>
```

**Why:** `<directory>` overrides the property Maven uses for all generated output. Run `mvn clean package` and the JAR/WAR appears under `build_output/`.

### Q8. You want to skip tests during the build. What command?

```bash
mvn clean install -DskipTests
# skip compiling the tests too:
mvn clean install -Dmaven.test.skip=true
```

**Why:** `-DskipTests` compiles test sources but does not run them; `-Dmaven.test.skip=true` skips both compilation and execution. Use the first when you still want compile-time validation.

### Q9. How do you generate a site report with test coverage and dependency analysis?

```bash
mvn site
# often combined with:
mvn clean verify site
```

Add reporting/plugins such as JaCoCo (coverage), `maven-surefire-report-plugin`, and `maven-dependency-plugin`. The HTML output is in `target/site/index.html`.

**Why:** the `site` lifecycle aggregates plugin reports into a browsable HTML site; coverage needs JaCoCo to instrument the tests.

### Q10. How do you build a Java project with Maven, and what appears in `target/` after `mvn clean install`?

```bash
mvn clean install
```

| Output | Contents |
|---|---|
| `target/classes/` | Compiled main classes and resources |
| `target/test-classes/` | Compiled test classes |
| `target/*.jar` | The packaged artifact |
| `target/surefire-reports/` | Test results |
| `target/maven-archiver/` | Build metadata |
| `target/maven-status/` | Compiler input/output tracking |

**Why:** `clean` empties `target/`, then each lifecycle phase writes its output; `install` also copies the artifact to `~/.m2/repository`.

### Q11. How does Maven resolve dependency conflicts, and how do you view/manage the tree?

```bash
mvn dependency:tree
mvn dependency:tree -Dverbose
```

**Why:** Maven applies **nearest-wins** (the version closest to the project in the dependency graph) and **first-declared-wins** at the same depth. Use `<exclusions>` to drop a transitive version, or `<dependencyManagement>` to pin one version across the project.

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>lib-a</artifactId>
  <version>1.0</version>
  <exclusions>
    <exclusion>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

### Q12. How do you write and run a JUnit test, and where are the compiled classes and reports?

```java
// src/test/java/com/example/AppTest.java
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

**Why:** Surefire runs tests during the `test` phase. Compiled tests go to `target/test-classes/`; results go to `target/surefire-reports/` as `.txt` and `.xml`. A failing assertion fails the build.

### Q13. How do you create an executable JAR with a main method, and which plugin helps?

```xml
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
```

```bash
mvn package
java -jar target/my-app-1.0-SNAPSHOT.jar
```

**Why:** the JAR plugin writes `Main-Class` into `META-INF/MANIFEST.MF`. If the app has dependencies, use the Shade plugin to produce a fat JAR, otherwise `java -jar` fails with `ClassNotFoundException`.

### Q14. How do you install and use a custom third-party JAR, and confirm it is on the classpath?

```bash
mvn install:install-file \
  -Dfile=lib/custom.jar \
  -DgroupId=com.vendor \
  -DartifactId=custom-lib \
  -Dversion=1.0 \
  -Dpackaging=jar
```

```xml
<dependency>
  <groupId>com.vendor</groupId>
  <artifactId>custom-lib</artifactId>
  <version>1.0</version>
</dependency>
```

```bash
mvn dependency:tree | findstr custom-lib
mvn clean package
```

**Why:** `install:install-file` puts the JAR into the local `.m2` repository with proper coordinates, so Maven can resolve it like any other dependency. `dependency:tree` confirms it is part of the build.

### Q15. How do you create a Maven web project that packages into a WAR, and what is the standard structure?

```bash
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-web-app \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DinteractiveMode=false
```

```text
my-web-app/
├── pom.xml                 <packaging>war</packaging>
└── src/main/webapp/
    ├── index.jsp
    └── WEB-INF/web.xml
```

**Why:** the `maven-archetype-webapp` generates the web layout; `war` packaging tells Maven to assemble `webapp/` plus compiled classes into a `.war`.

### Q16. What command builds a WAR, where is it generated, and how do you deploy it to Tomcat?

```bash
mvn clean package
# target/my-web-app.war

# copy to Tomcat
cp target/my-web-app.war $CATALINA_HOME/webapps/
```

**Why:** Tomcat watches `webapps/` and auto-deploys any WAR placed there under a context path equal to the WAR name. Alternatively use the Tomcat Maven plugin (`mvn tomcat7:deploy`) or the Jenkins "Deploy to container" step.

### Q17. How do you add JSTL and servlet-api dependencies, and why is servlet-api `provided`?

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```

**Why:** the servlet API is implemented by Tomcat at runtime. `provided` makes it available at compile time but excludes it from the WAR, avoiding duplicate classes and version clashes with the container. JSTL is a library the app itself needs, so it is packaged normally.

### Q18. How do you set up a multi-module Maven web project with `core` and `web`, and how are they built/connected?

```xml
<!-- parent/pom.xml -->
<packaging>pom</packaging>
<modules>
  <module>core</module>
  <module>web</module>
</modules>
```

```xml
<!-- web/pom.xml -->
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

**Why:** the parent aggregates the modules and centralises versions/plugins. Maven builds them in dependency order (core first), and `web` consumes `core` from the reactor/local repository. `web` has `<packaging>war</packaging>`; `core` is a JAR.

### Q19. How does a Maven web project differ from a traditional WAR-based application in packaging and execution?

| | Traditional WAR | Maven web project |
|---|---|---|
| Packaging | Manual `war`/IDE export | `mvn package` → `target/*.war` |
| Structure | IDE-defined | Standard `src/main/webapp` |
| Dependencies | Copied into `WEB-INF/lib` by hand | Resolved by Maven, packaged automatically |
| Execution | Deploy to Tomcat/JBoss | Same WAR, but reproducible build; can also run embedded Tomcat/Jetty |

**Why:** the WAR format is the same; Maven only standardises how it is produced. A JAR is executed by the JVM (`java -jar`), while a WAR is executed by a servlet container (Tomcat).

## Week 5 and 6: Docker

### Q1. The app runs in Docker but you are not sure the container is active. Which command checks running containers?

```bash
docker ps
docker ps -a                          # include stopped containers
docker inspect -f '{{.State.Status}}' web_app
```

**Why:** `docker ps` lists only running containers; `-a` also shows exited ones. `inspect` gives the exact state (`running`, `exited`, `restarting`) for one container.

### Q2. A container is consuming high CPU. How do you stop `web_app`?

```bash
docker stats                          # identify the culprit
docker stop web_app                   # graceful SIGTERM, then SIGKILL after timeout
docker kill web_app                   # immediate, if it ignores stop
```

**Why:** `stop` asks the process to shut down cleanly; `kill` forces it. `docker stats` confirms which container is actually using the CPU.

### Q3. You wrote a Dockerfile and want to build an image named `myapi`. What command?

```bash
docker build -t myapi .
```

**Why:** `-t` names/tags the resulting image; the final `.` is the build context sent to the daemon. The Dockerfile must be in that context (or use `-f path/to/Dockerfile`).

### Q4. Run `web_app` and expose container port 5000 on host port 8080.

```bash
docker run -d --name web_app -p 8080:5000 web_app
```

**Why:** the mapping is `host:container`, so the browser uses `http://localhost:8080` while the app keeps listening on 5000 inside the container. `-d` runs it in the background.

### Q5. The container is running but you want a shell inside it to debug. What command?

```bash
docker exec -it web_app sh
# if bash exists:
docker exec -it web_app bash
```

**Why:** `exec` starts a new process in a running container; `-it` keeps stdin open and allocates a TTY. `sh` is used because slim/Alpine images often lack bash.

### Q6. You built test images and want to delete `old_api`. How?

```bash
docker ps -a                          # find containers using it
docker stop <container> && docker rm <container>
docker rmi old_api
```

**Why:** an image cannot be removed while a container (even a stopped one) references it. Remove the dependants first, or use `docker rmi -f` with care.

### Q7. The app crashed inside the container. How do you check the logs?

```bash
docker logs web_app
docker logs -f --tail 100 web_app
docker logs --since 10m web_app
```

**Why:** `logs` shows stdout/stderr captured by Docker. `-f` follows live output, `--tail` limits history, `--since` filters by time.

### Q8. You want to start a container in the background. Which flag?

```bash
docker run -d --name web_app -p 8080:5000 web_app
```

**Why:** `-d` (detach) returns the terminal immediately and keeps the container running. Without it, the container runs in the foreground and Ctrl+C stops it.

### Q9. You are not sure which container is using port 3000. How do you check?

```bash
docker ps
docker port <container>
docker inspect -f '{{.NetworkSettings.Ports}}' <container>

# host side
netstat -ano | findstr :3000          # Windows
ss -tulpn | grep :3000                # Linux
```

**Why:** `docker ps` shows published mappings; `docker port`/`inspect` show the container side. The host command catches non-Docker processes too.

### Q10. You built an image but forgot to tag it. How do you tag it now?

```bash
docker images                         # find the IMAGE ID
docker tag <image-id> <dockerhub-user>/myapi:1.0
```

**Why:** `tag` creates a new name pointing at the same image ID. The repository name must match your Docker Hub account before `docker push`.

### Q11. You want to export an image to a `.tar` file for offline transfer.

```bash
docker save -o myapi.tar <dockerhub-user>/myapi:1.0
# on the other machine:
docker load -i myapi.tar
```

**Why:** `save` archives the image layers and metadata; `load` restores them. This works without a registry. (`docker export` is for containers, not images.)

### Q12. You want a container to restart automatically if it crashes. What do you use?

```bash
docker run -d --restart unless-stopped --name web_app web_app
# change an existing container:
docker update --restart=always web_app
```

**Why:** the restart policy is enforced by the Docker daemon. `always` also restarts after a daemon reboot; `unless-stopped` respects a manual stop; `on-failure` only restarts on a non-zero exit.

### Q13. How do you see CPU/RAM per container, and how do you limit memory?

```bash
docker stats
docker stats --no-stream

docker run -d --name limited --memory=256m --cpus=1 nginx
docker update --memory=512m --memory-swap=512m limited
docker inspect -f '{{.HostConfig.Memory}}' limited
```

**Why:** `stats` reads the cgroup counters live. `--memory` caps RAM; exceeding it makes the kernel OOM-kill the process. `--cpus` limits CPU shares/quota. `--memory-swap` controls swap in addition to RAM.

### Q14. You want the container to run a shell command before starting the app. What goes in the Dockerfile?

```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl     # build-time preparation
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]                     # runtime preparation
CMD ["python", "app.py"]
```

**Why:** `RUN` executes at **build** time and becomes an image layer. `ENTRYPOINT`/`CMD` execute at **container start**. To run a setup command at start and then the app, use an entrypoint script that ends with `exec "$@"`.

### Q15. You want a custom Nginx image with your static files. What do you do?

```dockerfile
FROM nginx:alpine
COPY ./site /usr/share/nginx/html
EXPOSE 80
```

```bash
docker build -t my-site .
docker run -d --name my-site -p 8080:80 my-site
```

**Why:** Nginx serves `/usr/share/nginx/html` by default, so copying the files there replaces the default page. `EXPOSE` documents the port; `-p` actually publishes it.

---

## Week 7: Docker Compose

### Q1. Should two applications (Node.js backend and a Python jobs script) run in the same container?

**No.** Use two services.

```yaml
services:
  backend:
    image: node:alpine
    ports:
      - "3000:3000"
  jobs:
    image: python:alpine
    depends_on:
      - backend
```

```bash
docker compose up -d
```

**Why:** a container should have one primary process (PID 1). Two apps in one container share logs, lifecycle and scaling, and one crashing can take down the other. Separate services can be scaled, updated and restarted independently.

### Q2. Run a Flask API on 5000 and Nginx on 80, both on the same host.

```yaml
services:
  api:
    build: ./api
    ports:
      - "5000:5000"
  web:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - api
```

**Why:** each service publishes a different **host** port. Inside the Compose network they reach each other by service name (`api:5000`), which is how Nginx can proxy to Flask.

### Q3. Run a React frontend, Express backend and MongoDB together.

```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      MONGO_URL: mongodb://mongo:27017/app
    depends_on:
      - mongo
  mongo:
    image: mongo:7
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

```bash
docker compose up -d --build
```

**Why:** Compose creates one network, so `backend` resolves `mongo` by name. The named volume keeps database data across restarts. `depends_on` sets start order (add healthchecks for readiness).

### Q4. Two containers both try to expose host port 8080. What happens?

The second container fails to start with `port is already allocated` / `bind: address already in use`.

```bash
# change the host side of one mapping
docker run -d -p 8081:8080 app2
```

**Why:** a host port can be bound by only one process at a time. The container port may be the same; only the host side must differ. A reverse proxy (Nginx/Traefik) can route different hostnames to both on 80/443.

### Q5. You updated code and want to restart all services quickly. Easiest way?

```bash
docker compose up -d
# or, if only a restart is needed:
docker compose restart
```

**Why:** `up -d` reconciles the stack with the YAML — it recreates only changed services. `restart` simply restarts existing containers without applying config/image changes.

### Q6. The frontend has a new release, but backend/DB must keep running. How do you update only the frontend?

```bash
docker compose build frontend
docker compose up -d --no-deps frontend
# short form:
docker compose up -d --build frontend
```

**Why:** naming the service limits the operation to it. `--no-deps` prevents Compose from touching its dependencies; the other containers keep running.

### Q7. Tomcat listens on 8080 but the browser says "Connection refused". Why?

Check, in order:

```bash
docker ps                              # is the container running?
docker port <container>                # is 8080 published?
docker logs <container>                # did Tomcat start?
```

Common causes: the port was not published (`-p 8080:8080` missing), the app is not deployed (no WAR in `webapps/`), Tomcat is still starting, or a firewall/security group blocks the port. "Connection refused" means nothing is listening on the host port — unlike a timeout, which suggests a firewall.

### Q8. You are not sure which container is using port 3000. How do you check?

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}'
docker port <container>
netstat -ano | findstr :3000           # Windows
ss -tulpn | grep :3000                 # Linux
```

**Why:** `docker ps` shows the host→container mappings; `netstat`/`ss` show the owning process on the host, which also catches non-Docker services.

### Q9. How do you stop and clean up a running Tomcat container and its image?

```bash
docker stop tomcat
docker rm tomcat
docker rmi tomcat:9
```

**Why:** stop first (graceful), then remove the container, then the image. If other containers use the image, remove them too or the `rmi` fails.

### Q10. You want to share your app with a teammate. What do you do with the image?

```bash
docker tag myapp <dockerhub-user>/myapp:1.0
docker login
docker push <dockerhub-user>/myapp:1.0
```

Teammate:

```bash
docker pull <dockerhub-user>/myapp:1.0
docker run -d -p 8080:8080 <dockerhub-user>/myapp:1.0
```

**Why:** a registry is the standard distribution channel. For offline sharing, `docker save`/`load` a `.tar` instead.

---

## Week 8: Jenkins Theory and Viva

### Q1. What is Jenkins primarily used for?

**Continuous Integration and Continuous Delivery/Deployment (CI/CD).** It automatically builds, tests and deploys code whenever a change is pushed, so integration problems are found early.

### Q2. What are the key features of Jenkins?

Open source and highly extensible (1,800+ plugins), pipeline-as-code (Jenkinsfile), distributed builds via agents, easy web UI, support for many SCM/build tools, and scheduled or event-driven triggers.

### Q3. What is the default port of Jenkins?

**8080** (`http://localhost:8080`). It can be changed with `--httpPort=<port>` or in `jenkins.xml`.

### Q4. What can be integrated with Jenkins for version control?

Git, GitHub, GitLab, Bitbucket, Subversion (SVN), Mercurial — through their respective plugins.

### Q5. What is the purpose of Jenkins plugins?

Plugins extend Jenkins with new capabilities: SCM integration, build tools (Maven/Gradle), pipeline steps, Docker/Kubernetes support, notifications, credentials, and UI enhancements.

### Q6. Which job type suits one-off tasks or small scripts?

A **Freestyle project** — a simple job with build steps, no pipeline syntax required.

### Q7. How do you manage sensitive information such as API keys in Jenkins?

Use the **Credentials Manager** (`Manage Jenkins → Credentials`). Store secrets there, reference them by ID in jobs/pipelines (`credentialsId`, `withCredentials`); Jenkins masks them in logs. Never hard-code them in a Jenkinsfile.

### Q8. What is Blue Ocean in Jenkins?

A modern visual interface for Jenkins pipelines — it shows stages, branches and pull requests graphically, making pipeline status easier to read than the classic UI.

### Q9. What does the Blue Ocean feature refer to? (repeat)

It provides a visual interface for creating and monitoring pipelines (stage view, logs per step, GitHub/Bitbucket integration).

### Q10. Which Jenkins component allows distributed builds across multiple machines?

**Agents (nodes).** The controller schedules jobs; agents execute them. Labels route jobs to agents with the required tools.

### Q11. Five important plugins for a microservices CI/CD pipeline.

| Plugin | Purpose |
|---|---|
| **Git** | Clone/poll Git repositories |
| **Pipeline** | Define builds as code (Jenkinsfile) |
| **Docker** | Build/push/run container images |
| **Maven Integration** | Run Maven goals as build steps |
| **GitHub** | Webhooks, PR status, repository integration |
| *(extra)* Kubernetes | Dynamic agents as pods |
| *(extra)* Blue Ocean | Visual pipeline UI |

### Q12. Steps to install a plugin via the UI, and compatibility considerations.

1. `Manage Jenkins → Plugins → Available plugins`.
2. Search the plugin name.
3. Select it → **Install**.
4. Restart Jenkins if required.

Considerations: the plugin version must match the Jenkins core version, dependencies are installed automatically (check for conflicts), and updates should be tested — a bad update can break jobs. Read the plugin's changelog and keep Jenkins itself reasonably current.

### Q13. Same as Q12 (repeat).

`Manage Jenkins → Plugins → Available` → search → select → install → restart if asked. Verify compatibility with the Jenkins version and existing plugins before updating in production.

### Q14. After installing the Git plugin, how do you configure it for a pipeline?

1. `Manage Jenkins → Tools → Git` — verify the Git executable (or let Jenkins install it).
2. `Manage Jenkins → Credentials` — add a username/token or SSH key.
3. In the job/pipeline, set the repository URL and `credentialsId`.
4. For webhooks, install the GitHub plugin and configure the hook trigger.

```groovy
git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/user/repo.git'
```

### Q15. Common plugin problems and troubleshooting.

| Problem | Symptom | Fix |
|---|---|---|
| Dependency conflict | Plugin fails to load, `NoSuchMethodError` | Update Jenkins/plugins, check the dependency tree |
| Version incompatibility | "Plugin requires Jenkins X or newer" | Upgrade core or downgrade the plugin |
| Missing dependency | Plugin appears but steps are absent | Install the required dependency plugin |
| Corrupt update | Jenkins fails to start | Restore from backup, remove the plugin directory, restart |

Troubleshoot with `Manage Jenkins → System Log`, `jenkins.log`, and the plugin manager's error messages. Restart after changes.

---

## Week 9: Jenkins Pipeline Scenarios

### Q1. The manager wants the workspace cleaned before building. Which stage handles it?

The **`Checkout & Clean`** stage — it runs `mvn clean` (or `cleanWs()`).

```groovy
stage('Checkout & Clean') {
    steps {
        cleanWs()
        bat 'mvn clean -f mavenjava'
    }
}
```

**Why:** `clean` deletes `target/`, so stale classes cannot mask a compile error.

### Q2. The GitHub repository link is missing. Where exactly do you provide it?

In the checkout/clone step, or in the `scm` block:

```groovy
git branch: 'main', url: 'https://github.com/<user>/<repo>.git'
```

**Why:** the `git` step is a pipeline shortcut for checkout; "Pipeline script from SCM" stores the URL in the job configuration instead.

### Q3. Maven is not configured globally in Jenkins. Which part fails first?

The **`tools` block**:

```groovy
tools { maven 'MAVEN_HOME' }
```

**Why:** Jenkins resolves tool installations before running any stage. If the name does not exist under `Manage Jenkins → Tools`, the pipeline aborts immediately with "No tool named MAVEN_HOME found".

### Q4. The pipeline is not creating `.war` files. Which stage is responsible?

The **Package** stage (`mvn package`). Check that `pom.xml` has `<packaging>war</packaging>` and that the stage actually runs (not skipped after a failure).

### Q5. Tests failed but the pipeline continued. How does Jenkins behave in the test stage?

By default, `mvn test` returns a non-zero exit code → the stage **FAILS** and later stages are skipped. If the pipeline continued, something suppressed the failure:

```groovy
// mark the build unstable but continue
catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
    bat 'mvn test'
}
```

Other causes: `mvn test -Dmaven.test.failure.ignore=true`, `|| true`, or a `try/catch`. To make tests stop the pipeline, call `mvn test` directly in `steps` with no wrapper.

### Q6. You want the pipeline to trigger only when GitHub changes, not nightly. Where do you configure it?

In the `triggers` block, with a webhook:

```groovy
triggers {
    githubPush()
    // or pollSCM('H/5 * * * *')
}
```

Plus GitHub repo → Settings → Webhooks → `.../github-webhook/`.

**Why:** `githubPush()` reacts to the webhook immediately; `pollSCM` checks periodically but adds delay and load.

### Q7. What difference does `mvn compile` make compared with `mvn clean`?

`mvn compile` compiles sources but leaves old artifacts in `target/`, so deleted/renamed classes can survive and cause stale behaviour. `mvn clean` deletes `target/` first, guaranteeing a fresh build.

### Q8. The project folder is `studentapp`, not `mavenjava`. Which parts of the script change?

Every path that references the folder:

```groovy
dir('studentapp') { ... }
bat 'mvn clean -f studentapp'
```

**Why:** `-f` points at the POM; `dir` changes the working directory. Update both if the pipeline uses them.

### Q9. If the install stage fails, will test and package still run?

**No.** A Declarative Pipeline stops at the first failed stage and skips the rest. To continue deliberately, wrap the stage in `catchError` and decide the final build result.

### Q10. Where do you add deployment to Tomcat after packaging?

A new `Deploy` stage immediately after `Package`, or in `post { success { ... } }`:

```groovy
stage('Deploy') {
    steps {
        bat 'copy target\\*.war C:\\tomcat\\webapps\\'
    }
}
```

**Why:** deployment must only happen after a successful package, and `post { success }` guarantees the build passed.

### Q11. Why is `tools { maven 'MAVEN_HOME' }` used?

It tells Jenkins to use the Maven installation configured under that name and puts `mvn` on the PATH for the build. Without it, `mvn` may not be found or the wrong version may run.

### Q12. The GitHub repository is private. How do you secure credentials in the clone stage?

Store them in the Credentials Manager, then:

```groovy
git branch: 'main',
    credentialsId: 'github-creds',
    url: 'https://github.com/<user>/<private-repo>.git'
```

Or use `withCredentials` for other commands. **Why:** the credential is injected only for that step and masked in logs; nothing secret is stored in the Jenkinsfile.

### Q13. Which plugin is needed to recognize the `pipeline { ... }` syntax?

The **Pipeline** plugin (the `workflow-aggregator` bundle, which includes Pipeline: Declarative). Without it, Jenkins treats the script as an unknown job type.

### Q14. The script uses `bat`. What changes on Linux?

Replace every `bat` with `sh`:

```groovy
sh 'mvn clean install'
```

**Why:** `bat` invokes `cmd.exe` on Windows; `sh` invokes the POSIX shell. A Linux agent has no `bat` step.

### Q15. How do you stop execution if the GitHub clone fails?

Do **not** wrap the clone in `catchError` or `try/catch` — a failed step already aborts the pipeline. If you need an explicit message:

```groovy
script {
    if (sh(script: 'git clone ...', returnStatus: true) != 0) {
        error 'Git clone failed'
    }
}
```

**Why:** `error` marks the build as failed and stops execution immediately.

### Q15b. Print a welcome message to confirm the pipeline works.

```groovy
steps {
    echo 'Welcome to the Jenkins Pipeline!'
}
```

**Why:** `echo` writes to the console log; it is the simplest smoke test that a stage executed.

### Q16. A teammate forgot to pull the latest code. How do you fix it in the pipeline?

Add a checkout stage at the start:

```groovy
stage('Checkout') {
    steps {
        git branch: 'main', url: 'https://github.com/<user>/<repo>.git'
        // or: checkout scm
    }
}
```

**Why:** the pipeline always builds the exact revision it checks out; a fresh checkout removes dependence on anyone's local state.

### Q17. `Hello.java` must be compiled on every commit. How do you add this?

```groovy
stage('Build') {
    steps {
        bat 'javac Hello.java'
    }
}
triggers { githubPush() }
```

**Why:** `javac` compiles the file; the webhook makes the pipeline run on every push. Use `sh` on Linux.

### Q18. Tests must stop the pipeline if they fail. Where does `mvn test` go?

Directly inside the `steps` of a `Test` stage, with no `catchError` or `|| true`:

```groovy
stage('Test') {
    steps {
        bat 'mvn test'
    }
}
```

**Why:** a non-zero exit code fails the stage, and Jenkins skips the remaining stages.

### Q19. The pipeline should run every evening at 6 PM. How?

```groovy
triggers {
    cron('0 18 * * *')
}
```

**Why:** the five cron fields are minute, hour, day-of-month, month, day-of-week. `0 18 * * *` = 18:00 every day. Jenkins uses its own timezone unless configured otherwise.

### Q20. Package only if compilation succeeds. How do you connect the stages?

Declare them sequentially inside `stages`:

```groovy
stage('Compile') { steps { bat 'mvn compile' } }
stage('Package') { steps { bat 'mvn package' } }
```

**Why:** Declarative stages run in order, and a failed stage halts the pipeline, so `Package` only runs after a successful `Compile`.

### Q21. The professor wants a `.jar` for submission. Which stage?

A `Package` stage with `mvn package` (or `mvn install`), then archive the artifact:

```groovy
stage('Package') {
    steps {
        bat 'mvn package'
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
}
```

### Q22. Git clone fails due to wrong credentials. How do you secure and use them?

1. `Manage Jenkins → Credentials` → update the username/token.
2. Reference the credential ID:

```groovy
git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/...'
```

**Why:** the old token is cached in the credential store, not in the script; updating it there fixes every job that references the ID.

### Q23. A teammate wants the build number in the console log. How?

```groovy
steps {
    echo "Build Number: ${env.BUILD_NUMBER}"
}
```

**Why:** `env.BUILD_NUMBER` is a built-in environment variable; `${...}` interpolates it into the Groovy string.

---

## Week 10: Minikube and Nagios

### Q1. A pod keeps restarting. What do you do?

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
```

**Why:** `describe` shows events (image pull errors, OOMKilled, failed probes); `--previous` shows logs from the crashed instance. Common causes: wrong command/image, missing config/secret, failing liveness probe, memory limit too low.

### Q2. A pod is stuck in `Pending`. What are the possible reasons and how do you troubleshoot?

```bash
kubectl describe pod <pod-name>
kubectl get nodes
kubectl describe node <node>
```

**Why:** `Pending` means the scheduler could not place the pod. Check insufficient CPU/RAM, node taints/affinity, unschedulable nodes, and unbound PersistentVolumeClaims. The `Events` section at the bottom of `describe` usually names the exact reason.

### Q3. How do you debug a failed deployment?

```bash
kubectl get deployments
kubectl describe deployment <name>
kubectl get replicasets
kubectl get pods -l app=<name>
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

**Why:** work down the chain — Deployment → ReplicaSet → Pod. `describe` shows rollout conditions and events; pod logs show application errors.

### Q4. Some replicas fail health checks. How do you find the root cause?

```bash
kubectl describe pod <pod-name>        # probe failures and events
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- sh      # test the endpoint manually
```

**Why:** check the readiness/liveness probe path, port and initial delay; confirm the app listens on `0.0.0.0` (not only `127.0.0.1`); verify startup time against `initialDelaySeconds`. A wrong probe path/port is the most common cause.

### Q5. How do you roll back a faulty deployment?

```bash
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout status deployment/<name>
```

**Why:** Kubernetes keeps ReplicaSet revisions; `undo` switches back to the previous one and performs a rolling update. Add `--to-revision=<n>` to pick a specific revision.

### Q6. How do you debug a running pod?

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
kubectl exec -it <pod-name> -- sh
```

**Why:** logs show application output, `describe` shows cluster events and state, and `exec` lets you inspect files, environment variables, DNS and connectivity from inside.

### Q7. You need to expose a local service externally. How?

```bash
kubectl expose deployment mynginx --type=NodePort --port=80
minikube service mynginx
# or:
kubectl port-forward svc/mynginx 8081:80
```

**Why:** a `NodePort` service opens a port on every node; `minikube service` resolves the reachable URL. On a cloud cluster, `type=LoadBalancer` provisions an external load balancer.

### Q8. How do you start and stop Nagios?

```bash
docker start nagiosdemo
docker stop nagiosdemo
```

**Why:** the Nagios server runs in a container; start/stop control the container lifecycle without deleting its configuration.

### Q9. Nagios shows "Unable to connect to Nagios process". How do you resolve it?

```bash
docker ps -a
docker logs nagiosdemo
docker restart nagiosdemo
```

**Why:** the web UI is up but the Nagios core process is not. Check the logs for a configuration error (`nagios -v /opt/nagios/etc/nagios.cfg`), then restart. Persistent config errors must be fixed in the config files, not just restarted away.

### Q10. A new host was added but does not appear in the web UI. How do you check?

```bash
# on the host
docker exec -it nagiosdemo nagios -v /opt/nagios/etc/nagios.cfg
docker exec -it nagiosdemo ls /opt/nagios/etc/
docker restart nagiosdemo
```

**Why:** Nagios only loads configuration at startup/reload. Check that the host definition is in an included directory, the syntax is valid, and the configuration was reloaded. Also check the host's `parents`/address and that notifications are enabled.

### Q11. How do you check whether Nagios is running properly?

```bash
docker ps
docker logs nagiosdemo
curl -I http://localhost:8888
```

**Why:** container status, logs and an HTTP response cover the three layers: process, application, and web UI.

### Q12. How do you view Nagios logs in real time?

```bash
docker logs -f nagiosdemo
```

**Why:** `-f` follows the container's stdout/stderr. For Nagios' own log file, `docker exec -it nagiosdemo tail -f /var/log/nagios/nagios.log`.

### Q13. What are the advantages of Nagios?

Real-time monitoring of hosts and services, automatic alerts/notifications, early problem detection, plugin-based extensibility, centralized dashboard, and reduced downtime through fast incident response.

---

## Week 11: CI/CD Theory and Viva

### Q1. What is Continuous Integration (CI)?

Developers merge code into a shared repository frequently (several times a day). Each merge triggers an automated build and test run, so integration errors are found within minutes instead of at release time.

### Q2. What is Continuous Delivery vs Continuous Deployment (CD)?

**Continuous Delivery:** every change that passes the pipeline is automatically prepared for release, but a human approves the production deploy. **Continuous Deployment:** every passing change is deployed to production automatically, with no manual gate.

### Q3. What is Jenkins' role in a CI/CD pipeline?

Jenkins is the automation server/orchestrator: it detects changes (webhook/polling), runs the build and tests, packages artifacts, deploys them, and reports results. It integrates with Git, Maven, Docker, Tomcat, email, etc.

### Q4. What is a webhook in GitHub?

An HTTP callback configured on a repository. When a selected event occurs (push, pull request, release), GitHub sends a POST request with a JSON payload to the configured URL. Jenkins' endpoint is `/github-webhook/`.

### Q5. Why are webhooks used in Jenkins integration?

They notify Jenkins **immediately** on a push, so the build starts within seconds. Without them Jenkins must poll the repository, which adds delay and unnecessary network/API load.

### Q6. What build triggers are available in Jenkins?

Manual ("Build Now"), SCM polling (`pollSCM`), GitHub webhook (`githubPush`), cron/scheduled (`cron`), upstream/downstream project triggers, remote trigger token, and generic webhook triggers.

### Q7. Difference between polling and webhook triggers?

| Polling | Webhook |
|---|---|
| Jenkins asks GitHub periodically | GitHub pushes an event to Jenkins |
| Delay up to the poll interval | Near-instant |
| Wastes requests even with no changes | Only fires on real events |
| Works without public access | Needs a reachable Jenkins URL (ngrok in the lab) |

### Q8. What is ngrok and why is it used?

A tunneling tool that exposes a local port through a public HTTPS URL. It is used because GitHub cannot reach `localhost`; ngrok forwards webhook requests to the local Jenkins server.

### Q9. How does ngrok help with webhooks for a local Jenkins?

Run `ngrok http 8080`; it prints a public URL such as `https://abc.ngrok-free.dev`. Configure GitHub's payload URL as `https://abc.ngrok-free.dev/github-webhook/`, and every push is forwarded to `localhost:8080`.

### Q10. Why configure email notifications, and how are they useful?

They inform the team automatically about build success, failure or instability, with links/logs. This shortens feedback time, makes failures visible without watching Jenkins, and creates a record of build health.

---

## Week 12: EC2 and Cloud Deployment

### Q1. `index.html` is on your laptop; an Amazon Linux 2 EC2 instance is running but the page does not load. How do you host it?

```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo cp index.html /var/www/html/
```

Then allow HTTP (port 80) in the security group and open `http://<public-ip>/index.html`.

**Why:** Apache (`httpd`) serves files from `/var/www/html`. On Amazon Linux 2 the package manager is `yum`; on Ubuntu it is `apt install apache2`.

### Q2. The page still does not load after copying to `/var/www/html/`. What two issues do you check?

1. **Security group** — is inbound TCP 80 allowed from your IP/`0.0.0.0/0`?
2. **Apache service** — is it running?

```bash
sudo systemctl status httpd
sudo systemctl start httpd
```

**Why:** the file being in place is not enough; the web server must be running and the network path must be open. Also check file permissions (`chmod 644`) and SELinux context on RHEL-family AMIs.

### Q3. Apache stops after an instance reboot. How do you make it auto-start?

```bash
sudo systemctl enable httpd
```

**Why:** `enable` creates the boot-time symlink; `start` only starts it now. Use both: `systemctl enable --now httpd`.

### Q4. Maven is not installed on the EC2 instance. How do you install it?

**Amazon Linux:**
```bash
sudo yum install -y java-17-amazon-corretto-devel maven
```

**Ubuntu:**
```bash
sudo apt update
sudo apt install -y default-jdk maven
```

**Why:** Maven needs a JDK; installing both ensures `mvn` can compile. Verify with `mvn -v`.

### Q5. A `.war` is generated in `target/`. Where do you place it for Tomcat, and why?

```bash
sudo cp target/*.war /opt/tomcat/webapps/
```

**Why:** Tomcat monitors `webapps/` and auto-deploys every WAR it finds, creating a context path from the file name. No server restart is needed (though it may take a few seconds).

### Q6. The app is deployed but the browser shows 404. What do you check?

```bash
sudo systemctl status tomcat
ls /opt/tomcat/webapps/
sudo tail -f /opt/tomcat/logs/catalina.out
```

Check the URL: `http://<public-ip>:8080/<war-name>/` (the context path is case-sensitive). A 404 means Tomcat is reachable but no app matches the path — the WAR may have failed to deploy or the context name is wrong.

### Q7. `curl localhost:8080` works on the instance but the browser cannot reach it. What AWS setting is missing?

An inbound rule in the **security group**:

| Type | Port | Source |
|---|---|---|
| Custom TCP | 8080 | My IP (or 0.0.0.0/0) |

**Why:** the security group is a firewall in front of the instance. Localhost bypasses it, which is why the app works inside but not outside.

### Q8. The public IP changes after reconnecting Wi-Fi. What AWS feature avoids this?

An **Elastic IP** — a static public IPv4 address. Allocate it and associate it with the instance; it remains the same across stop/start.

**Why:** by default, the public IPv4 is dynamic and released when the instance stops. An Elastic IP is reserved for your account until you release it.

### Q9. How do you automate deployment whenever the instance starts?

Use **EC2 user data** (a script that runs on first boot):

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Deployed</h1>" > /var/www/html/index.html
```

**Why:** user data runs during instance initialisation (first boot), so a new instance configures itself without manual SSH. For repeatable app deployment, combine it with a Docker image or a configuration-management tool; cloud-init can also be configured to run the script on every boot.

### Q10. The instance has no internet but Maven needs to download dependencies. What AWS change allows this securely?

Put the instance in a **private subnet** and route outbound traffic through a **NAT Gateway**:

```text
Private subnet route table: 0.0.0.0/0 → nat-xxxxxxxx
```

**Why:** a NAT Gateway allows outbound internet access (for Maven Central) while blocking inbound connections from the internet. A public subnet with an Internet Gateway would expose the instance directly.

---

## Pattern Recognition Cheat Sheet

| The question says… | Think… | Command |
|---|---|---|
| "discard / undo local edits" | working tree | `git restore <file>` |
| "unstage" | index | `git restore --staged <file>` |
| "wrong commit message, not pushed" | last commit | `git commit --amend` |
| "recover deleted branch" | reflog | `git reflog` + `git branch <name> <hash>` |
| "save work, switch branch" | stash | `git stash` / `git stash pop` |
| "fetch but don't merge" | remote-tracking | `git fetch origin` |
| "push first time / track" | upstream | `git push -u origin <branch>` |
| "delete remote branch" | refspec | `git push origin --delete <branch>` |
| "stale remote branch" | prune | `git remote prune origin` |
| "remove secret from history" | rewrite | `git filter-repo --path <f> --invert-paths` |
| "conflict markers" | manual merge | edit → `git add` → `git commit` |
| "apply patch with authorship" | mailbox patch | `git am file.patch` |
| "dependency conflict" | nearest-wins | `mvn dependency:tree` |
| "executable JAR" | manifest | `maven-jar-plugin` + `mainClass` |
| "WAR deploy" | container | copy to `webapps/` |
| "unsupported class version" | compiler/JDK | `maven-compiler-plugin` + `JAVA_HOME` |
| "skip tests" | build flag | `-DskipTests` |
| "container active?" | status | `docker ps` |
| "enter container" | exec | `docker exec -it <c> sh` |
| "port already allocated" | host port conflict | change `-p host:container` |
| "image for offline transfer" | save/load | `docker save -o x.tar <img>` |
| "container dies / restart" | restart policy | `--restart unless-stopped` |
| "which container uses port" | mapping | `docker ps` + `docker port` |
| "multiple services together" | Compose | `docker compose up -d` |
| "rebuild only one service" | targeted build | `docker compose up -d --build <svc>` |
| "data must persist" | volume | named volume + `-v` |
| "pod Pending" | scheduler | `kubectl describe pod` |
| "pod restarting" | crash loop | `kubectl logs --previous` |
| "rollback deployment" | revisions | `kubectl rollout undo` |
| "expose service" | NodePort | `kubectl expose ... --type=NodePort` |
| "monitor hosts" | Nagios | `docker run ... -p 8888:80` |
| "build on push" | webhook | `githubPush()` + `/github-webhook/` |
| "local Jenkins to GitHub" | tunnel | `ngrok http 8080` |
| "mail on failure" | SMTP | Email Extension + app password |
| "page not loading on EC2" | firewall | security group inbound rule |
| "IP keeps changing" | static IP | Elastic IP |
| "install on first boot" | user data | EC2 user data script |

---

*End of scenario answers. Steps and commands for each experiment: [`SE_Lab_Manual.md`](SE_Lab_Manual.md).*

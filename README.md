# CI/CD Pipeline with Jenkins, Maven & Nexus 🚀

A fully automated CI/CD pipeline built with Jenkins, Maven, and Sonatype Nexus, running entirely in Docker containers on a custom network.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| Jenkins | CI/CD automation server |
| Maven | Build and dependency management |
| Sonatype Nexus 3 | Private artifact repository manager |
| Docker | Container runtime |
| GitHub | Source code hosting |
| Java 1.8 | Application runtime |
| Groovy | Jenkinsfile scripting language |

---

## 📌 What I Built & Learned

### 🐳 Docker Infrastructure
- [ ] Created a custom Docker network (`maven-app1-network`) so Jenkins and Nexus containers can communicate using container names instead of IP addresses
- [ ] Ran Jenkins and Nexus as separate Docker containers with persistent volumes so data survives container restarts
- [ ] Used Docker volumes (`jenkins_home`, `nexus_data`) to protect data permanently
- [ ] Mounted Docker socket into Jenkins container so Jenkins can run Docker commands inside the pipeline

### ⚙️ Jenkins Configuration
- [ ] Configured Jenkins with Maven installation named `maven3` via Global Tool Configuration
- [ ] Installed required plugins: `Nexus Artifact Uploader`, `Pipeline Utility Steps`
- [ ] Stored Nexus credentials securely in Jenkins credentials manager with ID `nexus-credentials`
- [ ] Configured Pipeline job to pull `Jenkinsfile` directly from GitHub (Pipeline script from SCM)
- [ ] Added `buildDiscarder` to keep only last 3 builds and builds from last 5 days to save disk space

### 📦 Nexus Repository Manager
- [ ] Created two separate hosted Maven repositories:
  - `maven-app1-snapshots` → for development builds (allow redeploy)
  - `maven-app1-releases` → for stable builds (disable redeploy)
- [ ] Created a dedicated Jenkins user in Nexus with admin role for artifact uploads
- [ ] Understood why snapshot and release artifacts must be segregated to protect production stability

### 📝 Jenkinsfile Pipeline
- [ ] Written in Groovy declarative pipeline syntax
- [ ] **Stage 1 — Build:** Maven compiles code, runs unit tests, and packages into `.war` file
- [ ] **Stage 2 — Upload:** Artifact uploaded to correct Nexus repo based on version

### 🔄 Smart Version Detection
- [ ] Used `Pipeline Utility Steps` plugin to read `pom.xml` dynamically with `readMavenPom`
- [ ] Implemented automatic repository routing logic:
```groovy
def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ?
                   "maven-app1-snapshots" : "maven-app1-releases"
```
- [ ] No hardcoded values — artifact name, version, and target repo all read from `pom.xml` automatically

### 🐛 Real Errors Debugged
- [ ] Fixed DNS resolution failure inside Jenkins container (`Temporary failure in name resolution`)

---

## 🏗️ Pipeline Architecture

```
Developer pushes code to GitHub
        ↓
Jenkins pulls Jenkinsfile + source code
        ↓
Maven builds project
  └── Compiles Java code
  └── Runs unit tests (CalculatorTest.java)
  └── Packages into .war file
        ↓
readMavenPom reads version from pom.xml
        ↓
version ends with SNAPSHOT?
  YES → upload to maven-app1-snapshots
  NO  → upload to maven-app1-releases
        ↓
Artifact stored in Nexus ✅
```

---

## 🔄 Snapshot vs Release Strategy

| | Snapshot | Release |
|---|---|---|
| Version example | `1.0.0-SNAPSHOT` | `1.0.0` |
| Repository | maven-app1-snapshots | maven-app1-releases |
| Can overwrite | ✅ Yes | ❌ No |
| Used for | Development | Production |
| Changes every build | ✅ Yes | ❌ Locked forever |

---

## 📁 Project Structure

```
maven-app1/
├── src/
│   ├── main/
│   │   ├── java/in/javahome/myweb/controller/
│   │   │   └── Calculator.java        ← business logic
│   │   └── webapp/
│   │       ├── index.jsp              ← web UI
│   │       └── WEB-INF/web.xml        ← web config
│   └── test/
│       └── java/in/javahome/myweb/controller/
│           └── CalculatorTest.java    ← unit tests
├── pom.xml                            ← Maven config
└── Jenkinsfile                        ← Pipeline definition
```

---

## 🐳 Docker Setup

```bash
# Create network
docker network create maven-app1-network

# Start Jenkins
docker run -d \
  --name jenkins \
  --network maven-app1-network \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  -p 8082:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /run/host-services/docker.proxy.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

# Start Nexus
docker run -d \
  --name nexus \
  --network maven-app1-network \
  -p 8081:8081 \
  -v nexus_data:/nexus-data \
  sonatype/nexus3
```

---

## 🔗 Access

```
Jenkins → http://localhost:8082
Nexus   → http://localhost:8081
```

---

## 💡 Key Concepts Learned

- **Why Docker network?** Containers on the same network communicate by name. Jenkins reaches Nexus via `nexus:8081` not `localhost:8081` because localhost inside a container means the container itself.
- **Why two Nexus repos?** To protect production. Snapshot artifacts change constantly during development. Release artifacts are locked forever. Mixing them risks overwriting a stable production version with a broken dev build.
- **Why build discarder?** Every build consumes disk space. Without cleanup Jenkins disk fills up and the team cannot deploy. Keeping only the last 3 builds is standard practice.
- **Why read pom.xml dynamically?** Hardcoding version numbers in Jenkinsfile means updating two files every release. Reading from pom.xml means one change propagates every

<img width="1871" height="865" alt="لقطة شاشة 2026-03-08 095514" src="https://github.com/user-attachments/assets/70e03b1a-8651-479d-8f52-d1eb94259c9a" />
<img width="916" height="452" alt="لقطة شاشة 2026-03-08 090847" src="https://github.com/user-attachments/assets/4ea63802-5844-424a-9b6b-1edebe00e7ed" />
<img width="1838" height="794" alt="لقطة شاشة 2026-03-08 170039" src="https://github.com/user-attachments/assets/c4e76db4-74d5-42cc-b98d-960fbbe8c2fd" />
<img width="1860" height="885" alt="لقطة شاشة 2026-03-08 221743" src="https://github.com/user-attachments/assets/066e76ef-1ccc-46be-80c2-8af27e2040a4" />
<img width="655" height="363" alt="لقطة شاشة 2026-03-08 085953" src="https://github.com/user-attachments/assets/f79350aa-517b-4fa2-9217-15f43429da0f" />
<img width="1884" height="454" alt="لقطة شاشة 2026-03-08 223400" src="https://github.com/user-attachments/assets/8f6c9f9f-590f-4a9b-969c-8f91a4d248d1" />


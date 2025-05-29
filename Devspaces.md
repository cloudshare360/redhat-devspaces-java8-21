You're looking to **create a DevSpace environment** (e.g., in Red Hat DevSpaces or Eclipse Che) that:

✅ Includes **SDKMAN!**  
✅ Has pre-installed tools to install packages like Java, Python, Node.js, Go, etc.  
✅ Supports development for **Java, Python, Node.js, Go, React, Angular, Next.js**  
✅ Has **JDK 11, 17, and 21** managed via SDKMAN!, with **JDK 21 as default**

---

## ✅ Recommended Solution: Use a **Custom Devfile with a Base Image**

You’ll need:
- A base image that includes basic dev tools
- A `devfile.yaml` to define the workspace setup
- Post-start commands to install **SDKMAN!**, then JDKs, Node.js, Python, etc.

---

## 🔧 Step-by-Step Setup

### 📦 1. **Choose a Base Image**
Use an image that already has many tools installed — like [Eclipse Theia](https://theia-ide.dev/) or a custom one based on **Ubuntu** or **Red Hat UBI**.

Example base image:

```yaml
image: quay.io/devfile/universal-developer-image:latest
```

This image includes:
- Git
- Docker
- Podman
- Make, GCC, etc.
- Java (but not multiple versions)
- Node.js, Python, Go

> You can also build your own image using Dockerfile if needed.

---

### 📄 2. **Create a `devfile.yaml`**

Here’s a full example of a `devfile.yaml` that:

- Uses the Universal Dev Image
- Installs SDKMAN!
- Sets up JDK 11, 17, 21
- Sets JDK 21 as default
- Installs Node.js, Python, Go, etc.

```yaml
schemaVersion: 2.2.0
metadata:
  name: fullstack-dev-environment
components:
  - name: universal-image
    container:
      image: quay.io/devfile/universal-developer-image:latest
      memoryLimit: 4Gi
      cpuLimit: "2"
commands:
  - exec:
      id: setup-sdkman
      component: universal-image
      commandLine: |
        bash -c '
          # Install SDKMAN!
          curl -s "https://get.sdkman.io" | bash &&
          source "$HOME/.sdkman/bin/sdkman-init.sh" &&

          # Install JDKs
          sdk install java 11.0.22-open
          sdk install java 17.0.10-open
          sdk install java 21.0.2-open

          # Set default JDK
          sdk default java 21.0.2-open &&

          # Install other tools
          sdk install nodejs
          sdk install python
          sdk install go

          echo "✅ SDKMAN! setup complete with Java 11, 17, 21, Node.js, Python, Go"
        '
      label: "Setup SDKMAN and required runtimes"
events:
  postStart:
    - setup-sdkman
```

---

### 🛠️ 3. **Optional: Add Language Tools & Extensions**

If you're using VS Code-based DevSpace UI, you can also include extensions in your devfile:

```yaml
attributes:
  extensions:
    - https://open-vsx.org/api/redhat/java/0.26.0/file/redhat.java-0.26.0.vsix
    - ms-python.python
    - dbaeumer.vscode-eslint
    - esbenp.prettier-vscode
    - angular.ng-template
    - next-labs.nextjs
```

---

### 🚀 4. **How to Launch Your Dev Environment**

You can launch this environment using:

#### Option A: Open in Red Hat DevSpaces / Eclipse Che
Just put your `devfile.yaml` at the root of your GitHub/Git repo, and open it in DevSpaces:

```bash
https://<your-devspaces-instance>/che/index.html#https://github.com/yourname/your-repo
```

#### Option B: Use `chectl` CLI
Install the DevSpaces CLI and start from local or remote devfile:

```bash
chectl workspace:start --devfile=path/to/devfile.yaml
```

---

## 📌 Summary

| Requirement | How It's Met |
|------------|--------------|
| SDKMAN! | Installed via `curl` + `source` |
| Multiple JDKs | Installed using `sdk install java <version>` |
| Default JDK 21 | `sdk default java 21.x` |
| Java, Python, Node.js, Go | Installed via SDKMAN |
| React/Angular/Next.js | Supported via SDKMAN + IDE extensions |
| Customizable | Yes, via devfile |

---

## 💡 Bonus: SDKMAN Quick Commands (Once Installed)

To switch between Java versions later:

```bash
sdk use java 17.0.10-open   # Switch to Java 17
sdk use java 21.0.2-open    # Back to Java 21
```

To list installed versions:

```bash
sdk ls java
```

---

Let me know if you'd like help creating a Dockerfile to build your own image or automating IDE extension installs!
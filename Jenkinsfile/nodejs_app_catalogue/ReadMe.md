# Why Build on Docker container
1. **Zero Tools Required on the Jenkins Host**
If you don't use a custom Docker agent, Jenkins has to run the compilation commands (npm install, npm build) directly on the WSL host system. This means you would be forced to manually install and manage Node.js, npm, and specific version runtimes directly on your host OS.

2. **Eliminating the "It Works on My Machine" Syndrome (Determinism)**
If a different developer runs the Jenkins job on a server that has Node v14 installed, but your app requires Node v18, the build will break or silently corrupt packages.

3. **Native Security Isolation during the Build**
Node.js dependencies (downloaded from public repositories) can sometimes carry malicious scripts that trigger during the installation phase.



# Local Testing & Jenkins Directory Architecture
* **Jenkins Home Directory:** /var/lib/jenkins

* **jenkins/Active Pipeline Workspace:** /var/lib/jenkins/workspace/<your-job-name>/* 

* **Operational Insight:** Jenkins clones your Git repository and executes configurations strictly inside this workspace. To test components manually from your personal user terminal, you must navigate into this directory or copy the assets out.

* **Manual Execution & Local Compilation:** You can enter the workspace directory to test the microservice natively or trigger manual builds:

```
cd /var/lib/jenkins/workspace/<your-job-name>/catalogue/
node server.js
docker build -t awajid3/nodejs-catalogue:v1 .
```

# Setup the Custom Docker Agent

**Action:** 
Build the custom builder tool environment directly from the workspace directory.
```
cd /var/lib/jenkins/workspace/<your-job-name>/
docker build -f Dockerfile_node_agent_custom -t awajid3/node18agent:v1 .
```
# Initialize Private Isolated Network
**Action:** Create a user-defined bridge network to activate automated container-name DNS resolution.

```
docker network create my_custom_network
```
# Build and Launch the Pre-seeded Database
**Action:** Navigate to the mongo/ companion folder, build the self-populating database image, and run it inside the private network using a network alias.

```
cd /var/lib/jenkins/workspace/<your-job-name>/mongo/
docker build -t awajid3/mongo:v1 .

docker run -d \
  --name mongodb \
  --network my_custom_network \
  --network-alias mongodb \
  awajid3/mongo:v1
```

# Launch the Catalogue Microservice & Verify Communication
**Action:** Run the custom Catalogue image within the identical network space.The 

**Networking Magic:** Because both containers reside in my_custom_network, they communicate seamlessly using the container name/alias mongodb. The connection target is dynamically read by the retry loop (mongoLoop) inside server.js.

**API Validation:** Test the integration endpoint using standard http protocol (not https, as the app container handles raw HTTP traffic):

```
curl http://localhost:8091/categories
``` 
**Expected Result:** A clean JSON data payload outputting ["Artificial Intelligence","Robot"], proving the end-to-end integration works flawlessly!
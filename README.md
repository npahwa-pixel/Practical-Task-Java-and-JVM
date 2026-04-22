# Tomcat + Jenkins Practical Task

## Objective

This practical task demonstrates how to install and configure **Apache Tomcat**, deploy **Jenkins WAR** inside Tomcat, enable **JMX monitoring**, tune JVM heap settings, connect with **JConsole**, and finally run **Jenkins as a standalone Java application**.

This task helps in understanding:

- How Tomcat runs as a Java process
- How web applications are deployed in Tomcat
- How JVM ports behave before and after enabling JMX
- How heap size affects Java applications
- How Jenkins can run both inside Tomcat and independently

---

## Environment Used

| Component | Version |
|----------|---------|
| OS | macOS |
| Java | OpenJDK 17 |
| Tomcat | Apache Tomcat 9.0.117 |
| Application | Jenkins WAR |
| Monitoring Tool | JConsole |

---

## Prerequisites

Tomcat requires Java to run, so Java must be installed and configured before starting Tomcat.

### Install Java

```bash
# Install OpenJDK 17 using Homebrew
brew install openjdk@17
```

### Set Java environment variables

```bash
# Set JAVA_HOME to the installed JDK path
export JAVA_HOME=$(/usr/libexec/java_home)

# Add Java binaries to PATH so commands like java and javac work globally
export PATH=$JAVA_HOME/bin:$PATH
```

### Verify Java installation

```bash
# Check whether Java is installed correctly
java -version
```

### Sample Output

```text
openjdk version "17.0.18" 2026-01-20
OpenJDK Runtime Environment Homebrew (build 17.0.18+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.18+0, mixed mode, sharing)
```

---

## Step 1: Install and Start Tomcat

Extract the downloaded Tomcat archive:

```bash
# Extract the Tomcat tar.gz archive
# This creates the apache-tomcat-9.0.117 directory

tar -xvf apache-tomcat-9.0.117.tar.gz
```

Move into the Tomcat `bin` directory and make scripts executable:

```bash
# Enter the Tomcat bin directory where startup and shutdown scripts are located
cd apache-tomcat-9.0.117/bin

# Give execute permission to all scripts in the bin directory
chmod +x *
```

Start Tomcat:

```bash
# Start the Tomcat server
./startup.sh
```

### Sample Output

```text
Using CATALINA_BASE: /Users/npahwa/Downloads/apache-tomcat-9.0.117
Using CATALINA_HOME: /Users/npahwa/Downloads/apache-tomcat-9.0.117
Using CATALINA_TMPDIR: /Users/npahwa/Downloads/apache-tomcat-9.0.117/temp
Using JRE_HOME: /opt/homebrew/Cellar/openjdk@17/17.0.18/libexec/openjdk.jdk/Contents/Home
Using CLASSPATH: /Users/npahwa/Downloads/apache-tomcat-9.0.117/bin/bootstrap.jar:/Users/npahwa/Downloads/apache-tomcat-9.0.117/bin/tomcat-juli.jar
Using CATALINA_OPTS:
Tomcat started.
```

### Explanation

- `CATALINA_HOME` points to the Tomcat installation directory.
- `CATALINA_BASE` is the runtime base directory.
- `JRE_HOME` shows which Java runtime is being used.
- `startup.sh` launches Tomcat as a Java process.

---

## Step 2: Verify Tomcat

Open the following URL in the browser:

```text
http://localhost:8080
```

### Expected Result

The default Tomcat welcome page should be displayed.

### Explanation

- `localhost` means the Tomcat server is running on your own machine.
- `8080` is Tomcat's default HTTP port.
- If the page loads, Tomcat is running successfully.

---

## Step 3: Check Ports Used by the Java Process

Run the following command:

```bash
# Show all listening network connections related to the Java process
lsof -i -P -n | grep java
```

### Sample Output

```text
java 13417 dsarraf 44u IPv6 0xbfec41bd44640209 0t0 TCP *:8080 (LISTEN)
java 13417 dsarraf 53u IPv6 0x50f1f644013abe37 0t0 TCP 127.0.0.1:8005 (LISTEN)
```

### Ports Used

- **8080** → HTTP port used by Tomcat
- **8005** → Shutdown port used by Tomcat

### Explanation

- `8080` handles browser requests for hosted applications.
- `8005` is used internally by Tomcat for shutdown commands.
- Since Tomcat runs on the JVM, these ports appear under the Java process.

---

## Step 4: Remove Default Applications

Move to the `webapps` directory:

```bash
# Move from bin to webapps directory
cd ../webapps

# Remove all default deployed applications
rm -rf *
```

This removes all default Tomcat applications, including:

- `ROOT`
- `manager`
- `host-manager`
- `docs`
- `examples`

Restart Tomcat:

```bash
# Stop the Tomcat server
../bin/shutdown.sh

# Start Tomcat again after removing default apps
../bin/startup.sh
```

### Expected Result

Tomcat restarts successfully, but the default welcome applications are no longer available.

### Explanation

- Tomcat automatically serves any application placed in the `webapps` folder.
- Removing the contents of `webapps` removes all default deployed applications.
- Restarting forces Tomcat to reload the deployment directory.

---

## Step 5: Download and Deploy Jenkins WAR into Tomcat

Download Jenkins WAR:

```bash
# Download the latest stable Jenkins WAR package
wget https://get.jenkins.io/war-stable/latest/jenkins.war
```

Copy the WAR file into Tomcat's `webapps` directory:

```bash
# Copy Jenkins WAR into Tomcat webapps so Tomcat can deploy it automatically
cp jenkins.war ../webapps/
```

Restart Tomcat:

```bash
# Restart Tomcat so Jenkins is deployed cleanly
../bin/shutdown.sh
../bin/startup.sh
```

### Expected Result

Tomcat automatically detects and deploys `jenkins.war`.

### Explanation

- A `.war` file is a Java web application archive.
- Tomcat automatically expands and deploys WAR files placed inside `webapps`.
- The deployment path becomes the WAR filename, so `jenkins.war` becomes `/jenkins`.

---

## Step 6: Verify Jenkins Deployment

Open the following URL in the browser:

```text
http://localhost:8080/jenkins
```

### Expected Result

The Jenkins setup page should appear.

### Explanation

- If this page opens, Jenkins was successfully deployed inside Tomcat.
- Tomcat is acting as the servlet container, while Jenkins is running as a hosted Java web app.

---

## Step 7: Enable JMX in Tomcat

To enable JMX monitoring, create or edit `setenv.sh` inside the Tomcat `bin` directory.

### Create or update `setenv.sh`

```bash
# Create a file named setenv.sh in the Tomcat bin directory
# Tomcat reads this file automatically during startup
cat > ./setenv.sh <<'EOF'
export CATALINA_OPTS="
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=1234
-Dcom.sun.management.jmxremote.rmi.port=1235
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
"
EOF

# Make the file executable
chmod +x ./setenv.sh
```

Restart Tomcat:

```bash
# Restart Tomcat so the new JVM options are applied
./shutdown.sh
./startup.sh
```

Check ports again:

```bash
# Check which ports are now being used by the Java process
lsof -i -P -n | grep java
```

### Ports After Enabling JMX

- **8080** → HTTP
- **8005** → Shutdown
- **1234** → JMX port
- **1235** → RMI port

### Explanation

- `CATALINA_OPTS` passes JVM arguments to Tomcat.
- `jmxremote.port=1234` enables JMX monitoring on port 1234.
- `jmxremote.rmi.port=1235` uses a separate RMI communication port.
- `authenticate=false` and `ssl=false` are convenient for local lab work, but should not be used in production without proper security.

---

## Step 8: Use the Same Port for RMI and JMX

Update `setenv.sh` so JMX and RMI both use the same port:

```bash
# Overwrite setenv.sh with JMX and RMI using the same port
cat > ./setenv.sh <<'EOF'
export CATALINA_OPTS="
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=1234
-Dcom.sun.management.jmxremote.rmi.port=1234
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
-Djava.rmi.server.hostname=localhost
"
EOF

# Ensure the script remains executable
chmod +x ./setenv.sh
```

Restart Tomcat:

```bash
# Restart to apply the updated JMX configuration
./shutdown.sh
./startup.sh
```

Check ports again:

```bash
# Verify the updated port usage
lsof -i -P -n | grep java
```

### Ports Now

- **8080** → HTTP
- **8005** → Shutdown
- **1234** → JMX + RMI

### Explanation

- When the JMX port and RMI port are different, Java opens two monitoring ports.
- When both use the same port, monitoring becomes simpler and firewall rules are easier to manage.
- `java.rmi.server.hostname=localhost` ensures JMX advertises the correct host for local connection.

---

## Step 9: Run Tomcat with Small Heap Size

Update `CATALINA_OPTS` to use a very small heap:

```bash
# Set an intentionally very small JVM heap
# This is done only to observe memory failure behavior
export CATALINA_OPTS="-Xms10m -Xmx20m"
```

Start Tomcat:

```bash
# Start Tomcat with the tiny heap configuration
./startup.sh
```

### Expected Error

```text
java.lang.OutOfMemoryError: Java heap space
```

### Explanation

- `-Xms10m` sets the initial heap to 10 MB.
- `-Xmx20m` sets the maximum heap to 20 MB.
- This heap size is too small for Tomcat and Jenkins to initialize properly.
- As a result, the JVM throws `OutOfMemoryError`.

---

## Step 10: Increase Heap Size and Enable Parallel Garbage Collector

Update `CATALINA_OPTS` with larger memory values and enable Parallel GC:

```bash
# Set a larger heap and enable the Parallel Garbage Collector
export CATALINA_OPTS="
-Xms1g -Xmx3g -XX:+UseParallelGC
"
```

Restart Tomcat:

```bash
# Restart Tomcat with the updated JVM memory configuration
./shutdown.sh
./startup.sh
```

### JVM Parameters Used

- **Xms1g** → Initial heap size = 1 GB
- **Xmx3g** → Maximum heap size = 3 GB
- **XX:+UseParallelGC** → Enables Parallel Garbage Collector

### Explanation

- `Xms1g` ensures the JVM starts with 1 GB heap.
- `Xmx3g` allows the heap to grow up to 3 GB.
- `UseParallelGC` improves throughput by using parallel threads for garbage collection.
- With these settings, Tomcat and Jenkins should start more reliably.

---

## Step 11: Connect Using JConsole

Launch JConsole:

```bash
# Start Java's built-in monitoring tool
jconsole
```

Connect to:

```text
localhost:1234
```

### Things to Observe in JConsole

- Memory usage
- Threads
- CPU activity
- Loaded classes
- MBeans

### Explanation

JConsole allows real-time monitoring of the JVM. Using it, you can inspect:

- Heap and non-heap memory usage
- Active threads
- Class loading behavior
- JVM performance metrics
- MBeans exposed by Tomcat and the JVM

---

## Step 12: Stop Tomcat

Stop the server using:

```bash
# Gracefully stop the Tomcat server
./shutdown.sh
```

### Expected Result

Tomcat shuts down and the Java process stops listening on Tomcat ports.

### Explanation

- `shutdown.sh` sends a shutdown signal to Tomcat.
- Once Tomcat exits, the Java process and its listening ports are closed.

---

## Step 13: Launch Jenkins as a Standalone Application

Run Jenkins directly using Java:

```bash
# Run Jenkins directly as a standalone Java application
java -jar jenkins.war
```

Open the following URL:

```text
http://localhost:8080
```

### Expected Result

Jenkins starts independently without Tomcat.

### Explanation

- In this mode, Jenkins runs as its own embedded server.
- Tomcat is not required.
- This demonstrates that Jenkins can run either as a deployed WAR inside Tomcat or as a standalone Java application.

---

## Port Summary

| Stage | Ports Used |
|------|------------|
| Tomcat initial startup | 8080, 8005 |
| After enabling JMX | 8080, 8005, 1234, 1235 |
| After using same JMX and RMI port | 8080, 8005, 1234 |

---

## Key Learnings

- Java must be installed before Tomcat can run.
- Tomcat itself runs as a Java process.
- Default Tomcat ports are **8080** and **8005**.
- Deploying a WAR file inside `webapps` allows Tomcat to host Java web applications.
- JMX monitoring introduces additional JVM ports.
- Using the same port for JMX and RMI simplifies remote connections.
- Very small heap sizes can trigger `OutOfMemoryError`.
- Heap tuning and garbage collector settings directly affect runtime stability.
- JConsole is useful for observing memory, threads, classes, and MBeans.
- Jenkins can run either inside Tomcat or as a standalone Java application.

---

## Notes

- On macOS, `lsof` is commonly used instead of `netstat -tulnp`.
- Restart Tomcat after every change in `CATALINA_OPTS`.
- Ensure `JAVA_HOME` and `PATH` are set correctly before starting Tomcat.
- After copying `jenkins.war` into `webapps`, Tomcat may take a short time to extract and deploy it.
- For real environments, never leave JMX authentication and SSL disabled.

---

## Conclusion

This practical task covered the complete workflow of:

- installing Java
- starting and verifying Tomcat
- checking Java process ports
- removing default Tomcat applications
- deploying Jenkins WAR
- enabling JMX monitoring
- tuning JVM heap size and garbage collection
- monitoring with JConsole
- running Jenkins independently as a standalone application

Overall, this practical provided strong hands-on experience with **Tomcat administration**, **Jenkins deployment**, **JVM configuration**, and **Java process monitoring**.

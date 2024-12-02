# weblogic_nr

Certainly! Below are the detailed steps for each of the tasks you’ve requested: installing WebLogic on a Linux machine, creating a WebLogic Managed Server, deploying a `helloworld.war` application, and installing the New Relic APM agent over it.

---

### **Step 1: Install WebLogic on a Linux Machine**

#### 1.1 **Prerequisites**

Before installing WebLogic, ensure that your Linux machine meets the following requirements:
- **Java Development Kit (JDK)**: WebLogic requires JDK 8 or 11.
- **Operating System**: Linux (e.g., Ubuntu, CentOS, RedHat).
- **WebLogic Version**: This guide assumes you are using Oracle WebLogic 12c or 14c.

#### 1.2 **Install JDK**
WebLogic requires a Java JDK. You can install it using the following commands:

```bash
sudo apt update
sudo apt install openjdk-8-jdk  # For OpenJDK 8
```

You can verify the installation by running:

```bash
java -version
```

#### 1.3 **Download WebLogic Installer**
1. Go to Oracle's official website and download the WebLogic installation ZIP file:
   - [Oracle WebLogic Server Downloads](https://www.oracle.com/middleware/technologies/weblogic.html)
2. Transfer the downloaded ZIP file to your Linux machine (e.g., `weblogic-12c.zip`).

#### 1.4 **Extract and Install WebLogic**
Once the WebLogic ZIP file is downloaded, follow these steps to install WebLogic:

1. Create a directory for WebLogic installation:
   ```bash
   mkdir -p /opt/weblogic
   cd /opt/weblogic
   ```

2. Extract the ZIP file:
   ```bash
   unzip /path/to/weblogic-12c.zip
   ```

3. Start the WebLogic installer:
   ```bash
   java -jar fmw_12.2.1.4.0_weblogic.jar
   ```

4. Follow the installation wizard:
   - Choose to install WebLogic Server.
   - Select the installation directory (`/opt/weblogic`).
   - Accept the license agreement and follow the prompts.

#### 1.5 **Set Up WebLogic Environment**
1. Once installed, create a new environment variable for WebLogic:
   ```bash
   export MW_HOME=/opt/weblogic
   export WL_HOME=$MW_HOME/wlserver
   ```

2. Add these variables to your `.bashrc` or `.profile` file to make them permanent:
   ```bash
   echo "export MW_HOME=/opt/weblogic" >> ~/.bashrc
   echo "export WL_HOME=\$MW_HOME/wlserver" >> ~/.bashrc
   source ~/.bashrc
   ```

---

### **Step 2: Create WebLogic Managed Server and Deploy a `helloworld.war` Application**

#### 2.1 **Create a WebLogic Domain**
1. Navigate to the WebLogic `server/bin` directory:
   ```bash
   cd $WL_HOME/common/bin
   ```

2. Run the WebLogic domain creation wizard:
   ```bash
   ./config.sh
   ```

3. In the Domain Configuration wizard:
   - Choose **Generate a New Domain**.
   - Select **Production Mode** (or Development mode as per your requirement).
   - Choose **WLS Custom Domain**.
   - Provide a domain name and location (e.g., `/opt/weblogic/user_projects/domains/base_domain`).
   - Set the **Administrator username** and **password**.

4. Complete the wizard and exit. This will create the WebLogic domain.

#### 2.2 **Start Admin Server**
To start the Admin Server:
```bash
cd /opt/weblogic/user_projects/domains/base_domain/bin
./startWebLogic.sh
```

The Admin Server should now be running, and you can access the Admin Console at:
```
http://<server_host>:7001/console
```

#### 2.3 **Create WebLogic Managed Server**
1. Log in to the WebLogic Admin Console.
2. Under **Domain Structure**, click on `Servers`.
3. Click **New** and select **Managed Server**.
4. Fill in the following details:
   - **Server Name**: `ManagedServer1`
   - **Listen Address**: Leave it empty or specify your IP address.
   - **Listen Port**: `7003` (default for Managed Servers).
   - **Cluster**: Leave it empty or assign to a cluster if necessary.
5. Click **Next** and then **Finish** to create the Managed Server.

#### 2.4 **Start the Managed Server**
To start the Managed Server:
```bash
cd /opt/weblogic/user_projects/domains/base_domain/bin
./startManagedWebLogic.sh ManagedServer1 http://<admin_server_host>:7001
```

#### 2.5 **Deploy the `helloworld.war` Application**
1. Download or create the `helloworld.war` file (a simple Java web application).
2. Log in to the WebLogic Admin Console.
3. In the Admin Console, go to **Deployments** under **Domain Structure**.
4. Click **Install**.
5. Browse and select the `helloworld.war` file, then click **Next**.
6. Choose **Target** as `ManagedServer1` (or any other server you want to deploy it to).
7. Click **Finish** to deploy the application.

#### 2.6 **Verify the Application**
Once deployed, navigate to:
```
http://<managed_server_host>:7003/helloworld
```
You should see the output of the `helloworld.war` application.

---

### **Step 3: Install New Relic APM Agent Over WebLogic**

#### 3.1 **Download New Relic APM Java Agent**
1. Go to the [New Relic website](https://newrelic.com) and log in to your account.
2. Download the New Relic Java APM agent.

#### 3.2 **Extract the New Relic Java Agent**
1. Extract the New Relic agent:
   ```bash
   mkdir -p /opt/newrelic
   unzip /path/to/newrelic-java.zip -d /opt/newrelic
   ```

2. After extraction, the `newrelic.jar` will be located in `/opt/newrelic`.

#### 3.3 **Configure the New Relic Agent**
1. Edit the `newrelic.yml` file located in `/opt/newrelic/newrelic.yml`.
2. Set the **license_key** and **app_name**:
   ```yaml
   common: &default_settings
     license_key: 'your_new_relic_license_key'
     app_name: 'WebLogic-helloworld'
   ```

   Replace `your_new_relic_license_key` with your New Relic license key, and `WebLogic-helloworld` with your desired application name.

#### 3.4 **Integrate the New Relic Agent with WebLogic**
1. Open the WebLogic `setDomainEnv.sh` script located in your domain directory:
   ```bash
   vi /opt/weblogic/user_projects/domains/base_domain/bin/setDomainEnv.sh
   ```

2. Add the following line at the end of the `JAVA_OPTIONS` variable to include the New Relic agent:
   ```bash
   export JAVA_OPTIONS="$JAVA_OPTIONS -javaagent:/opt/newrelic/newrelic.jar"
   ```

#### 3.5 **Restart WebLogic Servers**
To apply the New Relic agent changes, restart both Admin and Managed Servers.

1. Stop the servers:
   ```bash
   ./stopWebLogic.sh
   ./stopManagedWebLogic.sh ManagedServer1
   ```

2. Start the servers again:
   ```bash
   ./startWebLogic.sh
   ./startManagedWebLogic.sh ManagedServer1
   ```

#### 3.6 **Verify the New Relic Integration**
1. Log in to the [New Relic dashboard](https://newrelic.com).
2. Navigate to **APM** and you should see the `WebLogic-helloworld` application in the list.
3. Verify that performance metrics such as response times, throughput, and error rates are being tracked.

---

### **Conclusion**
You’ve now successfully:
1. Installed WebLogic on a Linux machine.
2. Created a WebLogic Managed Server and deployed a `helloworld.war` application.
3. Installed and configured the New Relic APM agent for monitoring your WebLogic instance.

This setup will allow you to monitor the performance of your WebLogic applications using New Relic’s APM tools.

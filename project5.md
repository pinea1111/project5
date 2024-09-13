# Setup Service Discovery Using Nginx & Consul

## Documentation

Please reference [**Project1**](https://github.com/pinea1111/project-1-docu/blob/master/project1docu.md) for guidance on spinning up an Ubuntu server.

- Rename your EC2 instances to prevent any confusion during your project.

- Click on the **edit icon**.

![1](img/Screenshot%20(155).png)

- Name your server and click the **checkmark icon**.

![2](img/Screenshot%20(156).png)

- Name your Consul server, LoadBalancer server, and the two backend servers for easy identification.

![3](img/Screenshot%20(157).png)

### Allow Required Ports In The Security Group

The Consul service requires specific ports to function correctly. Please open the following ports in your security group.

### Consul Servers

|S/N |Port Name  |Protocol      |Default Port   |
|----|-----------|--------------|---------------|
| 1  |DNS        |TCP and UDP   |8600           |
| 2  |HTTP API   |TCP           |8500           |
| 3  |HTTPS API  |TCP           |8501           |
| 4  |gRPC       |TCP           |8502           |
| 5  |gRPC TLS   |TCP           |8503           |
| 6  |Server RPC |TCP           |8300           |
| 7  |LAN Serf   |TCP and UDP   |8301           |
| 8  |WAN Serf   |TCP and UDP   |8302           |

- Select the **checkbox①** next to your instance, click on **Security②**, and then click on the **security group ID③**.

![4](img/Screenshot%20(158).png)

- Click on **Edit inbound rules**.

![5](img/Screenshot%20(159).png)

- click add rule

![6](img/Screenshot%20(160).png)

- Enter port range

![7](img/Screenshot%20(161).png)

> [!NOTE]
This port supports both TCP and UDP protocols, so you'll need to configure both.

- Choose the appropriate CIDR block.

![8](img/Screenshot%20(162).png)

> [!NOTE]
For the purpose of this project, security measures have been intentionally relaxed by opening SSH, HTTP, HTTPS, and Consul ports to all traffic (0.0.0.0) for rapid development and testing. This configuration is highly insecure and should never be used in production environments.
In a production setting, it is crucial to:
Create separate security groups for each type of instance (consul server, backend server, load balancer), strictly limit inbound and outbound traffic to necessary ports and IP addresses, and implement additional security measures like network ACLs, IAM roles, and encryption.
By following these guidelines, you can significantly enhance the security of your infrastructure and protect your systems from unauthorized access.

- Click on **Add Rule** to specify the port range for the UDP protocol.

- Click on the **Type field①** and choose **Custom UDP**② from the dropdown menu.

![9](img/Screenshot%20(163).png)

- Enter the **Port range** and choose the **CIDR blocks**.

> [!NOTE]
Repeat this process until you've opened up all necessary ports.

- Verify that all the necessary ports are open.

![10](img/Screenshot%20(164).png)

- Click on **Save rules** to apply the updated security group settings.

> [!NOTE]
Currently, the ports are being opened manually one at a time. However, in future projects, you'll learn how to automate these tasks, such as creating multiple instances and configuring all the security group rules at once using code. You'll explore this further when you work with Terraform.

---

### Setup Consul Server

- SSH into the consul server and run **`sudo apt update`** to refresh the package cache.

- Visit the consul [**downloads**](https://developer.hashicorp.com/consul/install) page to **copy** the installation command.

![11](img/Screenshot%20(165).png)

Or execute the following commands to install Consul.

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install consul
```
![12](img/Screenshot%20(166).png)

- Confirm Consul installation by checking its version with the **`consul --version`** command.

![13](img/Screenshot%20(167).png)

- All the Consul server configurations are located in the **`/etc/consul.d`** folder. To configure the Consul server, start by backing up the default configuration file **`consul.hcl`** by renaming it to **`consul.hcl.back`**, using the following command: **`sudo mv /etc/consul.d/consul.hcl /etc/consul.d/consul.hcl.back`**

![14](img/Screenshot%20(167).png)

- Generate an **encrypted key** using the **`consul keygen`** command.

![15](img/Screenshot%20(169).png)

- Create a new file named **`consul.hcl`** in the **`/etc/consul.d`** directory, using the following command: **`sudo vi /etc/consul.d/consul.hcl`**

- Add the following content to the **`consul.hcl`** file, replacing **<YOUR_ENCRYPTED_KEY>** with the encrypted key you generated:

```
"bind_addr" = "0.0.0.0"
"client_addr" = "0.0.0.0"
"data_dir" = "/var/consul"
"encrypt" = "<YOUR_ENCRYPTED_KEY>"
"datacenter" = "dc1"
"ui" = true
"server" = true
"log_level" = "INFO"
```
![16](img/Screenshot%20(170).png)

---

**Here's an explanation of each configuration setting in the Consul configuration file:**

1. **`bind_addr = "0.0.0.0"`**: Specifies the IP address on which Consul will bind to listen for incoming connections. `0.0.0.0` means Consul will listen on all available network interfaces.

2. **`client_addr = "0.0.0.0"`**: Determines the IP address on which the Consul client API will be available. Setting it to `0.0.0.0` allows connections from any IP address.

3. **`data_dir = "/var/consul"`**: Specifies the directory where Consul will store its data, such as the state and logs.

4. **`encrypt = "<YOUR_ENCRYPTED_KEY>"`**: Sets the encryption key for securing communication between Consul servers and clients. Replace this placeholder with your actual generated encryption key.

5. **`datacenter = "dc1"`**: Defines the datacenter name that this Consul server will use. Consul uses datacenters to organize services and nodes.

6. **`ui = true`**: Enables the Consul Web UI. This provides a graphical interface for interacting with Consul's data.

7. **`server = true`**: Indicates that this instance is a Consul server. Server nodes participate in the consensus protocol and store the state of the system.

8. **`log_level = "INFO"`**: Sets the verbosity of the logs. `INFO` level provides a balance of details, logging general information, warnings, and errors.

---

- Run the following command to start the Consul server in the background: **`sudo nohup consul agent -dev -config-dir /etc/consul.d/ &`**.

![17](img/Screenshot%20(171).png)

> [!NOTE]
We use the **`-dev`** flag to indicate that we are running a single Consul server in development mode.

- You can check the status of the Consul server with the following command: **`consul members`**.

![18](img/Screenshot%20(172).png)

- If you visit **`<EC2 Consul Server IP>:8500`**, you should be able to access the Consul dashboard.

![19](img/Screenshot%20(173).png)

> [!NOTE]
Ensure you replace **`<EC2 Consul Server IP>`** with the public IP address of the EC2 instance you're using as the Consul server.

---

> [!NOTE]
Ensure you replace **`<EC2 Consul Server IP>`** with the public IP address of the EC2 instance you're using as the Consul server.

---

### Setup Backend Servers

Since we have the Consul server up and running, let's manage our Nginx backend servers more easily using service discovery. To do this, we'll install Nginx and the Consul agent on all the backend servers. The Consul agent acts like a messenger, automatically registering both the server and the Nginx service running on it with the Consul server, which acts like a central directory.

**Apply the configurations below on both backend servers:**

- SSH into the backend servers and run **`sudo apt-get update -y`** to update package information.

> [!NOTE]
The `y` flag automatically answers **"yes"** to any prompts during the installation process, ensuring that the installation proceeds without requiring manual confirmation.

- Install Nginx on both instances by running the following command: **`sudo apt install nginx -y`**.

> [!NOTE]
After installing Nginx, navigate to the default HTML directory and modify the index.html file on both servers to differentiate them.

- Navigate to the HTML directory by executing the following command: **`cd /var/www/html`**.

- Open the HTML file with your preferred text editor to make edits: **`sudo vi index.html`**.

- Copy the HTML content below into the index.html file. On the second server, replace **SERVER-01** with **SERVER-02** in the HTML file to differentiate between the two backend servers.

```
<!DOCTYPE html>
<html>
<head>
	<title>Kanekis Backend Server </title>
</head>
<body>
	<h1>This is Backend SERVER-01</h1>
</body>
</html>
```

- Install Consul as an agent on the servers. Run the following commands to install Consul:

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install consul
```

- Verify that Consul is installed properly by running the following command: **`consul --version`**.

![20](img/Screenshot%20(178).png)
![21](img/Screenshot%20(179).png)

- Replace the default Consul configuration file **`config.hcl`** located in **`/etc/consul.d`** with your custom **`consul.hcl`** file.

- Rename the default file and create a new one by running the following commands:

```
sudo mv /etc/consul.d/consul.hcl /etc/consul.d/consul.hcl.back
sudo vi /etc/consul.d/consul.hcl
```

- Add the following contents to the file. Replace **`<YOUR_ENCRYPTED_KEY>`①** with your encryption key. Also, replace **`34.201.77.72`②** with your Consul server's IP address.

```
"server" = false
"datacenter" = "dc1"
"data_dir" = "/var/consul"
"encrypt" = "<YOUR_ENCRYPTED_KEY>"
"log_level" = "INFO"
"enable_script_checks" = true
"enable_syslog" = true
"leave_on_terminate" = true
"start_join" = ["34.201.77.72"]
```

- Next, we need to create a **`backend.hcl`** configuration file in the **`/etc/consul.d`** directory to register the Nginx service and its health check URLs with the Consul server. This will enable the Consul server to continuously monitor the health of the Nginx service. Use the following command to create and edit the file: **`sudo vi /etc/consul.d/backend.hcl`**.

- Add the following contents to the **`backend.hcl`** file and save it.

```
"service" = {
  "Name" = "backend"
  "Port" = 80
  "check" = {
    "args" = ["curl", "localhost"]
    "interval" = "3s"
  }
}
```

This configuration registers your backend servers with the Consul server and sets up a health check that uses curl to test the service every 3 seconds.

- Verify the configurations by executing the following command: **`consul validate /etc/consul.d`**.

- Once all configurations are complete, start the Consul agent with the following command: **`sudo nohup consul agent -config-dir /etc/consul.d/ &`**.

- To verify if everything is working correctly, visit your Consul UI. If you see the backend listed in the UI as depicted below, it indicates that the backend has successfully registered itself with Consul.

![21](img/Screenshot%20(186).png)

![22](img/Screenshot%20(187).png)

---

### Setup Load-Balancer

Next, set up the load balancer to automatically update its backend server information based on the service registry maintained by Consul.
To retrieve the backend server details, we will use the **`consul-template`** binary. This tool interacts with the Consul server via API calls to fetch the backend server information. It then uses a template to substitute values and generate the **`loadbalancer.conf`** file, which is utilized by Nginx.

- Log in to the load-balancer server. Update the package information and install unzip with the following commands:

```
sudo apt-get update -y
sudo apt-get install unzip -y
```

![23](img/Screenshot%20(188).png)

- Install Nginx using the following command: **`sudo apt install nginx -y`**.

![24](img/Screenshot%20(189).png)

- Download the consul-template binary using the following command:

```
sudo curl -L  https://releases.hashicorp.com/consul-template/0.30.0/consul-template_0.30.0_linux_amd64.zip -o /opt/consul-template.zip

sudo unzip /opt/consul-template.zip -d  /usr/local/bin/
```

![25](img/Screenshot%20(190).png)

- To verify the installation of consul-template, check its version with the following command: **`consul-template --version`**.

![26](img/Screenshot%20(191).png)

- Create and edit a file named **`load-balancer.conf.ctmpl`** in the **`/etc/nginx/conf.d`** directory, using the following command: **`sudo vi /etc/nginx/conf.d/load-balancer.conf.ctmpl`**.

- Paste the following content into the file:

```
upstream backend {
 {{- range service "backend" }} 
  server {{ .Address }}:{{ .Port }}; 
 {{- end }} 
}

server {
   listen 80;

   location / {
      proxy_pass http://backend;
   }
}
```
---

**Here's a breakdown of the configuration:**

1. **Upstream Block**

```
upstream backend {
 {{- range service "backend" }} 
  server {{ .Address }}:{{ .Port }}; 
 {{- end }} 
}
```

- **upstream backend:** This defines a group of backend servers that Nginx can load-balance requests across.
- **{{- range service "backend" }}:** This is a Consul-Template directive that iterates over all services registered with Consul under the name "backend".
- **server {{ .Address }}:{{ .Port }};:** For each backend service, it adds an entry to the upstream block with the server's address and port.
- **{{- end }}:** Ends the iteration block.

2. **Server Block**

```
server {
   listen 80;

   location / {
      proxy_pass http://backend;
   }
}

```

- **server:** Defines a virtual server that listens for incoming requests.
- **listen 80:** Specifies that this server block will listen on port 80 (the default HTTP port).
- **location /:** Defines a location block for all requests to the root URL (/).
- **proxy_pass http://backend:** Forwards incoming requests to the upstream group named "backend" defined above. Nginx will use the addresses and ports listed in the upstream backend block to balance the requests.

> [!NOTE]
This setup keeps Nginx's backend server list in sync with Consul's. It ensures that Nginx always routes traffic to the currently available backend servers.

---
- Create a file named **`consul-template.hcl`** in the **`/etc/nginx/conf.d/`** directory. This configuration file is used by **consul-template** to specify details about the Consul server IP and the destination path where the processed load-balancer.conf file will be saved.

Use the following command to create and edit the file: **`sudo vi /etc/nginx/conf.d/consul-template.hcl`**.

- Add the following content to the file, replacing **`<Consul Server IP>`** with your Consul server's IP address. This configuration specifies the Consul server details, the path to the template file, the destination for the rendered Nginx configuration, and the command to reload Nginx after updating the configuration.

```
consul {
 address = "<Consul Server IP>:8500"

 retry {
   enabled  = true
   attempts = 12
   backoff  = "250ms"
 }
}
template {
 source      = "/etc/nginx/conf.d/load-balancer.conf.ctmpl"
 destination = "/etc/nginx/conf.d/load-balancer.conf"
 perms       = 0600
 command = "service nginx reload"
 }
```

- Delete the default server configuration to disable it by running the following command: **`sudo rm /etc/nginx/sites-enabled/default`**.

*The default server configuration file should be deleted to avoid inconsistencies with the server's settings.*

- Restart Nginx to apply the changes by running the following command: **`sudo systemctl restart nginx`**.

- Once configurations are complete, start the Consul Template agent using the following command. It continuously monitors Consul for changes.

```
sudo nohup consul-template -config=/etc/nginx/conf.d/consul-template.hcl &
```

- Upon completion, a load-balancer.conf file will be created with backend server information populated from the Consul service registry.

Now, if you access the load balancer IP in your web browser, it will display the custom HTML content from one of the backend servers. When you refresh the page, the load balancer will route your request to the other backend server, displaying its custom HTML content.

![27](img/Screenshot%20(194).png)

![28](img/Screenshot%20(195).png)

his behavior occurs because the load balancer uses a round-robin algorithm by default, distributing incoming requests evenly across all available backend servers.

---

### Service Discovery Test

Now that everything is set up and running, you can test the configuration by observing what happens when you stop one of your backend servers.

![29](img/Screenshot%20(197).png)

Stop one of the backend servers. The Consul server will monitor the health of each registered service. Once a backend server is stopped, Consul will detect the server's unavailability and mark it as unhealthy. The health check for that server will fail, and it will be removed from the load balancer's active pool of servers.

![30](img/Screenshot%20(198).png)

As a result, the load balancer will only direct traffic to the remaining healthy backend servers. This ensures that your application continues to run smoothly without any disruption to users, demonstrating the effectiveness of your service discovery and health check configuration with Consul and Nginx.

![31](img/Screenshot%20(199).png)

![32](img/Screenshot%20(200).png)


**Service Check:** These checks are specific to the services running on the nodes (in this case, Nginx). When you stopped Nginx, the service check that monitors the health of Nginx on that particular node would fail, leading to the "all service checks failed" error.

**Node checks:** These checks monitor the overall health of the node itself, which includes the underlying operating system and possibly other metrics (like CPU, memory, and disk usage). Since stopping Nginx does not necessarily mean the node is unhealthy (the node could still be up and running, responding to pings, etc.), the node checks would still pass.

---
---

#### The End Of Project 5









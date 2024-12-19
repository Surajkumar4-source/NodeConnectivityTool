# NodeConnectivityTool


<br>







## Main Role of the Script:
### *This script is designed to facilitate managing SSH connectivity to multiple compute nodes defined in a YAML configuration file (config.yaml). The script performs the following primary functions:*

### 1. Parse a Configuration File:

- Reads the compute_nodes list from the config.yaml file.
- Extracts the list of nodes (usernames and IP addresses) specified under compute_nodes.

### 2. Check SSH Connectivity:

- Verifies SSH connectivity to each node by attempting to execute a simple command (touch ~/FROM_MASTER.txt) on the remote machine.

### 3. Display Node Status:

- Provides visual feedback about the success or failure of SSH connectivity for each node.

### 4. CLI Options:

- -c or --config <config_file>: Specifies a custom YAML configuration file which contains, your Cluster Nodes IPs.
- -list: Triggers the execution of the SSH connectivity check for all nodes listed in the configuration file.
- -h or --help: Displays usage instructions for the script.

## Key Features:
- Background Execution: Uses background processes (&) to check connectivity for all nodes simultaneously, improving efficiency.
- Color-Coded Output: Provides color-coded status messages for better readability:
- Green: Successful SSH execution.
- Red: Error messages or failures.
- Extensibility: The script is modular and can be extended to perform additional tasks on the remote nodes.

## Use Case:
*This script is particularly useful in High-Performance Computing (HPC) environments, where administrators need to manage and monitor connectivity to a cluster of compute nodes efficiently. It simplifies verifying connectivity and ensures nodes are reachable and operational for tasks like distributed computation or system management.*




<br>
<br>

### 1. config.yaml
  **This YAML file is the configuration file where you define the compute nodes for monitoring. Each node is listed with its username and IP address.**


```yml

# Configuration File for SSH Checker Script
# List of compute nodes in the format username@IP

compute_nodes:
  - suraj@192.168.206.130
  - dhpcsa@192.168.82.149
  - dhpcsa@192.168.82.226
  - demo@1.1.1.1

```

  *It serves as the input for the ssh_cli.sh script, enabling easy customization of the nodes you want to monitor.*


<br>



### 2. ssh_cli.sh
  *This is the main script that checks the SSH connectivity of nodes listed in config.yaml. It attempts to log in to each node using SSH, performs a basic operation (like creating a file), and reports whether the operation succeeded or failed.*

### Features:
- Node status check (up or down).
- Configurable via command-line arguments.
- Outputs the results in a user-friendly format.
- Use case: Automating the process of identifying accessible nodes in a cluster.




<br>

```yml

#!/bin/bash

# SSH Checker Script
# This script checks SSH connectivity to multiple nodes specified in a YAML configuration file.

# Color codes for output
YELLOW='\033[1;33m'
RED='\e[31m'
GREEN='\e[32m'
CYAN='\033[1;36m'
RESET='\e[0m'

# Default configuration file
CONFIG_FILE="config.yaml"

# Function to display the help message
show_help() {
    echo -e "\nUsage: $0 [options]"
    echo -e "\nOptions:"
    echo -e "  -c, --config <config_file>   Specify the YAML configuration file (default: config.yaml)."
    echo -e "  -list                        Check SSH connectivity to nodes in the config file."
    echo -e "  -h, --help                   Display this help message."
}

# Parse the YAML file to extract compute nodes
read_config_file() {
    if [[ ! -f "$CONFIG_FILE" ]]; then
        echo -e "${RED}Error: Configuration file '$CONFIG_FILE' not found.${RESET}"
        exit 1
    fi

    SLAVE_NODES=($(sed -n '/^compute_nodes:/,/^[^ ]/{/^  - /s/^  - //p}' "$CONFIG_FILE"))
    if [ ${#SLAVE_NODES[@]} -eq 0 ]; then
        echo -e "${RED}Error: No compute nodes specified in the configuration file.${RESET}"
        exit 1
    fi
}

# Function to check SSH connectivity to a single node
check_node() {
    NODE=$1
    if ssh -o BatchMode=yes -o ConnectTimeout=2 -o StrictHostKeyChecking=no -o PasswordAuthentication=no "${NODE}" "touch ~/FROM_MASTER.txt" &>/dev/null; then
        echo -e "└──$NODE: ${GREEN}Successfully Executed${RESET}"
    else
        echo -e "└──$NODE: ${RED}Failed to Connect${RESET}"
    fi
}

# Function to iterate over all nodes and check SSH connectivity
run() {
    read_config_file
    echo -e "${CYAN}${HOSTNAME}${RESET}[${YELLOW}SSH Status Check${RESET}]"
    for NODE in "${SLAVE_NODES[@]}"; do
        check_node "$NODE" &
    done
    wait
}

# Parse command-line arguments
while [[ "$#" -gt 0 ]]; do
    case $1 in
        -c|--config)
            CONFIG_FILE="$2"
            shift
            ;;
        -list)
            run
            ;;
        -h|--help)
            show_help
            exit 0
            ;;
        *)
            echo -e "${RED}Error: Unknown parameter passed: $1${RESET}"
            show_help
            exit 1
            ;;
    esac
    shift
done

```

<br>
<br>




## Final Testing
 **Run the script as follows:**

1. List Nodes:

```yml
./ssh_cli.sh -list        # Default demo file  or update this file as per your Cluster Nodes IPs
```

2. Specify a Custom Config File:

```yml
./ssh_cli.sh -c custom_config.yaml -list          # OR create new file ,also update in main script
```

3. View Help:

```yml
./ssh_cli.sh -h
```
 
 *If all prerequisites are met, the script should execute without errors.*










<br>
<br>


## Use Cases
1. Cluster Node Monitoring

     - Determine which nodes in a cluster are operational and ready for tasks.
     - Helps administrators quickly identify connectivity issues, reducing troubleshooting time.

2. Pre-deployment Checks

    - Verify SSH connectivity to all nodes before deploying distributed applications or workloads.
    - Ensures that compute nodes are properly configured and reachable.

3. Configuration Validation

    - Test if newly added nodes to a cluster are accessible via SSH.
    - Validate changes to network or firewall settings.

4. Automated Maintenance

    - Perform automated maintenance tasks (e.g., file transfers, script execution) by ensuring connectivity in advance.
    - Helps in scenarios where scripts require successful SSH communication.

5. Disaster Recovery

    - Quickly identify unreachable nodes during network outages or cluster failures.
    - Prioritize recovery actions based on node accessibility.

6. Integration with Other Tools

    - Serve as a base for more complex tools that require periodic node health checks or status reporting.
    - Output can be integrated with monitoring systems like Prometheus or custom dashboards.

### Why Use This Script?
   - Simplicity: No additional software is required; it uses standard Bash utilities.
   - Efficiency: Lightweight design ensures minimal resource usage, even in large clusters.
   - Flexibility: Nodes are defined in a YAML configuration file, making it easy to customize.
   - Automation: Can be scheduled with cron jobs or integrated into automated workflows for continuous monitoring.

   <br>
   
   **This script is ideal for system administrators, DevOps engineers, and cluster operators who require a reliable and straightforward way to monitor the connectivity of nodes in distributed environments.**









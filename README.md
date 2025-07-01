# Installation

## macOS Installation Guide

### Prerequisites

1. **Install uv** (Python package manager):
   ```bash
   brew install uv
   ```

### Install the Vast.ai MCP Server

1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-repo/vastai-mcp.git
   cd vastai-mcp
   ```

2. **Install dependencies using uv**:
   ```bash
   uv sync
   ```

3. **Install the server as a tool**:
   ```bash
   uv tool install -e .   # Install from current directory
   ```

### Configuration

1. **Get your Vast.ai API key**:
   - Log in to [console.vast.ai](https://console.vast.ai)
   - Go to Account > API Keys
   - Create or copy your API key


3. **Configure MCP client** (for Claude Desktop or other MCP clients):
   
   Update your MCP configuration file (`~/.cursor/mcp.json` for Cursor):
   ```json
   {
     "mcpServers": {
        "vast-ai": {
          "command": "uv",
          "args": [
              "run",
              "vast-mcp-server"
          ],
          "env": {
              "VAST_API_KEY": "your_vast_api_key_here",
              "SSH_KEY_FILE": "~/.ssh/id_rsa",
              "SSH_KEY_PUBLIC_FILE": "~/.ssh/id_rsa.pub"
          }
        }
     }
   }
   ```

### Verify Installation

1. **Test the server directly**:
   ```bash
   uv tool run vast-mcp-server --help
   ```

### SSH Key Setup (Recommended)

For full functionality, ensure you have SSH keys set up:

1. **Generate SSH key pair** (if you don't have one):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

2. **Verify key files exist**:
   ```bash
   ls -la ~/.ssh/id_rsa*
   ```
   You should see both `id_rsa` (private) and `id_rsa.pub` (public) files.

### Troubleshooting

- **Permission denied**: Make sure your SSH key has correct permissions:
  ```bash
  chmod 600 ~/.ssh/id_rsa
  chmod 644 ~/.ssh/id_rsa.pub
  ```

- **API key issues**: Verify your API key is correct and has proper permissions on Vast.ai

- **Network issues**: Ensure you can reach `console.vast.ai` from your network


# Vast.ai MCP Server Usage Guide

This document describes how to use the Vast.ai MCP (Model Context Protocol) server to interact with the Vast.ai GPU cloud platform.

## Available Tools

This server provides **23 tools** for managing Vast.ai GPU instances:

### 1. show_user_info()
Show current user information and account balance.

**Returns:**
- Username, email, account balance, user ID
- SSH key information (if available)
- Total spent amount

### 2. show_instances(owner: str = "me")
Show user's instances (running, stopped, etc.)

**Parameters:**
- `owner` (optional): Owner of instances to show (default: "me")

**Returns:**
- List of all instances with their details:
  - Instance ID and status
  - Label and machine ID
  - GPU type and specifications
  - Hourly cost
  - Docker image
  - Public IP address (if available)
  - Creation date

### 3. search_offers(query: str = "", limit: int = 20, order: str = "score-")
Search for available GPU offers/machines to rent.

**Parameters:**
- `query` (optional): Search query in key=value format (e.g., "gpu_name=RTX_4090 num_gpus=2")
- `limit` (optional): Maximum number of results to return (default: 20)
- `order` (optional): Sort order, append '-' for descending (default: "score-")

**Returns:**
- List of available offers with:
  - Offer ID
  - GPU specifications (name, count)
  - CPU and RAM details
  - Storage space
  - Hourly cost
  - Location and reliability score
  - CUDA version
  - Internet speeds

**Example queries:**
- `"gpu_name=RTX_4090"` - Search for RTX 4090 GPUs
- `"num_gpus=2 cpu_ram>=32"` - Search for dual GPU setups with 32GB+ RAM

### 4. create_instance(offer_id: int, image: str, disk: float = 10.0, ssh: bool = False, jupyter: bool = False, direct: bool = False, env: str = "", label: str = "", bid_price: float = None)
Create a new instance from an offer.

**Parameters:**
- `offer_id`: ID of the offer to rent (from search_offers)
- `image`: Docker image to run (e.g., "pytorch/pytorch:latest")
- `disk` (optional): Disk size in GB (default: 10.0)
- `ssh` (optional): Enable SSH access (default: False)
- `jupyter` (optional): Enable Jupyter notebook (default: False)
- `direct` (optional): Use direct connections (default: False)
- `env` (optional): Environment variables as dict (default: None)
- `label` (optional): Label for the instance
- `bid_price` (optional): Bid price for interruptible instances

**Returns:**
- Success message with instance ID or error details

**Example:**
```
create_instance(
    offer_id=12345,
    image="pytorch/pytorch:latest",
    disk=40.0,
    ssh=True,
    direct=True,
    env={"JUPYTER_ENABLE_LAB": "yes"},
    label="My PyTorch Training"
)
```

### 5. destroy_instance(instance_id: int)
Destroy an instance, completely removing it from the system. Don't need to stop it first.

**Parameters:**
- `instance_id`: ID of the instance to destroy

**Returns:**
- Success/failure message

### 6. start_instance(instance_id: int)
Start a stopped instance.

**Parameters:**
- `instance_id`: ID of the instance to start

**Returns:**
- Success/failure message

### 7. stop_instance(instance_id: int)
Stop a running instance (without destroying it).

**Parameters:**
- `instance_id`: ID of the instance to stop

**Returns:**
- Success/failure message

### 8. search_volumes(query: str = "", limit: int = 20)
Search for available storage volume offers.

**Parameters:**
- `query` (optional): Search query in key=value format
- `limit` (optional): Maximum number of results to return (default: 20)

**Returns:**
- List of available volume offers with:
  - Volume offer ID
  - Storage capacity
  - Cost per GB per month
  - Location and reliability
  - Disk bandwidth
  - Internet speeds

### 9. label_instance(instance_id: int, label: str)
Set a label on an instance for easier identification.

**Parameters:**
- `instance_id`: ID of the instance to label
- `label`: Label text to set

**Returns:**
- Success/failure message

### 10. launch_instance_workflow(gpu_name: str, num_gpus: int, image: str, region: str = "", disk: float = 16.0, ssh: bool = True, jupyter: bool = False, direct: bool = True, label: str = "")
Launch the top instance from search offers based on given parameters (streamlined alternative to create_instance).

**Parameters:**
- `gpu_name`: Name of GPU model (e.g., "RTX_4090")
- `num_gpus`: Number of GPUs required
- `image`: Docker image to run
- `region` (optional): Geographical region preference
- `disk` (optional): Disk size in GB (default: 16.0)
- `ssh` (optional): Enable SSH access (default: True)
- `jupyter` (optional): Enable Jupyter notebook (default: False)
- `direct` (optional): Use direct connections (default: True)
- `label` (optional): Label for the instance

**Returns:**
- Success message with instance details or error

**Example:**
```
launch_instance_workflow(
    gpu_name="RTX_4090",
    num_gpus=2,
    image="pytorch/pytorch:latest",
    region="North_America",
    disk=40.0,
    ssh=True,
    direct=True,
    label="My Training Job"
)
```

### 11. prepay_instance(instance_id: int, amount: float)
Deposit credits into a reserved instance for discounted rates.

**Parameters:**
- `instance_id`: ID of the instance to prepay
- `amount`: Amount of credits to deposit

**Returns:**
- Details about discount rate and coverage period

### 12. reboot_instance(instance_id: int)
Reboot an instance (stop/start) without losing GPU priority.

**Parameters:**
- `instance_id`: ID of the instance to reboot

**Returns:**
- Success/failure message

### 13. recycle_instance(instance_id: int)
Recycle an instance (destroy/create from newly pulled image) without losing GPU priority.

**Parameters:**
- `instance_id`: ID of the instance to recycle

**Returns:**
- Success/failure message

### 14. show_instance(instance_id: int)
Show detailed information about a specific instance.

**Parameters:**
- `instance_id`: ID of the instance to show

**Returns:**
- Detailed instance information including:
  - Status and specifications
  - Connection details (IP, SSH, Jupyter)
  - Cost and runtime information
  - Configuration details

### 15. logs(instance_id: int, tail: str = "1000", filter_text: str = "", daemon_logs: bool = False)
Get logs for an instance.

**Parameters:**
- `instance_id`: ID of the instance to get logs for
- `tail` (optional): Number of lines from end of logs (default: "1000")
- `filter_text` (optional): Grep filter for log entries
- `daemon_logs` (optional): Get daemon system logs instead of container logs

**Returns:**
- Instance logs text or status message

### 16. attach_ssh(instance_id: int)
Attach an SSH key to an instance for secure access.

**Parameters:**
- `instance_id`: ID of the instance to attach SSH key to

**Returns:**
- Success/failure message

**Examples:**
```python
# Attach SSH key from configured public key file
attach_ssh(12345)
```

**Notes:**
- Uses the SSH public key file configured in SSH_KEY_PUBLIC_FILE environment variable
- Only public SSH keys are accepted (not private keys)
- SSH key must start with 'ssh-' prefix (e.g., ssh-rsa, ssh-ed25519)
- After attaching, you can SSH to the instance using the corresponding private key

### 17. search_templates()
Search for available templates on Vast.ai.

**Parameters:**
- None

**Returns:**
- List of available templates with:
  - Template ID and name
  - Docker image
  - Description (if available)
  - Environment variables
  - Run type configuration
  - SSH and Jupyter settings

**Example:**
```python
# Get all available templates
search_templates()
```

**Notes:**
- Templates are pre-configured environments that simplify instance creation
- Templates may include specific Docker images, environment setups, and startup scripts

### 18. execute_command(instance_id: int, command: str)
Execute a (constrained) remote command only available on stopped instances. Use ssh to run commands on running instances.

**Parameters:**
- `instance_id`: ID of the instance to execute command on
- `command`: Command to execute (limited to ls, rm, du)

**Returns:**
- Command output or status message

**Available commands:**
- `ls`: List directory contents
- `rm`: Remove files or directories  
- `du`: Summarize device usage for a set of files

**Examples:**
```python
# List directory contents
execute_command(12345, "ls -l -o -r")

# Remove files
execute_command(12345, "rm -r home/delete_this.txt")

# Check disk usage
execute_command(12345, "du -d2 -h")
```

**Notes:**
- Only works on stopped instances
- For running instances, use ssh_execute_command instead
- Limited to specific safe commands for security

### 19. ssh_execute_command(remote_host: str, remote_user: str, remote_port: int, command: str)
Execute a command on a remote host via SSH.

**Parameters:**
- `remote_host`: The hostname or IP address of the remote server
- `remote_user`: The username to connect as (e.g., 'root', 'ubuntu', 'ec2-user')
- `remote_port`: The SSH port number (typically 22 or custom port like 34608)
- `command`: The command to execute on the remote host

**Returns:**
- Command output with exit status, stdout, and stderr

**Example:**
```python
# Execute a command on a running instance
ssh_execute_command(
    remote_host="116.43.148.85",
    remote_user="root", 
    remote_port=26378,
    command="nvidia-smi"
)
```

**Notes:**
- Works with any SSH-accessible server, not just Vast.ai instances
- Uses the SSH private key file configured in SSH_KEY_FILE environment variable
- Automatically handles different SSH key types (RSA, Ed25519, ECDSA, DSS)
- Returns detailed output including exit status and both stdout/stderr

### 20. ssh_execute_background_command(remote_host: str, remote_user: str, remote_port: int, command: str, task_name: str = None)
Execute a long-running command in the background on a remote host via SSH using nohup.

**Parameters:**
- `remote_host`: The hostname or IP address of the remote server
- `remote_user`: The username to connect as (e.g., 'root', 'ubuntu', 'ec2-user')
- `remote_port`: The SSH port number (typically 22 or custom port like 34608)
- `command`: The command to execute in the background
- `task_name` (optional): Optional name for the task (for easier identification)

**Returns:**
- Task information including task ID, process ID, and log file path

**Example:**
```python
# Start a long-running training job
ssh_execute_background_command(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    command="python train.py --epochs 100",
    task_name="training_job"
)
```

**Notes:**
- Returns task_id and process_id for monitoring
- Creates log files on the remote server for output capture
- Use ssh_check_background_task to monitor progress
- Use ssh_kill_background_task to stop if needed

### 21. ssh_check_background_task(remote_host: str, remote_user: str, remote_port: int, task_id: str, process_id: int, tail_lines: int = 50)
Check the status of a background SSH task and get its output.

**Parameters:**
- `remote_host`: The hostname or IP address of the remote server
- `remote_user`: The username to connect as
- `remote_port`: The SSH port number
- `task_id`: The task ID returned by ssh_execute_background_command
- `process_id`: The process ID returned by ssh_execute_background_command
- `tail_lines` (optional): Number of recent log lines to show (default: 50)

**Returns:**
- Status report with process status, log output, and progress information

**Example:**
```python
# Check on a background task
ssh_check_background_task(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    task_id="training_job_a1b2c3d4",
    process_id=12345,
    tail_lines=100
)
```

**Notes:**
- Shows whether the task is still running or completed
- Displays recent log output from the task
- Provides total log line count for progress indication

### 22. ssh_kill_background_task(remote_host: str, remote_user: str, remote_port: int, task_id: str, process_id: int)
Kill a running background SSH task.

**Parameters:**
- `remote_host`: The hostname or IP address of the remote server
- `remote_user`: The username to connect as
- `remote_port`: The SSH port number
- `task_id`: The task ID returned by ssh_execute_background_command
- `process_id`: The process ID returned by ssh_execute_background_command

**Returns:**
- Status of the kill operation and cleanup results

**Example:**
```python
# Stop a background task
ssh_kill_background_task(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    task_id="training_job_a1b2c3d4",
    process_id=12345
)
```

**Notes:**
- Attempts graceful termination first, then force kill if necessary
- Automatically cleans up temporary log and PID files
- Safe to call even if the process has already completed

### 23. configure_mcp_rules(auto_attach_ssh: bool = None, auto_label: bool = None, wait_for_ready: bool = None, label_prefix: str = None)
Configure MCP automation rules that control automatic behaviors during instance creation.

**Parameters:**
- `auto_attach_ssh` (optional): Enable/disable automatic SSH key attachment for SSH/Jupyter instances
- `auto_label` (optional): Enable/disable automatic instance labeling
- `wait_for_ready` (optional): Enable/disable waiting for instance readiness after creation
- `label_prefix` (optional): Set the prefix for automatic instance labels

**Returns:**
- Current configuration status and any changes made

**Example:**
```python
# Configure MCP rules
configure_mcp_rules(
    auto_attach_ssh=True,
    auto_label=True,
    label_prefix="my-project",
    wait_for_ready=True
)

# View current configuration
configure_mcp_rules()
```

**Notes:**
- These rules affect the behavior of create_instance and launch_instance_workflow
- Auto-attach SSH applies only when SSH or Jupyter is enabled
- Auto-labeling creates timestamps labels when no label is provided
- Wait for ready monitors instance status until it becomes "running"

## Configuration

### API Key Setup
The server requires a Vast.ai API key. You can configure it in several ways:

1. **Environment Variable:**
   ```bash
   export VAST_API_KEY="your_api_key_here"
   ```
   
2. **API Key File:**
   Create `~/.vastai_api_key` with your API key

3. **Hardcoded (for development):**
   The current server has a hardcoded API key for testing purposes

### Running the Server

   ```bash
# Run with default settings (localhost:8000)
python vast_mcp_server.py

# Run with custom host and port
python vast_mcp_server.py --host 0.0.0.0 --port 9000
   ```

## Common Workflows

### 1. Basic Instance Creation Workflow
```python
# 1. Check your account
show_user_info()

# 2. Search for available offers
search_offers("gpu_name=RTX_4090", limit=10)

# 3. Create instance from an offer
create_instance(
    offer_id=12345, 
    image="pytorch/pytorch:latest",
    disk=20.0,
    ssh=True,
    direct=True
)

# 4. Check instance status
  show_instances()
  ```

### 2. Instance Management
```python
# View all instances
show_instances()

# Stop an instance
stop_instance(instance_id=67890)

# Start it again later
start_instance(instance_id=67890)

# Permanently destroy when done
destroy_instance(instance_id=67890)
  ```

### 3. Finding Storage
```python
# Search for storage volumes
search_volumes("disk_space>=100", limit=5)
  ```

### 4. Advanced Instance Management
```python
# Launch instance with specific GPU requirements (streamlined approach)
launch_instance_workflow(
    gpu_name="RTX_4090",
    num_gpus=2,
    image="pytorch/pytorch:latest",
    region="North_America",
    disk=40.0,
    ssh=True,
    direct=True,
    label="Training Job"
)

# Get detailed information about an instance
show_instance(instance_id=12345)

# Set a label for easier identification
label_instance(instance_id=12345, label="Production Model")

# Get instance logs
logs(instance_id=12345, tail="500", filter_text="error")

# Reboot instance without losing GPU priority
reboot_instance(instance_id=12345)
```

### 5. Instance Monitoring and Maintenance
```python
# Monitor instance logs with filtering
logs(instance_id=12345, filter_text="WARNING|ERROR", tail="100")

# Check instance details
show_instance(instance_id=12345)

# Recycle instance to update to latest image
recycle_instance(instance_id=12345)

# Prepay for discounted rates
prepay_instance(instance_id=12345, amount=50.0)
```

### 6. SSH Access Management
```python
# Create instance with SSH enabled
create_instance(
    offer_id=12345,
    image="ubuntu:22.04",
    ssh=True,
    direct=True,
    label="SSH Server"
)

# Attach your SSH key for access
attach_ssh(instance_id=67890)

# Get instance details including SSH connection info
show_instance(instance_id=67890)

# Monitor instance through logs
logs(instance_id=67890, tail="50")
```

### 7. Template Browsing
```python
# Browse available templates
search_templates()
```

### 8. Instance Command Execution
```python
# For stopped instances, use constrained execute_command
stop_instance(instance_id=12345)

# Execute safe commands on stopped instance
execute_command(instance_id=12345, command="ls -la /workspace")
execute_command(instance_id=12345, command="du -sh /workspace")
execute_command(instance_id=12345, command="rm -rf /tmp/old_files")

# For running instances, use SSH commands
start_instance(instance_id=12345)

# Get instance connection details
instance_details = show_instance(instance_id=12345)
# Extract SSH host, port from the output

# Execute commands via SSH on running instance
ssh_execute_command(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    command="nvidia-smi"
)

# Check system resources
ssh_execute_command(
    remote_host="116.43.148.85",
    remote_user="root", 
    remote_port=26378,
    command="df -h && free -h && ps aux"
)
```

### 9. Background Task Management
```python
# Start a long-running training job in background
task_info = ssh_execute_background_command(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    command="python train.py --epochs 100 --batch-size 32",
    task_name="pytorch_training"
)

# Extract task_id and process_id from task_info output
# Format: "Task ID: pytorch_training_a1b2c3d4" and "Process ID: 12345"

# Monitor progress periodically
ssh_check_background_task(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    task_id="pytorch_training_a1b2c3d4",
    process_id=12345,
    tail_lines=100
)

# Stop the task if needed
ssh_kill_background_task(
    remote_host="116.43.148.85", 
    remote_user="root",
    remote_port=26378,
    task_id="pytorch_training_a1b2c3d4",
    process_id=12345
)
```

### 10. Complete ML Training Workflow
```python
# 1. Find and create a GPU instance
search_offers("gpu_name=RTX_4090", limit=5)

create_instance(
    offer_id=12345,
    image="pytorch/pytorch:latest",
    disk=50.0,
    ssh=True,
    direct=True,
    env={},
    label="ML Training"
)

# 2. Get connection details
instance_details = show_instance(instance_id=67890)

# 3. Set up the environment
ssh_execute_command(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    command="pip install wandb tensorboard"
)

# 4. Upload your training code (assume already done)
# 5. Start training in background
training_task = ssh_execute_background_command(
    remote_host="116.43.148.85",
    remote_user="root", 
    remote_port=26378,
    command="cd /workspace && python train.py --config config.yaml",
    task_name="main_training"
)

# 6. Monitor training progress
ssh_check_background_task(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    task_id="main_training_a1b2c3d4",
    process_id=12345,
    tail_lines=50
)

# 7. Check GPU utilization
ssh_execute_command(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    command="nvidia-smi"
)

# 8. When training is complete, save results
ssh_execute_command(
    remote_host="116.43.148.85",
    remote_user="root",
    remote_port=26378,
    command="tar -czf model_results.tar.gz /workspace/outputs"
)

# 9. Clean up
destroy_instance(instance_id=67890)
```

## Query Syntax

When searching for offers or volumes, you can use these operators:
- `=` or `==` - Equal to
- `!=` - Not equal to
- `>` - Greater than
- `>=` - Greater than or equal to
- `<` - Less than
- `<=` - Less than or equal to

**Example queries:**
- `"gpu_name=RTX_4090 num_gpus>=2"` - RTX 4090 with 2 or more GPUs
- `"cpu_ram>64 reliability2>=99"` - High RAM and reliability
- `"dph_total<=1.0"` - Cost under $1/hour

## Error Handling

All methods include error handling and will return descriptive error messages if:
- API key is missing or invalid
- Network connectivity issues occur
- Invalid parameters are provided
- Vast.ai API returns errors

Check the server logs for detailed error information during development.

# Python Scripting

[← Back to Scripting Index](README.md) | [← Back to Main Index](../../README.md)

This section covers Python scripting for DevOps automation.

---

<details>
<summary><strong>Q11: Why is Python preferred for DevOps automation?</strong></summary>

| Advantage | Description |
|-----------|-------------|
| **Readability** | Clean syntax, easy to maintain |
| **Rich ecosystem** | Extensive libraries for any task |
| **Cross-platform** | Works on Linux, macOS, Windows |
| **API integration** | Excellent HTTP/REST support |
| **Infrastructure tools** | Ansible, SaltStack, Fabric use Python |
| **Cloud SDKs** | boto3 (AWS), azure-sdk, google-cloud |

**Common DevOps Python Libraries:**

| Library | Purpose |
|---------|---------|
| `requests` | HTTP requests |
| `boto3` | AWS SDK |
| `paramiko` | SSH connections |
| `fabric` | Remote execution |
| `pyyaml` | YAML parsing |
| `jinja2` | Template rendering |
| `click` | CLI applications |
| `pytest` | Testing |
| `docker` | Docker SDK |
| `kubernetes` | Kubernetes SDK |

</details>

<details>
<summary><strong>Q12: Explain Python file handling for DevOps</strong></summary>

**Reading Files:**
```python
# Read entire file
with open('config.txt', 'r') as f:
    content = f.read()

# Read lines into list
with open('hosts.txt', 'r') as f:
    lines = f.readlines()

# Read line by line (memory efficient)
with open('large_log.txt', 'r') as f:
    for line in f:
        process(line.strip())
```

**Writing Files:**
```python
# Write to file
with open('output.txt', 'w') as f:
    f.write('Hello, World!\n')

# Append to file
with open('log.txt', 'a') as f:
    f.write(f'{datetime.now()}: Event occurred\n')
```

**Working with JSON:**
```python
import json

# Read JSON
with open('config.json', 'r') as f:
    config = json.load(f)

# Write JSON
with open('output.json', 'w') as f:
    json.dump(data, f, indent=2)

# Parse JSON string
data = json.loads('{"name": "value"}')
```

**Working with YAML:**
```python
import yaml

# Read YAML
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# Write YAML
with open('output.yaml', 'w') as f:
    yaml.dump(data, f, default_flow_style=False)
```

**Working with CSV:**
```python
import csv

# Read CSV
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], row['value'])

# Write CSV
with open('output.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['name', 'value'])
    writer.writeheader()
    writer.writerow({'name': 'foo', 'value': 'bar'})
```

</details>

<details>
<summary><strong>Q13: How do you make HTTP/API requests in Python?</strong></summary>

**Using requests library:**
```python
import requests

# GET request
response = requests.get('https://api.example.com/users')
if response.status_code == 200:
    users = response.json()
    
# GET with parameters
response = requests.get(
    'https://api.example.com/search',
    params={'q': 'python', 'limit': 10}
)

# POST request with JSON
response = requests.post(
    'https://api.example.com/users',
    json={'name': 'John', 'email': 'john@example.com'},
    headers={'Authorization': 'Bearer token123'}
)

# PUT request
response = requests.put(
    'https://api.example.com/users/1',
    json={'name': 'Updated Name'}
)

# DELETE request
response = requests.delete('https://api.example.com/users/1')

# Handling errors
try:
    response = requests.get(url, timeout=30)
    response.raise_for_status()  # Raise exception for 4xx/5xx
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

**API Wrapper Example:**
```python
import requests
from typing import Dict, Any

class GitHubAPI:
    def __init__(self, token: str):
        self.base_url = 'https://api.github.com'
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'token {token}',
            'Accept': 'application/vnd.github.v3+json'
        })
    
    def get_repos(self, org: str) -> list:
        response = self.session.get(f'{self.base_url}/orgs/{org}/repos')
        response.raise_for_status()
        return response.json()
    
    def create_issue(self, owner: str, repo: str, title: str, body: str) -> Dict:
        response = self.session.post(
            f'{self.base_url}/repos/{owner}/{repo}/issues',
            json={'title': title, 'body': body}
        )
        response.raise_for_status()
        return response.json()

# Usage
api = GitHubAPI(os.environ['GITHUB_TOKEN'])
repos = api.get_repos('mycompany')
```

</details>

<details>
<summary><strong>Q14: How do you execute shell commands from Python?</strong></summary>

**Using subprocess (recommended):**
```python
import subprocess

# Simple command
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)

# With shell=True (be careful with user input!)
result = subprocess.run('echo $HOME', shell=True, capture_output=True, text=True)

# Check for errors
result = subprocess.run(
    ['docker', 'build', '-t', 'myapp', '.'],
    capture_output=True,
    text=True,
    check=True  # Raises exception on non-zero exit
)

# Stream output in real-time
process = subprocess.Popen(
    ['kubectl', 'logs', '-f', 'pod-name'],
    stdout=subprocess.PIPE,
    text=True
)
for line in process.stdout:
    print(line, end='')

# Run with timeout
try:
    result = subprocess.run(
        ['long_running_command'],
        timeout=60,
        capture_output=True
    )
except subprocess.TimeoutExpired:
    print("Command timed out")
```

**Practical Example - Deploy Script:**
```python
import subprocess
import sys

def run_command(cmd: list[str], description: str) -> bool:
    print(f"→ {description}")
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode != 0:
        print(f"  ✗ Failed: {result.stderr}")
        return False
    print(f"  ✓ Success")
    return True

def deploy():
    steps = [
        (['docker', 'build', '-t', 'myapp:latest', '.'], 'Building Docker image'),
        (['docker', 'push', 'myapp:latest'], 'Pushing to registry'),
        (['kubectl', 'rollout', 'restart', 'deployment/myapp'], 'Restarting deployment'),
    ]
    
    for cmd, desc in steps:
        if not run_command(cmd, desc):
            sys.exit(1)
    
    print("\n✓ Deployment complete!")

if __name__ == '__main__':
    deploy()
```

</details>

<details>
<summary><strong>Q15: How do you handle exceptions in Python?</strong></summary>

**Try-Except Block:**
```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except (TypeError, KeyError) as e:
    print(f"Type or key error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
    raise  # Re-raise the exception
else:
    print("No exception occurred")
finally:
    cleanup()  # Always runs
```

**Custom Exceptions:**
```python
class DeploymentError(Exception):
    """Raised when deployment fails"""
    def __init__(self, message: str, stage: str):
        self.message = message
        self.stage = stage
        super().__init__(f"Deployment failed at {stage}: {message}")

def deploy(environment: str):
    try:
        build()
    except BuildError as e:
        raise DeploymentError(str(e), "build")
    
    try:
        push_to_registry()
    except RegistryError as e:
        raise DeploymentError(str(e), "push")

# Usage
try:
    deploy("production")
except DeploymentError as e:
    print(f"Failed at stage: {e.stage}")
    notify_team(e.message)
```

**Context Manager for Resource Management:**
```python
from contextlib import contextmanager

@contextmanager
def ssh_connection(host: str):
    conn = create_connection(host)
    try:
        yield conn
    finally:
        conn.close()

# Usage
with ssh_connection('server1') as conn:
    conn.execute('uptime')
```

</details>

<details>
<summary><strong>Q16: Explain Python regex for log parsing</strong></summary>

**Basic Regex Operations:**
```python
import re

text = "Error at 192.168.1.100: Connection refused at 2024-01-15 10:30:45"

# Search for pattern
match = re.search(r'\d+\.\d+\.\d+\.\d+', text)
if match:
    print(f"Found IP: {match.group()}")  # 192.168.1.100

# Find all matches
ips = re.findall(r'\d+\.\d+\.\d+\.\d+', text)

# Replace
cleaned = re.sub(r'\d+\.\d+\.\d+\.\d+', '[REDACTED]', text)

# Split
parts = re.split(r'\s+', text)

# Compile for reuse
IP_PATTERN = re.compile(r'(\d+\.\d+\.\d+\.\d+)')
match = IP_PATTERN.search(text)
```

**Log Parsing Example:**
```python
import re
from collections import Counter

# Apache log pattern
LOG_PATTERN = re.compile(
    r'(?P<ip>\d+\.\d+\.\d+\.\d+) - - '
    r'\[(?P<timestamp>[^\]]+)\] '
    r'"(?P<method>\w+) (?P<path>[^ ]+) [^"]*" '
    r'(?P<status>\d+) (?P<size>\d+)'
)

def parse_log(log_file: str):
    stats = {
        'total_requests': 0,
        'status_codes': Counter(),
        'top_ips': Counter(),
        'top_paths': Counter()
    }
    
    with open(log_file, 'r') as f:
        for line in f:
            match = LOG_PATTERN.match(line)
            if match:
                data = match.groupdict()
                stats['total_requests'] += 1
                stats['status_codes'][data['status']] += 1
                stats['top_ips'][data['ip']] += 1
                stats['top_paths'][data['path']] += 1
    
    return stats

# Usage
stats = parse_log('/var/log/apache2/access.log')
print(f"Total requests: {stats['total_requests']}")
print(f"Top 5 IPs: {stats['top_ips'].most_common(5)}")
```

</details>

<details>
<summary><strong>Q17: How do you use argparse for CLI tools?</strong></summary>

**Basic CLI Tool:**
```python
import argparse
import sys

def main():
    parser = argparse.ArgumentParser(
        description='Deploy application to environment',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog='''
Examples:
  %(prog)s deploy --env production
  %(prog)s rollback --env staging --version v1.2.3
        '''
    )
    
    # Subcommands
    subparsers = parser.add_subparsers(dest='command', required=True)
    
    # Deploy command
    deploy_parser = subparsers.add_parser('deploy', help='Deploy application')
    deploy_parser.add_argument(
        '--env', '-e',
        required=True,
        choices=['dev', 'staging', 'production'],
        help='Target environment'
    )
    deploy_parser.add_argument(
        '--dry-run',
        action='store_true',
        help='Simulate deployment without making changes'
    )
    deploy_parser.add_argument(
        '--version', '-v',
        default='latest',
        help='Version to deploy (default: latest)'
    )
    
    # Rollback command
    rollback_parser = subparsers.add_parser('rollback', help='Rollback to previous version')
    rollback_parser.add_argument('--env', '-e', required=True)
    rollback_parser.add_argument('--version', '-v', required=True)
    
    args = parser.parse_args()
    
    if args.command == 'deploy':
        deploy(args.env, args.version, args.dry_run)
    elif args.command == 'rollback':
        rollback(args.env, args.version)

def deploy(env: str, version: str, dry_run: bool):
    print(f"Deploying {version} to {env}")
    if dry_run:
        print("(Dry run - no changes made)")

def rollback(env: str, version: str):
    print(f"Rolling back {env} to {version}")

if __name__ == '__main__':
    main()
```

**Using Click (alternative):**
```python
import click

@click.group()
def cli():
    """Deployment CLI tool"""
    pass

@cli.command()
@click.option('--env', '-e', required=True, type=click.Choice(['dev', 'staging', 'prod']))
@click.option('--version', '-v', default='latest')
@click.option('--dry-run', is_flag=True)
def deploy(env, version, dry_run):
    """Deploy application to environment"""
    click.echo(f"Deploying {version} to {env}")

@cli.command()
@click.option('--env', '-e', required=True)
@click.option('--version', '-v', required=True)
def rollback(env, version):
    """Rollback to previous version"""
    click.echo(f"Rolling back {env} to {version}")

if __name__ == '__main__':
    cli()
```

</details>

<details>
<summary><strong>Q18: How do you set up logging in Python?</strong></summary>

**Basic Logging:**
```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()  # Console output
    ]
)

logger = logging.getLogger(__name__)

# Usage
logger.debug('Debug message')
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message')
logger.exception('Exception with traceback')  # Use in except block
```

**Production-Ready Logging:**
```python
import logging
import logging.handlers
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        return json.dumps(log_data)

def setup_logging(app_name: str, log_level: str = 'INFO'):
    logger = logging.getLogger(app_name)
    logger.setLevel(getattr(logging, log_level))
    
    # Console handler
    console = logging.StreamHandler()
    console.setFormatter(JSONFormatter())
    logger.addHandler(console)
    
    # File handler with rotation
    file_handler = logging.handlers.RotatingFileHandler(
        f'{app_name}.log',
        maxBytes=10_000_000,  # 10MB
        backupCount=5
    )
    file_handler.setFormatter(JSONFormatter())
    logger.addHandler(file_handler)
    
    return logger

# Usage
logger = setup_logging('myapp', 'INFO')
logger.info('Application started', extra={'version': '1.0.0'})
```

</details>

<details>
<summary><strong>Q19: Explain Python virtual environments and package management</strong></summary>

**Creating Virtual Environments:**
```bash
# Using venv (built-in)
python -m venv myenv
source myenv/bin/activate  # Linux/macOS
myenv\Scripts\activate     # Windows

# Deactivate
deactivate
```

**Package Management with pip:**
```bash
# Install package
pip install requests

# Install specific version
pip install requests==2.28.0

# Install from requirements
pip install -r requirements.txt

# Generate requirements
pip freeze > requirements.txt

# Upgrade package
pip install --upgrade requests

# Uninstall
pip uninstall requests
```

**requirements.txt Best Practices:**
```
# Pin exact versions for reproducibility
requests==2.28.1
boto3==1.26.0
pyyaml==6.0

# Or use ranges
requests>=2.28,<3.0

# Development dependencies (use separate file)
# requirements-dev.txt
pytest>=7.0
black>=22.0
mypy>=0.990
```

**Using Poetry (modern approach):**
```bash
# Initialize project
poetry init

# Add dependency
poetry add requests
poetry add pytest --group dev

# Install all dependencies
poetry install

# Run script
poetry run python myscript.py

# Build and publish
poetry build
poetry publish
```

**pyproject.toml Example:**
```toml
[tool.poetry]
name = "mydevops-tool"
version = "1.0.0"
description = "DevOps automation tool"

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.28"
boto3 = "^1.26"
click = "^8.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0"
black = "^22.0"

[tool.poetry.scripts]
mytool = "mydevops_tool.cli:main"
```

</details>

<details>
<summary><strong>Q20: Practical Python DevOps Scripts</strong></summary>

**AWS EC2 Instance Manager:**
```python
import boto3
from typing import List, Dict

class EC2Manager:
    def __init__(self, region: str = 'us-east-1'):
        self.ec2 = boto3.client('ec2', region_name=region)
    
    def list_instances(self, state: str = None) -> List[Dict]:
        filters = []
        if state:
            filters.append({'Name': 'instance-state-name', 'Values': [state]})
        
        response = self.ec2.describe_instances(Filters=filters)
        
        instances = []
        for reservation in response['Reservations']:
            for instance in reservation['Instances']:
                name = next(
                    (t['Value'] for t in instance.get('Tags', []) if t['Key'] == 'Name'),
                    'N/A'
                )
                instances.append({
                    'id': instance['InstanceId'],
                    'name': name,
                    'state': instance['State']['Name'],
                    'type': instance['InstanceType'],
                    'public_ip': instance.get('PublicIpAddress', 'N/A')
                })
        return instances
    
    def start_instances(self, instance_ids: List[str]):
        self.ec2.start_instances(InstanceIds=instance_ids)
    
    def stop_instances(self, instance_ids: List[str]):
        self.ec2.stop_instances(InstanceIds=instance_ids)
```

**Kubernetes Pod Monitor:**
```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException
import time

def monitor_pods(namespace: str = 'default'):
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    while True:
        try:
            pods = v1.list_namespaced_pod(namespace)
            print(f"\n{'Pod Name':<40} {'Status':<15} {'Restarts':<10}")
            print("-" * 65)
            
            for pod in pods.items:
                name = pod.metadata.name
                status = pod.status.phase
                restarts = sum(
                    cs.restart_count 
                    for cs in (pod.status.container_statuses or [])
                )
                
                if status != 'Running':
                    status = f"⚠️ {status}"
                elif restarts > 0:
                    status = f"⚠️ {status} ({restarts} restarts)"
                else:
                    status = f"✓ {status}"
                
                print(f"{name:<40} {status:<15} {restarts:<10}")
            
            time.sleep(10)
            
        except ApiException as e:
            print(f"Error: {e}")
            break
        except KeyboardInterrupt:
            break
```

**Slack Alert Sender:**
```python
import requests
import os
from datetime import datetime

class SlackNotifier:
    def __init__(self, webhook_url: str = None):
        self.webhook_url = webhook_url or os.environ.get('SLACK_WEBHOOK_URL')
    
    def send_message(self, message: str, channel: str = None):
        payload = {'text': message}
        if channel:
            payload['channel'] = channel
        
        response = requests.post(self.webhook_url, json=payload)
        response.raise_for_status()
    
    def send_deployment_notification(
        self, 
        app: str, 
        version: str, 
        environment: str,
        status: str
    ):
        color = '#36a64f' if status == 'success' else '#ff0000'
        
        payload = {
            'attachments': [{
                'color': color,
                'title': f'Deployment {status.upper()}',
                'fields': [
                    {'title': 'Application', 'value': app, 'short': True},
                    {'title': 'Version', 'value': version, 'short': True},
                    {'title': 'Environment', 'value': environment, 'short': True},
                    {'title': 'Time', 'value': datetime.now().strftime('%Y-%m-%d %H:%M:%S'), 'short': True}
                ]
            }]
        }
        
        requests.post(self.webhook_url, json=payload)

# Usage
notifier = SlackNotifier()
notifier.send_deployment_notification('myapp', 'v1.2.3', 'production', 'success')
```

</details>

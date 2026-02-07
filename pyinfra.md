 PyInfra Complete Guide for LLMs
 
 Installation:
 pip install pyinfra
 
 Basic Structure:
 - Inventory: Defines target hosts and groups
 - State: Runtime context containing host data  
 - Operations: Actions performed on hosts
 - Facts: Information about target systems
 
 Inventory Examples:
 
 1. Simple hosts file (hosts):
 server1.example.com
 server2.example.com
 
 [webservers]
 web1.example.com
 web2.example.com
 
 [databases]  
 db1.example.com
 
 2. Python inventory (inventory.py):
 hosts = ['server1.example.com', 'server2.example.com']
 
 Core Operations:
 
 System Operations:
 from pyinfra.operations import server
 
 server.shell(name='Run command', commands=['ls -la'])
 server.user(name='Create user', user='deploy', ensure_home=True)
 server.group(name='Create group', group='admin')
 server.systemd(name='Start service', service='nginx', running=True)
 server.cron(name='Add cron', command='/script.sh', minute='0', hour='2')
 
 Package Management:
 from pyinfra.operations import apt, yum, dnf
 
 apt.packages(name='Install packages', packages=['nginx'], update=True)
 apt.update(name='Update apt cache')
 yum.packages(name='Install RHEL packages', packages=['httpd'])
 
 File Operations:
 from pyinfra.operations import files
 
 files.directory(name='Create dir', path='/app', user='app', mode='755')
 files.put(name='Copy file', src='local/file', dest='/remote/file')
 files.download(name='Download file', src='url', dest='/path')
 files.template(name='Template file', src='template.j2', dest='/path')
 files.line(name='Add line', path='/file', line='content')
 files.replace(name='Replace text', path='/file', match='old', replace='new')
 files.link(name='Create symlink', path='/link', target='/target')
 
 Git Operations:
 from pyinfra.operations import git
 
 git.repo(name='Clone repo', src='url', dest='/path', branch='main')
 git.deploy(name='Deploy repo', src='url', dest='/path', branch='main')
 
 Database Operations:
 from pyinfra.operations import postgresql, mysql
 
 postgresql.database(name='Create DB', database='mydb')
 mysql.user(name='Create user', user='user', password='pass')
 
 Accessing Facts:
 from pyinfra import host
 
 host.facts.os              # Operating system
 host.facts.linux_name      # Linux distribution name
 host.facts.arch           # Architecture
 host.facts.memory_mb      # Memory in MB
 host.facts.users          # List of users
 host.facts.packages       # Installed packages
 
 Conditional Operations:
 
 if host.facts.linux_name == 'Ubuntu':
     apt.packages(name='Ubuntu packages', packages=['pkg'])
 elif host.facts.linux_name == 'CentOS':
     yum.packages(name='RHEL packages', packages=['pkg'])
 
 server.shell(
     name='Conditional command',
     commands=['echo "Ubuntu"'],
     when=lambda: host.facts.linux_name == 'Ubuntu'
 )
 
 Environment Variables:
 import os
 ENV = os.environ.get('ENVIRONMENT', 'development')
 
 if ENV == 'production':
     # Production-specific operations
     pass
 
 Command Line Usage:
 
 pyinfra inventory.py deploy.py                    # Basic deployment
 pyinfra --sudo --user deploy inventory.py script.py  # With sudo
 pyinfra --dry inventory.py script.py             # Dry run
 pyinfra --serial inventory.py script.py          # Serial execution
 pyinfra --limit server1 inventory.py script.py   # Specific host
 pyinfra --debug inventory.py script.py           # Debug mode
 pyinfra --yes inventory.py script.py             # Auto confirm
 
 Configuration:
 from pyinfra.api import Config
 
 config = Config(
     ssh_user='deploy',
     ssh_key='~/.ssh/key',
     sudo=True,
     timeout=10,
     parallel=5
 )
 
 Facts Available:
 - host.facts.os
 - host.facts.linux_name  
 - host.facts.os_version_info
 - host.facts.arch
 - host.facts.memory_mb
 - host.facts.disk_usage
 - host.facts.users
 - host.facts.groups
 - host.facts.packages
 - host.facts.files
 - host.facts.service('name')
 - host.facts.processes
 
 Example Deployment Function:
 
 def deploy_app(state, host):
     # Update system
     apt.update(state=state, host=host, name='Update packages')
     
     # Install dependencies based on OS
     if host.facts.linux_name == 'Ubuntu':
         apt.packages(state=state, host=host, name='Install packages', packages=['nginx'])
     else:
         yum.packages(state=state, host=host, name='Install packages', packages=['nginx'])
     
     # Deploy application
     git.deploy(state=state, host=host, name='Deploy app', src='repo_url', dest='/app')
     
     # Configure and start service
     server.systemd(state=state, host=host, name='Start service', service='nginx', running=True)
 
 Key Parameters for All Operations:
 - state: Required for all operations
 - host: Required for all operations  
 - name: Description of the operation
 - sudo: Whether to run with sudo
 - user: User to run as
 - ignore_errors: Continue if operation fails
 - when: Conditional execution
 
 Error Handling:
 try:
     server.shell(state=state, host=host, name='Command', commands=['cmd'])
 except Exception as e:
     print(f"Error: {e}")
 
 Best Practices:
 - Use templates for configuration files
 - Separate inventory from operations
 - Use environment variables for configuration
 - Test with --dry flag first
 - Handle different OS types with conditions
 - Use facts to adapt to host characteristics
 - Group related operations logically
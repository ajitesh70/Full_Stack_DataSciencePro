**User & Project Management System - Ansible Assignment**

**EC2 Server:** web1 (51.20.125.245)
**Inventory:** /etc/ansible/hosts.ini
**Key:** \~/.ssh/vm2.pem

---

### Step 0: Test connectivity

```bash
ansible all -i /etc/ansible/hosts.ini -m ping -b
```

### Step 1: Create Groups

```bash
ansible all -i /etc/ansible/hosts.ini -b -m group -a "name=dev-team state=present"
ansible all -i /etc/ansible/hosts.ini -b -m group -a "name=devops-team state=present"
ansible all -i /etc/ansible/hosts.ini -b -m group -a "name=admin-group state=present"
```

### Step 2: Create Users

**Dev Team**

```bash
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=dev1 groups=dev-team uid=2000 shell=/bin/bash create_home=yes state=present password='!'"
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=dev2 groups=dev-team uid=2001 shell=/bin/bash create_home=yes state=present password='!'"
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=dev3 groups=dev-team uid=2002 shell=/bin/bash create_home=yes state=present password='!'"
```

**DevOps Team**

```bash
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=ops1 groups=devops-team uid=2003 shell=/bin/zsh create_home=yes state=present password='!'"
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=ops2 groups=devops-team uid=2004 shell=/bin/zsh create_home=yes state=present password='!'"
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=ops3 groups=devops-team uid=2005 shell=/bin/zsh create_home=yes state=present password='!'"
```

**Admin Team**

```bash
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=admin1 groups=admin-group uid=2006 shell=/bin/bash create_home=yes state=present password='!'"
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=admin2 groups=admin-group uid=2007 shell=/bin/bash create_home=yes state=present password='!'"
ansible all -i /etc/ansible/hosts.ini -b -m user -a "name=admin3 groups=admin-group uid=2008 shell=/bin/bash create_home=yes state=present password='!'"
```

### Step 3: Create Personal Workspaces

```bash
for u in dev1 dev2 dev3 ops1 ops2 ops3 admin1 admin2 admin3; do
  ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=/home/$u/workspace state=directory owner=$u group=$u mode=0700"
done
```

### Step 4: Set Password Expiry Policy

```bash
for u in dev1 dev2 dev3 ops1 ops2 ops3 admin1 admin2 admin3; do
  ansible all -i /etc/ansible/hosts.ini -b -m command -a "chage -M 90 -m 7 -W 7 $u"
done
```

### Step 5: Configure Sudo Access

**Admin-group (full sudo)**

```bash
ansible all -i /etc/ansible/hosts.ini -b -m copy -a "dest=/etc/sudoers.d/admin_group content='%admin-group ALL=(ALL) NOPASSWD:ALL\n' mode=0440"
```

**DevOps-team (limited sudo)**

```bash
ansible all -i /etc/ansible/hosts.ini -b -m copy -a "dest=/etc/sudoers.d/devops_group content='%devops-team ALL=(ALL) NOPASSWD: /bin/systemctl, /usr/sbin/service, /bin/journalctl, /usr/bin/tail\n' mode=0440"
```

### Step 6: Create Base Directories

```bash
BASE=/srv/projects_management
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE state=directory recurse=yes mode=0755"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/teams state=directory mode=0755"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/projects state=directory mode=0775"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/shared_resources state=directory mode=0775"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/archive state=directory mode=0555"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/admin_area state=directory owner=root group=admin-group mode=0770"
```

### Step 7: Team Directories

```bash
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/teams/dev-team state=directory owner=root group=dev-team mode=0775"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/teams/devops-team state=directory owner=root group=devops-team mode=0775"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/teams/admin-group state=directory owner=root group=admin-group mode=0775"
```

### Step 8: Project Directories

```bash
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/projects/WebApp state=directory mode=0775"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/projects/API state=directory mode=0775"
ansible all -i /etc/ansible/hosts.ini -b -m file -a "path=$BASE/projects/Mobile state=directory mode=0775"
```

### Step 9: Install ACL Package

```bash
ansible all -i /etc/ansible/hosts.ini -b -m package -a "name=acl state=present"
```

### Step 10: Set ACL Permissions

```bash
# Team directories
ansible all -i /etc/ansible/hosts.ini -b -m command -a "setfacl -R -m g:dev-team:rwx -m g:devops-team:r-x -m g:admin-group:r-x $BASE/teams/dev-team"
ansible all -i /etc/ansible/hosts.ini -b -m command -a "setfacl -R -m g:devops-team:rwx -m g:dev-team:r-x -m g:admin-group:r-x $BASE/teams/devops-team"
ansible all -i /etc/ansible/hosts.ini -b -m command -a "setfacl -R -m g:admin-group:rwx -m g:dev-team:r-x -m g:devops-team:r-x $BASE/teams/admin-group"

# Shared resources
ansible all -i /etc/ansible/hosts.ini -b -m command -a "setfacl -R -m g:dev-team:rwx -m g:devops-team:rwx -m g:admin-group:rwx $BASE/shared_resources"

# Archive
ansible all -i /etc/ansible/hosts.ini -b -m command -a "setfacl -R -m g:dev-team:r-x -m g:devops-team:r-x -m g:admin-group:r-x $BASE/archive"

# Admin area
ansible all -i /etc/ansible/hosts.ini -b -m command -a "setfacl -R -m g:admin-group:rwx $BASE/admin_area"
```

### Verification Commands

```bash
ansible all -i /etc/ansible/hosts.ini -b -m command -a "getfacl $BASE/projects/WebApp"
ansible all -i /etc/ansible/hosts.ini -b -m command -a "id dev1"
ansible all -i
```
<img width="735" height="735" alt="Screenshot 2025-09-19 095310" src="https://github.com/user-attachments/assets/3bb7664e-dc57-403a-9e35-fdebf857b0e0" />


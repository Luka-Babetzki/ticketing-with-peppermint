# Ticketing with Peppermint

## Overview
Self-hosted ticketing system deployed via Docker on Ubuntu 22.04 LTS for simulating helpdesk workflows.

## Objectives
- Deploy Peppermint ticketing system inside a Docker container on Ubuntu
- Understand ticket assignment, prioritisation, and management workflows
- Document complete ticket lifecycle from creation to closure
- Learn to triage tickets based on impact and urgency
- Practise escalating tickets beyond scope to appropriate technicians

## Technologies Used
- VMware Workstation Pro
- Ubuntu 22.04 LTS
- Docker Engine
- Docker Compose
- Peppermint (open-source ticketing platform)

>**Lab Environment:** See my <a href="https://github.com/Luka-Babetzki/IT-home-lab">IT homelab setup documentation<a/> for virtualisation platform and base configuration details.

## Architecture

- Host: Ubuntu 22.04 LTS VM (`192.168.138.12`)
- Container orchestration: Docker Compose
- Database: PostgreSQL (containerised)
- Application: Peppermint web interface (ports 3000, 5003)
- Network: Bridge network between containers

## Implementation Steps

### 1. Install Docker on Ubuntu

Followed official Docker documentation to install Docker Engine and Docker Compose plugin.

**Add Docker's official GPG key and repository:**
```bash
# Update package index and install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

**Install Docker packages:**
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Verify installation:**
```bash
sudo docker run hello-world
```

**Challenge encountered:** Initial installation failed due to conflicting Docker packages from Ubuntu's default repositories. Resolved by removing all unofficial Docker packages before installing from official Docker repository (see Troubleshooting section).

### 2. Deploy Peppermint via Docker Compose

Created Docker Compose configuration to orchestrate Peppermint application and PostgreSQL database containers.

**Create project directory and compose file:**
```bash
mkdir ~/peppermint
cd ~/peppermint
nano docker-compose.yml
```

**Docker Compose configuration:**
```yaml
version: "3.1"

services:
  peppermint_postgres:
    container_name: peppermint_postgres
    image: postgres:latest
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: peppermint
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: peppermint

  peppermint:
    container_name: peppermint
    image: pepperlabs/peppermint:latest
    ports:
      - 3000:3000
      - 5003:5003
    restart: always
    depends_on:
      - peppermint_postgres
    environment:
      DB_USERNAME: "peppermint"
      DB_PASSWORD: "1234"
      DB_HOST: "peppermint_postgres"
      SECRET: 'peppermint4life'

volumes:
  pgdata:
```

**Verify configuration:**
```bash
cat docker-compose.yml
```

**Start containers in detached mode:**
```bash
sudo docker-compose up -d
```

**Verify containers are running:**
```bash
sudo docker ps
```

Both `peppermint` and `peppermint_postgres` containers showed status "Up".

### 3. Access Peppermint Web Interface

Accessed Peppermint via web browser on host machine.

**URL:** `http://192.168.138.12:3000`

**Default credentials:**
- Username: `admin@admin.com`
- Password: `1234`

Successfully logged in and presented with Peppermint dashboard.

**Security note:** Changed default admin password immediately after first login via Settings > Users.

### 4. Configure Users and Clients

<!--

Set up multiple user accounts with different permission levels to simulate multi-tier support structure.

**Created users:**
- **Tier 1 Technician** (`tier1@lab.local`) - Basic permissions, can view and update assigned tickets
- **Tier 2 Technician** (`tier2@lab.local`) - Elevated permissions, can reassign and escalate tickets
- **Tier 3 Engineer** (`tier3@lab.local`) - Full permissions, handles complex technical issues
- **Manager** (`manager@lab.local`) - Administrative access, oversees all tickets and generates reports

**Configuration steps:**
1. Navigate to Settings > Users
2. Click "Add User"
3. Fill in email, name, and role
4. Set initial password
5. Assign appropriate permissions based on tier level

**Created clients:**
- **Finance Department** - Simulated internal department
- **HR Department** - Simulated internal department
- **External Client Co.** - Simulated external customer

**Configuration steps:**
1. Navigate to Settings > Clients
2. Click "Add Client"
3. Enter client name and contact details
4. Save configuration

### 5. Simulated Ticket Walkthroughs

Created and worked through five realistic support tickets across different tiers to practise complete ticket lifecycle management.

#### Ticket #1: Password Reset Request (Tier 1)

**Scenario:** User from Finance Department cannot log into workstation after weekend.

**Ticket details:**
- Priority: Medium
- Client: Finance Department
- Assigned to: Tier 1 Technician
- Issue: "Cannot log into computer, says password is incorrect"

**Resolution steps:**
1. Verified user identity via email confirmation
2. Reset Active Directory password (simulated via documentation)
3. Confirmed user successfully logged in
4. Documented resolution: "Password reset performed, user authenticated and confirmed access restored"
5. Closed ticket

**Time to resolution:** 15 minutes  
**Escalation required:** No

#### Ticket #2: Printer Not Working (Tier 1)

**Scenario:** HR Department printer offline, multiple users affected.

**Ticket details:**
- Priority: High
- Client: HR Department
- Assigned to: Tier 1 Technician
- Issue: "Printer showing offline, cannot print payroll documents"

**Resolution steps:**
1. Verified printer power and network connectivity
2. Checked print queue for stuck jobs
3. Restarted print spooler service (simulated)
4. Tested print from technician workstation
5. Documented resolution: "Print spooler restarted, test page successful, users confirmed printing restored"
6. Closed ticket

**Time to resolution:** 25 minutes  
**Escalation required:** No

#### Ticket #3: Email Not Syncing on Mobile Device (Tier 1 → Tier 2)

**Scenario:** Executive's mobile device stopped syncing email overnight.

**Ticket details:**
- Priority: High (executive user)
- Client: Finance Department
- Initially assigned to: Tier 1 Technician
- Issue: "iPhone not receiving emails since this morning"

**Tier 1 troubleshooting:**
1. Verified basic connectivity (WiFi/cellular working)
2. Confirmed email credentials correct
3. Attempted removing and re-adding account - failed
4. Checked server status - all services operational
5. Determined issue beyond basic troubleshooting scope

**Escalation to Tier 2:**
- Added notes: "Basic troubleshooting completed, account credentials verified, suspect MDM policy conflict"
- Reassigned ticket to Tier 2 Technician

**Tier 2 resolution:**
1. Reviewed Mobile Device Management (MDM) policies
2. Identified recent policy update conflicting with iOS version
3. Temporarily exempted device from policy
4. Email sync restored immediately
5. Documented resolution: "MDM policy conflict identified, device exempted pending policy revision"
6. Closed ticket

**Time to resolution:** 1 hour 10 minutes  
**Escalation required:** Yes (Tier 1 → Tier 2)

#### Ticket #4: Application Crashes on Launch (Tier 2 → Tier 3)

**Scenario:** Multiple users reporting custom internal application crashing on startup.

**Ticket details:**
- Priority: Critical (affects multiple users, business-critical application)
- Client: External Client Co.
- Initially assigned to: Tier 2 Technician
- Issue: "Inventory management system crashes immediately on launch, error code 0xc0000005"

**Tier 2 troubleshooting:**
1. Verified issue affects 15+ users across multiple departments
2. Checked recent software updates - none deployed
3. Reviewed application logs - access violation error
4. Attempted application repair/reinstall - issue persists
5. Determined root cause likely code-level or database corruption

**Escalation to Tier 3:**
- Added notes: "Widespread issue, access violation error, repair/reinstall ineffective, suspect database or recent backend change"
- Reassigned ticket to Tier 3 Engineer

**Tier 3 resolution:**
1. Connected to application database server
2. Identified corrupted index on primary inventory table
3. Rebuilt database index
4. Verified application functionality across multiple test accounts
5. Coordinated with users to confirm resolution
6. Documented resolution: "Database index corruption identified and repaired, application functionality restored"
7. Closed ticket

**Time to resolution:** 3 hours 45 minutes  
**Escalation required:** Yes (Tier 2 → Tier 3)

#### Ticket #5: Network Drive Inaccessible (Tier 2)

**Scenario:** Entire department cannot access shared network drive.

**Ticket details:**
- Priority: Critical (business operations halted)
- Client: HR Department
- Assigned to: Tier 2 Technician
- Issue: "H: drive not accessible, all HR staff affected, cannot process employee records"

**Resolution steps:**
1. Verified issue scope - entire HR department (20 users)
2. Checked file server status - online and responsive
3. Verified network connectivity from affected segment
4. Identified security group membership change removed HR department access
5. Restored security group permissions on shared folder
6. Verified access restored for all affected users
7. Documented resolution: "Security group permissions inadvertently modified during routine audit, permissions restored, access confirmed"
8. Closed ticket

**Time to resolution:** 45 minutes  
**Escalation required:** No

-->

## Key Learnings

-
-
-
-
-

## Future Enhancements

- [ ] Add screenshots of Peppermint dashboard and ticket workflows
- [ ] Integrate email notifications for ticket updates
- [ ] Configure SLA (Service Level Agreement) timers for priority-based response times
- [ ] Create knowledge base articles from resolved tickets
- [ ] Set up ticket templates for common issue types

## Troubleshooting

**Docker installation fails due to conflicting packages:**

Remove all unofficial Docker packages from Ubuntu repositories before installing from official Docker repository:
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Then follow installation steps from Step 1.

**Peppermint container won't start or crashes:**

Check container logs for errors:
```bash
sudo docker logs peppermint
```

Restart containers:
```bash
sudo docker-compose down
sudo docker-compose up -d
```

**Cannot access Peppermint web interface:**

Verify containers are running:
```bash
sudo docker ps
```

Check Ubuntu firewall isn't blocking port 3000:
```bash
sudo ufw status
sudo ufw allow 3000/tcp
```

Verify you're accessing correct IP address (`192.168.138.12:3000` in this lab).

## Resources

- [Peppermint Official Documentation](https://docs.peppermint.sh/)
- [Docker Installation Guide for Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Peppermint GitHub Repository](https://github.com/Peppermint-Lab/peppermint)

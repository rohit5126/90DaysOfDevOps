# Runners: GitHub-Hosted & Self-Hosted

## Task 1: GitHub-Hosted Runners

```

name: ghhosted
on: 
  workflow_dispatch:

jobs:
  job1:
    strategy:
      matrix:
        os: [ ubuntu-latest, ubuntu-slim, ubuntu-24.04 ]
    runs-on: ${{ matrix.os }}
    steps:
      - name: OS name
        run: cat /etc/os-release | grep "PRETTY_NAME"
      - name: hostname
        run: hostname
      - name: current user
        run: whoami

```

#### github hosted runners are provided by github and they are managed by github itself. they are ephemeral. they are destroyed just after completion of job.
--------------------------------------------------------

## Task 2: Explore What's Pre-installed

```
name: ghhosted
on: 
  workflow_dispatch:

jobs:
  job2:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
    steps:
      - name: docker version
        run: docker version
      - name: git
        run: git version
      - name: python
        run: python --version
      - name: node
        run: node -v
```
#### all the below are pre installed.

**Docker version**
**Python version**
**Node version**
**Git version**

#### Runners (like GitHub-hosted runners) come with tools pre-installed to eliminate environment setup time
---------------------------------------------------------

## task 3: Set Up a Self-Hosted Runner

steps to configure runner
* go to repo settings
* got to action/ runners
* get new self hosted runner
* select linux
* follow the steps mentioned below
  
```
name: sfhosted
on:
  workflow_dispatch:
jobs:
  job1:
    runs-on: self-hosted
    strategy:
      fail-fast: false
    steps:
      - name: docker
        run: docker version
      - name: whoami
        run: whoami
      - name: hostname
        run: hostname
```

## Task 5: Labels

```
name: sfhosted
on:
  workflow_dispatch:
jobs:
  job1:
    runs-on: [self-hosted, linux-practice]
    strategy:
      fail-fast: false
    steps:
      - name: docker
        run: docker version
      - name: whoami
        run: whoami
      - name: hostname
        run: hostname
```
`Labels enable precise job routing to specific self-hosted runners. By assigning custom tags (e.g., gpu, macos, or staging), you guarantee workflows execute on machines matching their exact hardware or software dependencies, preventing misassignment issues and optimizing resource usage`

---------------------------------------

### Task 6: GitHub-Hosted vs Self-Hosted


| | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Who manages it? | fully managed github | Managed and maintained by your team/organization. |
| Cost | Free for public repos; uses included minutes/pay-as-you-go per minute for private repos. | Free to register with GitHub, but you pay the underlying cloud or server infrastructure costs. |
| Pre-installed tools | Extensive bundle (e.g., Node.js, Python, Docker, Git, etc.) updated automatically by GitHub.| Blank slate; you manually configure the OS, dependencies, and tools you need.|
| Good for | Standard workflows, open-source projects, and teams wanting a simple "set it and forget it" setup. | Custom hardware, accessing on-premises private networks, and large workloads where you want deep control. |
| Security concern | Very low; runs in a secure, ephemeral, isolated VM that is destroyed after each job. | Higher risk; untrusted workflows could potentially compromise your local network, leak secrets, or persist unwanted data if the machine isn't isolated. |

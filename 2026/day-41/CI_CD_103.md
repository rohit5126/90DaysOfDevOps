# Triggers & Matrix Builds

## Task 1: Trigger on Pull Request

```
name: Trigger
on:
  pull_request:
    branches: master
jobs:
  first:
    runs-on: ubuntu-latest
    steps:
      - name: PR
        run: echo "PR check running for branch ${{ github.ref_name }}"

```

### Verify: Does it show up on the PR page?

<img width="1242" height="425" alt="image" src="https://github.com/user-attachments/assets/2eb5a3cd-e6fb-4a3f-b44e-a81b7c718e9a" />

--------------------------------------------

## Task 2: Scheduled Trigger

```
name: trigger2
on:
  schedule:
    - cron: '0 9 * * 1'
  workflow_dispatch: 
jobs:
  second:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4
        
      - name: t2
        run: echo "scheduled job for the pull request"
```

## Task 4: Matrix Builds

```

name: matrix
on: 
  workflow_dispatch:
jobs:
  first:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python: ['3.10', '3.11', '3.12']
    steps:
      - name: installation
        uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python }}

```
### Then extend the matrix to also include 2 operating systems — how many total jobs run now?
```
name: matrix
on: 
  workflow_dispatch:
jobs:
  first:
    strategy:
      matrix:
        os: [ ubuntu-latest , ubuntu-slim ]
        python: ['3.10', '3.11', '3.12']
    runs-on: ${{ matrix.os }}
    steps:
      - name: installation
        uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python }}
```
------------------------------------------

## Task 5: Exclude & Fail-Fast

**this will exclude one job out of six**

```
name: matrix
on: 
  workflow_dispatch:
jobs:
  first:
    strategy:
      matrix:
        exclude:
          - os: ubuntu-slim
            python: 3.10
        os: [ ubuntu-latest , ubuntu-slim ]
        python: ['3.10', '3.11', '3.12']
    runs-on: ${{ matrix.os }}
    steps:
      - name: installation
        uses: actions/setup-python@v6
        with:
       python-version: ${{ matrix.python }}
```

### fail-fast

#### with true

```
name: matrix
on: 
  workflow_dispatch:
jobs:
  first:
    strategy:
      fail-fast: true
      matrix:
        exclude:
          - os: ubuntu-slim
            python: 3,10
        os: [ ubuntu-latest , ubuntu-slim ]
        python: ['3,10', '3.11', '3.12']
    runs-on: ${{ matrix.os }}
    steps:
      - name: installation
        uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python }}
```
<img width="1612" height="670" alt="image" src="https://github.com/user-attachments/assets/08da1383-0771-4310-a51b-76a5c4c8c9ca" />

-------------------------------------------

#### with false

```
name: matrix
on: 
  workflow_dispatch:
jobs:
  first:
    strategy:
      fail-fast: false
      matrix:
        exclude:
          - os: ubuntu-slim
            python: 3,10
        os: [ ubuntu-latest , ubuntu-slim ]
        python: ['3,10', '3.11', '3.12']
    runs-on: ${{ matrix.os }}
    steps:
      - name: installation
        uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python }}

```

<img width="1405" height="572" alt="image" src="https://github.com/user-attachments/assets/e9c4952b-0f92-4e91-bc4c-6a26abb8c9ed" />

**with false if one job is failed rest jobs will ctinue as usual.**

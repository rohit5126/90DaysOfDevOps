# Secrets, Artifacts & Running Real Tests in CI

### Secrets securely inject sensitive data (like API keys) into your workflow,
### while Artifacts preserve and share files (like test reports or build binaries) generated during execution

### Task 1: GitHub Secrets
```
name: security
on: 
  workflow_dispatch:
jobs:
  job1:
    runs-on: ubuntu-latest
    env:
      secret: ${{ secrets.SuperSecret }}
    steps:
      - name: print
        if: ${{ env.secret == '' }} 
        run: "secret is empty"
      - name: status]
        env:
          name: 'rohit'
        run: |
          echo " ${{ secrets.SuperSecret }}" 
          echo env.name
```

**Secrets cannot be directly referenced in if: conditionals. Instead, consider setting secrets as job-level environment variables, 
then referencing the environment variables to conditionally run steps in the job. For more information, see Contexts reference**
---------------------------------------------

### Task 2: Use Secrets as Environment Variables


### Upload Artifacts and Download Artifacts Between Jobs
```
name: artifacts
on: 
  workflow_dispatch: 

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - name: log file
        run: |
          echo "this is log file" >> logfile.txt
          ls
      - name: upload
        uses: actions/upload-artifact@v4
        with:
          name: log-file
          path: logfile.txt

  job2:
    needs: job1
    runs-on: ubuntu-latest
    steps:
      - name: download
        uses: actions/download-artifact@v4
        with: 
          name: log-file
          path: app/
      - name: print
        run: |
          cd app
          touch logfile2.txt
          ls
      
        
```

Can you see and download it from GitHub? yes we can download it from inside workflow run.

--------------------------------------------------

### Task 5: Run Real Tests in CI

```
name: main
on:
  workflow_dispatch:

jobs:
  code-build:
    runs-on: ubuntu-latest
    steps:
      - name: code-chekcout
        uses: actions/checkout@v7
      - name: install
        uses: actions/setup-node@v4
        with:
          node-version: 20
          
      - name: build
        run: |
          cd frontend
          npm install
          npm run build
      - name: upload
        uses: actions/upload-artifact@v4
        with:
          name: main_file
          path: frontend/dist/
```

### Task 6: Caching


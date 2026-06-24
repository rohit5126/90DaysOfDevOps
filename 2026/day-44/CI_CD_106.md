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

<img width="1170" height="702" alt="image" src="https://github.com/user-attachments/assets/dde01412-59a4-4a56-bf7e-24a253ddface" />

<img width="1245" height="715" alt="image" src="https://github.com/user-attachments/assets/d30b6f57-e7f9-49a0-9a8e-2135a4057b8b" />

----------------------------------------------

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

<img width="1652" height="857" alt="image" src="https://github.com/user-attachments/assets/088c5211-a652-4fd4-b99a-df1f2c562374" />


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
------------------------------------------------------------------

### Task 6: Caching

**GitHub Actions caching allows you to save and reuse files like build outputs and package dependencies across workflow runs to drastically decrease build times**

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
      - id: cache-d     #always use id
        uses: actions/cache@v6
        with:
          path: ~/.npm   #this path is for the dir you want to cache
          key: ${{ runner.os }}-node-${{ hashFiles('**/frontend/package-lock.json') }}  #its purpose is only to uniquely name the cache file. hashfiles provide a haskey
          restore-keys: |         #this step restore already existing cache 
            ${{ runner.os }}-node-
      - name: install
        if: steps.cache-d.outputs.cache-hit != 'true'    # this steps make sure to skip the install if cache is restored. if not it will install
        uses: actions/setup-node@v4
        with:
          node-version: 20   
      - id: cache
        uses: actions/cache@v6
        with: 
          path: frontend/dist/
          key: ${{ runner.os }}-node-build-${{ hashFiles('**/frontend/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-build-
      - name: build
        if: steps.cache.outputs.cache-hit != 'true'   # this steps make sure to skip the build if cache is restored. if not it will build
        run: |
          cd frontend
          npm install
          npm run build
      - name: print
        run: |
          cd frontend/dist/
          ls -l
          
        
```

**Write in your notes: What is being cached and where is it stored?**

the directory is being cached and it is stores in github repo. you can look for it under (Go to your repository > Actions > Caches ).

<img width="1232" height="776" alt="image" src="https://github.com/user-attachments/assets/81f86959-060b-418e-a243-00bc4413a91b" />



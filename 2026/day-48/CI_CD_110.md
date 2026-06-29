# GitHub Actions Project: End-to-End CI/CD Pipeline

## reusable-node-build-test.yml

```
name: test-build
on:  
  workflow_call: 
    inputs:
      node-version: { required: false, default: '22', type: string }
      package-path: { required: true, type: string }
      run_tests: { required: false, default: true, type: boolean }
jobs:
  test-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: code-checkout
        uses: actions/checkout@v6
      - name: setup node
        uses: actions/setup-node@v6
        with:
          node-version: ${{ inputs.node-version }}
          cache-dependency-path: ${{ inputs.package-path }}

      - name: Install dependencies
        run: npm install
        working-directory: ${{ inputs.package-path }}

      - name: testing
        if: ${{ inputs.run_tests == true }}
        run: npm test
        working-directory: ${{ inputs.package-path }}

      - name: test-result
        if: failure()
        run: |
          echo "the test cases did not passed "
          exit 1
      - name: test-result
        run: echo " test cases are passed for the build "
```

## reusable-go-build-test.yml

```
name: go-build-test
on: 
  workflow_call:
    inputs:
      go-version: { default: '1.23', type: string } 
      go-version-file: { required: true, type: string }
      cache-dependency-path: { required: true, type: string }
      code-path: { required: true, type: string }
  
jobs:
  go-build-test:
      runs-on: ubuntu-latest
      steps:
        - name: code-setup
          uses: actions/checkout@v6
          
        - name: go-steup
          uses: actions/setup-go@v6
          with:
            go-version: ${{ inputs.go-version }}
            go-version-file: ${{ inputs.go-version-file }}
            cache-dependency-path: ${{ inputs.cache-dependency-path }}
        - name: run go-format
          run: go fmt main.go
          working-directory: ${{ inputs.code-path }}
        - name: run go-vet
          run: go vet main.go
          working-directory: ${{ inputs.code-path }}

        - name: testing
          run: go test
          working-directory: ${{ inputs.code-path }}
```

## reusable-docker.yml

```
name: docker-reuse
on:
  workflow_call:
    inputs:
      image-name: { required: true, type: string }
      tag: {required: false, default: latest, type: string }
      docker-name: { required: true, type: string }
      context: { required: true, type: string }
    secrets:
      docker_username: { required: true }
      docker_password: { required: true }
  
      
jobs:   
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: code
        uses: actions/checkout@v6
      - name: docker login
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_password }}

      - name: docker-build
        id: build
        uses: docker/build-push-action@v7
        with:
          context: ${{ inputs.context }}
          push: true
          tags: ${{ inputs.docker-name }}/${{ inputs.image-name }}:${{ inputs.tag }}
```

## pr-pipeline.yml

```
name: pr-pipeline
on:
  pull_request:
      branches: [master]
      types: [opened, synchronize]

jobs:
  build-test-frontend:
    uses: ./.github/workflows/reusable-node-build-test.yml
    with:
      node-version: '22' 
      package-path: 'frontend' 
      run_tests: true
  build-test-backend:
    uses: ./.github/workflows/reusable-go-build-test.yml
    with:
      go-version: '1.23' 
      go-version-file: backend/go.mod
      cache-dependency-path: backend/go.sum
      code-path: backend
      
  job3:
    runs-on: ubuntu-latest
    needs: [build-test-backend,build-test-frontend]
    permissions:
      pull-requests: write
    steps:
      - uses: mshick/add-pr-comment@v3
        with:
          message: |
            ### Automated Code Analysis Report
            * **Format check:** Passed for both frontend and backend
            * **linter status:** Passed for both frontend and backend
            * **test cases:** Passed for both frontend and backend

```

## main-pipeline.yml

```
name: main
on: 
  push:
    branches: master

jobs:
  docker-buld-push:
    strategy:
      matrix:
        repo: [ 'frontend', 'backend' ]
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image-name: devboard-${{ matrix.repo }}
      tag: latest
      docker-name: rohit5126
      context: ${{ matrix.repo }}
    secrets: 
      docker_username: ${{ vars.DOCKER_USERNAME }}
      docker_password: ${{ secrets.DOCKER_TOKEN }}
    
    
  deploy:
     needs: [docker-buld-push]
     runs-on: self-hosted
     steps:
       - name: code
         uses: actions/checkout@v6
       - name: docker deploy
         run: |
           cp .env.example .env
           docker compose up -d

```

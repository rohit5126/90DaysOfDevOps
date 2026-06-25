# Docker Build & Push in GitHub Actions

## Task 1: Build the Docker Image in CI Push to Docker Hub

#### this also includes the testing and linting of the application code. for both frontend and backend. dokcer setup is already done in previous day. 

```
name: devboard
on:
  push:
    branches: master
    paths: .github/workflows/devboard.yml
jobs:
  frontend:
    runs-on: ubuntu-latest
    steps:
      - name: code-checkout
        uses: actions/checkout@v7
      - name: setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      - name: install-npm
        run: npm ci
        working-directory: frontend/
      - name: run-linter
        run: npm run lint
        working-directory: frontend/
      - name: test
        run: npm run test
        working-directory: frontend/
      - name: Docker-login
        uses: docker/login-action@v4
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: docker-build-push
        uses: docker/build-push-action@v7
        with:
          context: frontend/
          push: true
          tags: ${{ vars.DOCKER_USERNAME }}/devboard-frontend:latest

          
  backend:
    runs-on: ubuntu-latest
    steps:
      - name: code-checkout
        uses: actions/checkout@v7
      - name: setup-go
        uses: actions/setup-go@v6
        with:
          go-version: '1.23'
          go-version-file: backend/go.mod
          cache-dependency-path: backend/go.sum
      - name: run go-format
        run: go fmt main.go
        working-directory: backend/
      - name: run go vet
        run: go vet main.go
        working-directory: backend/

      - name: testing
        run: go test
        working-directory: backend/

      - name: docker login
        uses: docker/login-action@v4
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: docker build and push
        uses: docker/build-push-action@v7
        with:
          context: backend/
          push: true
          tags: ${{ vars.DOCKER_USERNAME }}/devboard-backend:latest

```
-------------------------------------------------------
## Task 4: Only Push on Main

```
name: docker-build-push
        if: ${{ github.ref_name == 'master' }}
        uses: docker/build-push-action@v7
        with:
          context: frontend/
          push: true
          tags: ${{ vars.DOCKER_USERNAME }}/devboard-frontend:latest

```
-------------------------------------------

## Task 6: Pull and Run It

**did that and everything si working fine. application is up and running.**

      
        
        
        

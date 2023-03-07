# CICD-Activity

### Apps needed:
- WSL-2 
  - https://pureinfotech.com/install-windows-subsystem-linux-2-windows-10/
- Git
  - https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-git
- Node v18
  - use nvm https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl
- VS Code
  - https://code.visualstudio.com/download
  
### 1. Gitlab setup
1. Create account https://gitlab.com/users/sign_in
2. Create blank project
3. Add ssh-key
  - https://gitlab.com/-/profile/keys
  - to generate: `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
  - to print the public key: `cat ~/.ssh/id_rsa.pub`

### 2. Create CRA
- https://reactjs.org/docs/create-a-new-react-app.html
- npx create-react-app my-cicd-app
- cd my-cicd-app
- npm start

### 3. Initialize git and push
- cd my-cicd-app
- git init
- git remote add origin git@gitlab.com:<username>/<project_name>.git
- git add .
- git commit -m "Initial commit"
- git push -u origin master

### 4. Create .gitlab-ci.yml for pipeline config
- https://docs.gitlab.com/ee/ci/quick_start/index.html
```
build-job:
  script:
    - echo "Hello"
```
- git add .
- git commit -m "Add pipeline config"
- git push

### 5. Check pipeline
- https://gitlab.com/<username>/<project_name>/-/pipelines
- if the pipeline failed due to *User validation required*,
  - Project Settings -> CI/CD -> Runners -> Click Expand
  - https://gitlab.com/<username>/<project_name>/-/settings/ci_cd
  - Disable the option *Enable shared runners for this project*
  - Copy registration token and send to the instructor with your family name (e.g. GR1348941xduQ8c1Bk1zuCDE7QkMp - gasataya)
  - Wait for the runners to be available
  - Rerun the pipeline
  
### 6. Update .gitlab-ci.yml
- add stages and additional jobs
```
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "build!!"

test-job1:
  stage: test
  script:
    - echo "test 1!!"

test-job2:
  stage: test
  script:
    - echo "test 2!!"

deploy-job:
  stage: deploy
  script:
    - echo "deploy!!"
```

### 7. Configure build and test jobs
- build
```
build-job:
  stage: build
  image: node:18
  script:
    - npm install
    - npm run build

test-job1:
  stage: test
  image: node:18
  script:
    - npm install
    - npm run test
```

### 8. Create netlify account and install netlify-cli
- https://app.netlify.com/signup
- netlify-cli
  - `npm install netlify-cli -g`
- login: `netlify login`
- create site: `netlify sites:create --disable-linking`
  - Site name: `<lastname-cicd>` (e.g. `gasataya-cicd`)
- copy and save details
```
Site Created

Admin URL: https://app.netlify.com/sites/cicd-app
URL:       https://cicd-app.netlify.app
Site ID:   12345
```

### 9. Create Netlify auth token
- https://app.netlify.com/user/applications
  - Personal access tokens -> New Access Token

### 10. Add Gitlab CICD Variables
- Project Settings -> CI/CD -> Variables -> Click Expand
- `NETLIFY_SITE_ID` 
- `NETLIFY_AUTH_TOKEN`
- Uncheck Protect Variable

### 11. Update .gitlab-ci.yml 
- update deploy job
```
deploy-job:
  stage: deploy
  script:
    - npm install
    - npm install -g netlify-cli --unsafe-perm=true
    - npm run build
    - netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --dir=./build --prod
```
- this will produce an error, so we need to add `image: node:18`


```
deploy-job:
  stage: deploy
  image: node:18
  script:
    - npm install
    - npm install -g netlify-cli --unsafe-perm=true
    - npm run build
    - netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --dir=./build --prod
```

### Others: Update files and push

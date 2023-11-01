# Continues deployment of code using gitlab.

- In this project we will push our node application to server not manually but using gitlab automation process.

## Diagram

![DevOpsProject 2](https://github.com/harsh-viradia/DevOps-Project-2/assets/140060556/f8defb92-db3b-4264-86be-bbce1ddbc6c5)

## Steps.

    1. Host a server and install nodejs and pm2.
    2. On gitlab add code and set variables and runner.
    3. Clone the repo on server.
    4. Perform initial build commands on server.
    5. Create .gitlab.yml file for continues deployment.
    6. Run pipeline.


### Setp-1

    - Install Nodejs by this github document.
    - https://github.com/nodesource/distributions
    - intsall pm2: npm install pm2@latest -g

### Step-2 

    - Add variables on gitlab.
    - prod_spk: to store public key of server to SSH server.
    - prod_usr: Ubuntu username.
    - prod_srv: Ubuntu public IP.
    - prod_webroot: Path to your project on local.

### Step-3

    - Clone git repo by git clone command.

### Step-4

    - Perform build commands which are mentioned on package.json file.
    - In my case I used yarn.
    - yarn run init(npm install && npm run build && pm2 start --name "As per need")
    - yarn serve(npm install && npm run build && pm2 reload --name "As per need")

### Step-5

    stages:
      - deploy

    deploy_prod:
    stage: deploy
    before_script:
        - 'command -v ssh-agent >/dev/null || ( apk add --update openssh )' 
        - eval $(ssh-agent -s)
        - echo "$prod_spk" | tr -d '\r' | ssh-add -
        - mkdir -p ~/.ssh
        - chmod 700 ~/.ssh
        - ssh-keyscan -H $prod_srv >> ~/.ssh/known_hosts
        - chmod 644 ~/.ssh/known_hosts
    script:
        - echo "Deploy to Ubuntu server"
        - ssh $prod_usr@$prod_srv "cd $prod_webroot/ && git fetch && git checkout git-branch-name && git pull origin git-branch-name"
        - ssh $prod_usr@$prod_srv "cd $prod_webroot/ && pm2 reload name-of-pm2-instance"
    only:
        - git-branch-name
    when: manual

### Step-6

Finally run the pipeline .

Thank you!
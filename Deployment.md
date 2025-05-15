# Deploy with OpenShift:

## 1️⃣ Create ``Deploy Token`` in Gitlab:
In you repo go to ``Settings →  Repository → DeployToken``:
- name it (ex. ci-token) and enable reads Scopes
- when you click on create, it will create a **Username** = ``gitlab+deploy-token-{n}`` and **Password** = ``glxt-xxxxxxx-YxZ1``, the username is the ``TokenName`` and the pass is the ``Token value``

⚠️ copy the token because you will not be able to see them again in gitlab

## 2️⃣ Create CI/CD Variables:
Go to ``Settings → CI/CD → Variables``, you should add 3 variables:
- GIT_USERNAME: the **TokenName** of deployToken you have created
- GIT_PASSWORD: the **TokenValue** of deployToken you have created
- OS_TOKEN: 
    - oc login --server=https ........... (from OpenShift)
    - oc get serviceaccount
    - oc describe sa/cicd
    - oc describe secret cicd-token-ftkqc

🟢 you don't have to change anything, only add the Key and Value

## 3️⃣ The files you should add in your project:
```
├── 📄 gitlab-ci.yml

📁 openshift/
├── 📄 openshift.yml
├── 📄 os-post-apply.sh
└── 📄 os-readiness-check.sh
```

- ``gitlab-ci.yml``: you will include a template componant from openShift (), and add the inputs like ``script-dir`` and ``url``, etc...

- ``openshift.yml``: this is the full openshift application definition:
    -  
# eks_wordpress
The actual website development and pipeline
The actual Wordpress in development that will have some sort of development on it.

Each time a pipeline is triggered:
    Github spinsup a preconfigured VM that pulls the repo inside
    /home/runner/work/REPO_NAME/REPO_NAME/
                  └─ Your repo name
                             └─ Actual code here
    Executes the workflows from the .github/workflows/CI.yaml or whatever name
    Then pushes back to that repo or to another repo

PIPELINE WORKFLOW

0. First we have to setup the secrets for Github (the VM ) to access the Github repos and pull/execute CI.yaml commands / push back
    GitHub → Settings → Developer settings → Personal access tokens (classic)
        Generate new token
        Select: repo (all permissions)
        Copy: ghp_blablatext

    eks_wordpress repo → Settings → Secrets and variables → Actions
        New repository secret
        Name: GITOPS_PAT
        Value: ghp_xxxxxxxxxxxx

    - uses: actions/checkout@v4
      with:
        repository: cezarbajenaru/eks_wordpress_gitops
        token: ${{ secrets.GITOPS_PAT }}  # The token generated in github Settings/developer settings/Personal access tokens PAT, and put into the repo settings/secrets/actions
    If you have two repos, create separate secrets for the two repos and include a cluename for the token name. So you find it easier
        
1. Developer pushes changes to repo ( according to bellow rules regarding the types of changes!!! Not everything goes in the same place)
2. Git receives the push and finds the .github/workflows and starts executing the pipeline
3. Github Spins up a VM with ubuntu and starts running the worflows/build-push.yaml in our case
4. VM checks code and uses the credentials (docker user and token) that have been put as secrets in github (separate entries in Github - username and token / token created in Dockerhub account)
5. Dockerfile from repo pulls the official image of wordpress and executes further Dockerfile commands like COPY or SH commands
6. VM pushes image to Dockerhub after it build it to spec ( ArgoCD did not detect this image yet!)
7. VM updates application.yaml with the sed command that replaces the ( GITHUB MUST HAVE ACCESS PRIVELEDGES TO ACCES THE ARGO REPO)
    Remember, we are now in app build pipeline repo, not in Argo repo. The two repos must communicate with access tokens ?? 

    Here is where you left off !

8. VM commits and pushes application.yaml to GIT - ??? something is happening here  Must document



6. After executing the commands, run Docker build and pushes the image to the private Dockerhub account where the updated app resides
6.5 . In the moment that Github received the initial push, Argo has been triggered also to pull that Dockerhub updated image.
Must there be a delay? Argo waits for the Docker image to finish as an updated artifact?
7. ArgoCD then takes the image, uses the Helm charts to create infrastructure + Argo specific Yaml configuration to automatically deploy or sync/refresh the applications.
8. Argo secrets section still to be written and executed with SOPS


If a push is going to anything else than ["main"] = prouction then other environments are linked to secondary branches. Tree bellow:

feature/add-payment-bug  →  Dev Environment      (AWS Account: Dev)
                       ↓
                    Staging Environment      (AWS Account: Staging)
                       ↓
main branch        →  Production Environment (AWS Account: Prod)


Possible deplyment scenarios ( havong multiple environments available - dev, staging, production):

1. Feature branches:
    App gets deployed only when pushed to main.
    If git checkout -b featuredev/branchname then no deployment in CI, only in dev ( or choose any other env except production which represents main)
    Code review exists before deployment

2. Manual deployment
    Just to go github-> actions -> run workflow -> select env -> trigger deployment
    Full control over deploy, you can forget to deploy or deploy broken code 

3. Tag based deployment - Deployments have tags and everything else does not.
    Regular commits do not do anything
        git commit -m "bbla bla" 
        git push
        Does not deploy anything

        git tag v1.25.0
        git push origin v1.25.0 
        No that we assign a tag version, the push will trigger the actions Pipeline

4. Path Based triggers ( the smartest yet ) - HIGH maintenance after creation!!!
    git commit -m "theme color changes blabla"  # these changes are pushed to a path that in actions is declared to trigger the CI
    the path can be wp-content/themes/style.css  ( the app code changed)
    git push

This kind of workflow can get complex and we constantly have to add paths that get created into the repo. Must have a tracking mechanism for listing new paths created by the devs compared to last commits ( check diffs ?)

We can use broader paths that when commited to, trigger the deployment 
    paths:
      - 'Dockerfile'
      - 'src/**'           # Everything under src
      - 'config/**'
      - '!docs/**'         # Exclude docs (note: limited support)

Or we can trigger on every path except these 
    paths-ignore:
      - 'docs/**'
      - 'argocd/**'
      - '*.md'
      - '.github/**'

Nx, Turborepo, or Bazel can detect affected projects and trigger only relevant builds programmatically.


The three ways WP handles data - so I know if I modify something, will it end up in the right place or not. 
```
wordpress_storage_model:
  content:
    examples:
      - pages
      - posts
      - menus
      - widgets
      - site_title
      - theme_customizer_settings
    stored_in: "mariadb_database"
    persisted_via_pv: true
    versioned_as_code: false

  media_files:
    examples:
      - images
      - videos
      - pdfs
      - uploads
    stored_in: "wp-content/uploads (PVC)"
    persisted_via_pv: true
    versioned_as_code: false

  code_and_design:
    examples:
      - theme_php
      - theme_css
      - plugin_php
    stored_in: "theme/plugin folders within container"
    persisted_via_pv: "only if PV mounted to theme/plugin paths"
    versioned_as_code: "should be"


```

How data is stored

```
wordpress_storage_model = {
    "content": {
        "examples": ["pages", "posts", "menus", "widgets", "site_title", "customizer_settings"],
        "stored_in": "mariadb_database",
        "persisted_via_pv": True,
        "versioned_as_code": False
    },
    "media_files": {
        "examples": ["images", "videos", "pdfs", "uploads"],
        "stored_in": "wp-content/uploads (PVC)",
        "persisted_via_pv": True,
        "versioned_as_code": False
    },
    "code_and_design": {
        "examples": ["theme_php", "theme_css", "plugin_php"],
        "stored_in": "theme/plugin folders",
        "persisted_via_pv": "only if PV-mounted",
        "versioned_as_code": "should be"
    }
}

```

Credentials are added into Github Actions. Token can be generated into Dockerhub PAT ( personal access token )
Then go to github repo, settings
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   deploy-app-in-k8-cluster/mongo-configmap.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	deploy-app-in-k8-cluster/dashboard-ingress.yaml
	mosquito/

no changes added to commit (use "git add" and/or "git commit -a")
gabidinica@Mac /Users/gabidinica/k8 [main]
% git checkout -b mosquitto-deployment
Switched to a new branch 'mosquitto-deployment'
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   deploy-app-in-k8-cluster/mongo-configmap.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	deploy-app-in-k8-cluster/dashboard-ingress.yaml
	mosquito/

no changes added to commit (use "git add" and/or "git commit -a")
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% clear          

gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   deploy-app-in-k8-cluster/mongo-configmap.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.DS_Store
	deploy-app-in-k8-cluster/dashboard-ingress.yaml
	mosquitto/

no changes added to commit (use "git add" and/or "git commit -a")
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% vim .gitignore
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   deploy-app-in-k8-cluster/mongo-configmap.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore
	deploy-app-in-k8-cluster/dashboard-ingress.yaml
	mosquitto/

no changes added to commit (use "git add" and/or "git commit -a")
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% vim readme.md
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% ls
deploy-app-in-k8-cluster	mosquitto			readme.md
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% vim readme.md
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   deploy-app-in-k8-cluster/mongo-configmap.yaml
	modified:   readme.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore
	mosquitto/

no changes added to commit (use "git add" and/or "git commit -a")
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git add .
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git commit -m 'Deploy Mosquitto message broker with ConfigMap and
Secret Volume Types'
[mosquitto-deployment d497bd5] Deploy Mosquitto message broker with ConfigMap and Secret Volume Types
 8 files changed, 207 insertions(+), 3 deletions(-)
 create mode 100644 .gitignore
 create mode 100644 mosquitto/config-file.yaml
 create mode 100644 mosquitto/example-simple-certificate.yaml
 create mode 100644 mosquitto/mosquitto-without-volumes.yaml
 create mode 100644 mosquitto/mosquitto.yaml
 create mode 100644 mosquitto/secret-file.yaml
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git push
fatal: The current branch mosquitto-deployment has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin mosquitto-deployment

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.

gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git push --set-upstream origin mosquitto-deployment
Enumerating objects: 16, done.
Counting objects: 100% (16/16), done.
Delta compression using up to 10 threads
Compressing objects: 100% (11/11), done.
Writing objects: 100% (12/12), 2.77 KiB | 2.77 MiB/s, done.
Total 12 (delta 2), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
remote: 
remote: Create a pull request for 'mosquitto-deployment' on GitHub by visiting:
remote:      https://github.com/gabidinica/container-orchestration-with-k8/pull/new/mosquitto-deployment
remote: 
To github.com:gabidinica/container-orchestration-with-k8.git
 * [new branch]      mosquitto-deployment -> mosquitto-deployment
branch 'mosquitto-deployment' set up to track 'origin/mosquitto-deployment'.
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Your branch is up to date with 'origin/mosquitto-deployment'.

nothing to commit, working tree clean
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git restore readme.md
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Your branch is up to date with 'origin/mosquitto-deployment'.

nothing to commit, working tree clean
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git log -- readme.md
commit d497bd5ecb8aa2fb204b86da5dfc5ebb95fc1510 (HEAD -> mosquitto-deployment, origin/mosquitto-deployment)
Author: Gabi <dinica_gabi30@yahoo.com>
Date:   Sun Aug 24 13:24:42 2025 +0300

    Deploy Mosquitto message broker with ConfigMap and
    Secret Volume Types

commit 561f66ac985f16e52d58e1e40906319f6bd4b00a
Author: Gabi <dinica_gabi30@yahoo.com>
Date:   Thu Aug 21 17:19:57 2025 +0300

    Added links to the repos
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% git status
On branch mosquitto-deployment
Your branch is up to date with 'origin/mosquitto-deployment'.

nothing to commit, working tree clean
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% vim readme.md
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% ls -la
total 32
drwxr-xr-x   8 gabidinica  staff   256 Aug 24 13:29 .
drwxr-x---+ 55 gabidinica  staff  1760 Aug 24 13:29 ..
-rw-r--r--@  1 gabidinica  staff  6148 Aug 24 13:10 .DS_Store
drwxr-xr-x  14 gabidinica  staff   448 Aug 24 13:28 .git
-rw-r--r--   1 gabidinica  staff    10 Aug 24 13:08 .gitignore
drwxr-xr-x   7 gabidinica  staff   224 Aug 24 13:10 deploy-app-in-k8-cluster
drwxr-xr-x   7 gabidinica  staff   224 Aug 24 12:51 mosquitto
-rw-r--r--@  1 gabidinica  staff  3495 Aug 24 13:29 readme.md
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% cd mosquitto 
gabidinica@Mac /Users/gabidinica/k8/mosquitto [mosquitto-deployment]
% ls
config-file.yaml		mosquitto-without-volumes.yaml	secret-file.yaml
example-simple-certificate.yaml	mosquitto.yaml
gabidinica@Mac /Users/gabidinica/k8/mosquitto [mosquitto-deployment]
% ls -la
total 40
drwxr-xr-x  7 gabidinica  staff  224 Aug 24 12:51 .
drwxr-xr-x  8 gabidinica  staff  256 Aug 24 13:29 ..
-rw-r--r--@ 1 gabidinica  staff  174 Aug 23 17:24 config-file.yaml
-rw-r--r--@ 1 gabidinica  staff  149 Aug 24 12:51 example-simple-certificate.yaml
-rw-r--r--@ 1 gabidinica  staff  369 Aug 23 17:38 mosquitto-without-volumes.yaml
-rw-r--r--@ 1 gabidinica  staff  789 Aug 23 17:55 mosquitto.yaml
-rw-r--r--@ 1 gabidinica  staff  134 Aug 23 17:27 secret-file.yaml
gabidinica@Mac /Users/gabidinica/k8/mosquitto [mosquitto-deployment]
% cd ..
gabidinica@Mac /Users/gabidinica/k8 [mosquitto-deployment]
% vim readme.md

  - name: mosquitto-secret
    mountPath: /mosquitto/secret
    readOnly: true
```

📝 Notes
- `mountPath` references the path in the container’s filesystem, as we’ve seen earlier in `/mosquitto/config`.
- The `secret` folder does not exist by default, so it will be automatically created at `/mosquitto/secret`.
- `readOnly` attribute ensures the data is only read, not modified. This is especially important for certificates, which only need to be read.

1. Apply changes:
```bash
kubectl apply -f mosquitto.yaml
```

2. Check the pod: `kubectl get pod`
3. Access the Mosquitto Container: `kubectl exec -it pod-name -- /bin/sh`

Once inside the container:
1. List files: `ls`
2. Go to Mosquitto folder: `cd mosquitto`
3. List contents: `ls`
> 👉 You’ll see, for example, the secret folder, as initially it wasn't there.
4. Check the config content to see that it was overwritten:
```bash
cd config
cat mosquitto.conf
```

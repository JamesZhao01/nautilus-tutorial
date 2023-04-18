# Nautilus Information

## Useful Links
- https://portal.nrp-nautilus.io/
- https://ucsd-prp.gitlab.io/ (official nautilus docs)
- Element (tech support) https://ucsd-prp.gitlab.io/userdocs/start/contact/

# Initial Setup

1. Have ~~Martha (mgahl@eng.ucsd.edu)~~ Me(James Zhao, jjz005@ucsd.edu as of April 2023) add you to the Nautilus whitelist 
2. Create a ~/.kube directory
3. Get a kubernetes config from (https://portal.nrp-nautilus.io/)
  - Click Login -> Google -> (login to your ucsd with google) -> Get Config
    - I think there is a UCSD SSO login, but it didn't work for me. This option also seems new and didn't exist when I first started using nautilus
  - add the `config` to ~/.kube
  - If you get an auth error whenever running a ```kubectl``` command, it probably means you need to refresh your config. It probably happens on average like every 2 weeks or something
4. Install [kubecli](https://kubernetes.io/docs/tasks/tools/)
  - I linked a cheat sheet doc that Martha gave me a while back containing common kubernetes commands
  - On my machine, I needed to restart my computer for kubecli to work
  - If at any time you get an error like 
    ```
    Error from server (Forbidden): namespaces is forbidden: User "http://cilogon.org/serverE/users/56166" cannot list resource "namespaces" in API group "" at the cluster scope
    ```
    probably reach out to Martha or support on Element
5. Switch to the ```guru-research``` namespace since other universities also use nautilus
  - `kubectl config set-context --current --namespace=guru-research`
6. You should be able to do `kubectl get pods` to see all running pods of people in GURU (usually Shubham has some Lp-net stuff running, or people in SMART have some hsqc model. All of my experiments are prefixed by a `j` (e.x. `j-pod-lightning`))
  - ```
    NAME                                READY   STATUS                     RESTARTS   AGE
    devpod-fx486                        1/1     Running                    0          20h
    hsqc                                0/1     Error                      0          26h
    ...
    ```

# Creating a Volume
- [kubernetes docs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- Bsaically a remote file system that saves your data since pods/jobs get wiped. Put logs/datasets on these

You can see all persistent volumes with:
- `kubectl get pvc` (pvc for persistent volume claim). Mine are all prefixed with `james`, you probs should to so Martha can manage them easier and know who they belong to

To create a persistent volume, I gave you a yaml file (`persistent-volume.yml`) that you just need to fill out for the name. You can also adjust the storage (prob anything under 500Gi is fine, whatever fits your needs since)
- this config uses `ReadWriteMany`, which lets you mount it to multiple pods/jobs at the same time. Normally if you use `ReadWriteOnce`, you can only mount it to one thing at a time at a time and launching another job/pod will throw an error. I haven't found any downsides to this, but it might be slower or smth?
- I think this command should work, I don't want to test it in case it overrides my current volume 😞
- `kubectl apply -f persistent-volume.yml`
- [docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

Poggers, time to test with
- `kubectl create -f pod-test.yml`
- you can try to `touch` a file in the volume, delete the pod, restart it, and check if the file is still there
- you can also try launching multiple pods (change the name for each) and see if the files are synced (I don't remember if it's treated like a network file system or if it just writes on pod close)

Delete the pod with
- `kubectl delete pod <whatever you named your pod>`

# Secrets
If you are cloning a private repo, you might need an ssh config or a PAT. You can use a kubernetes secret and pass it in as an environment variable, but the secret is stored in plaintext and anybody can see it (I did this before but I'm trying some other ways to avoid having my personal PAT in plaintext). Theoretically only guru people should be able to access but anyone can relaly switch to our namespace... You can try encrypting/descrypting the string using a secret key, using a command to substitute strings in your configs, etc...

# Creating Pods
The above configs can probably get you pretty far. You probably want to create your own docker image and host it on docker hub or something with whatever dependencies you use. Probably base it off a pytorch-cuda image or something
- Pods only last 6 hours and I usually only use them as a dev environment with a gpu

# Development
- I use visual studio code's Kubernetes extension with Remove Development to ssh into pods. They make it very easy to perform, and you get a lot of QOL benefits like auto-port forwarding (e.x. running tensorboard in your pod), all of your extensions (you will have to reinstall the Python extensions though within your pod).
- Install the Kubernetes and Remote Development Extension Pack
  - Click the Kubernetes button (looks like a web in a heptagon) -> And open the dropdowns (nautilus -> Workloads -> Pods -> **your pod**)
  - Right click the pod -> Attach Visual Studio Code, and a new window will pop up ssh'd into your pod!
  - In the new window, Open a folder (Ctrl+Shift+P and search "Open Folder"), choose your working directory, and happy coding!

# Jobs
Jobs are very similar to pods, except you are allowed to allocate more RAM (pods have <12 GB requirement) and can last longer than 6 hours. I've added an example job config, that uses a multi-line yaml command if you want to run many experiments with lots of cli args. It also has some configs for loading secrets as enviornment variables (I have removd my PAT though ＼ʕ •ᴥ•ʔ／ ). 

# Troubleshooting
If your shell command throws an error, it will probably kill the pod. You can view logs by doing 
`kubectl logs <podname>`  

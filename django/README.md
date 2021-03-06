# Django + Celery Sample App

This example shows how to leverage [Okteto](https://github.com/okteto/okteto) to develop a Django + Celery Sample App directly in Kubernetes. The Django + Celery Sample App is deployed using raw Kubernetes manifests.

Okteto works in any Kubernetes cluster (local or remote) by reading your local Kubernetes credentials.

## Step 1: Install the Okteto CLI

Install the Okteto CLI by following our [installation guides](https://github.com/okteto/okteto/blob/master/docs/installation.md).


## Step 2: Django + Celery Sample App

Clone the repository and go to the django folder.

```console
$ git clone https://github.com/okteto/samples
$ cd samples/django
```

The Django + Celery Sample App is a multi-service application. It consists of a web view, a worker, a queue, a cache and a DB. 
Deploy the entire application by using the following command:

```console
$ kubectl apply -f manifests
statefulset.apps "cache" created
service "cache" created
statefulset.apps "db" created
service "db" created
statefulset.apps "queue" created
service "queue" created
deployment.apps "web" created
service "web" created
deployment.apps "worker" created
```

Check that all pods are ready. You can do it by running the command below:
```
$ kubectl get pod                                                                                      
NAME                      READY   STATUS    RESTARTS   AGE
cache-0                   1/1       Running   0          2m
db-0                      1/1       Running   0          2m
queue-0                   1/1       Running   0          2m
web-7bccc4bc99-2nwtc      1/1       Running   0          2m
worker-654d7b8bd5-42rq2   1/1       Running   0          2m
```

## Step 3: Create your Okteto Environment

In order to activate your Cloud Native Development, execute:

```console
$ okteto up
 ✓  Files synchronized
 ✓  Okteto Environment activated
    Namespace: pchico83
    Name:      web
    Forward:   8080 -> 8080
    
curl: (52) Empty reply from server
Database is ready
No changes detected in app 'myproject'
Created migrations
Operations to perform:
  Apply all migrations: auth, contenttypes, myproject, sites
Running migrations:
  No migrations to apply.
Migrated DB to latest version
Performing system checks...

System check identified no issues (0 silenced).
June 26, 2019 - 11:00:02
Django version 1.11.21, using settings 'myproject.settings'
Starting development server at http://0.0.0.0:8080/
Quit the server with CONTROL-C.
```

The `okteto up` command will start a remote development environment that automatically synchronizes and applies your code changes to the web and worker deployments without having to rebuild or redeploy (eliminating the **docker build/push/pull/redeploy** cycle). You can control how your development environment is created by modifying `okteto.yml`.

Since our application is formed of multiple services, we decided to configure `okteto.yml` to create a multi-service remote development environment:

```yaml
name: web
image: okteto/django:latest
command: ["./run_web.sh"]
mountpath: /app
forward:
  - 8080:8080
services:
  - name: worker
    mountpath: /app
```

The `name`,`image`, `command`, `mountpath` and `forward` keys will give Okteto information about your primary service, while the `services` list tells Okteto which other services you want to have as part of your development environment. 

The `mountpath` key tells Okteto [where to mount your local folder](https://okteto.com/docs/reference/manifest/index.html#mountpath-string-optional) in your development environment.  [This page](https://okteto.com/docs/reference/manifest/index.html#services-object-optional) has more information about services, and all the values you can configure.

Once your environment is active, verify that the application is up and running by opening your browser and navigating to http://localhost:8080/jobs/ (Okteto is automatically forwarding port 8080 between your pod and your computer). 

## Step 4: Develop directly in the cloud

Now things get more exciting. Go to your browser, and try to calculate the `fibonacci` number for the number `5`. To do it, put the values shown below in the web UI, leaving everything else as is.

```
Type: fibonacci
Argument: 5
```

Press the `POST` button to submit the operation. The response payload will include the `url` of the job. Go to `http://localhost:8080/jobs/1/` and you will notice that the result is wrong (_hint: the fibonacci number of 5 is not 32_). This is because our worker has a bug 🙀!

Typically, fixing this would involve you running the app locally, fixing the bug, building a new container, pushing it and redeploying your app. Instead, we're going to do it the *Cloud Native way*:

Open `myproject/myproject/models.py` in your favorite IDE, and modify the value of the `task` variable in line 29 to apply the correct operation as shown below, and save your file.

```python
task = TASK_MAPPING['power']
```

to

```python
task = TASK_MAPPING[self.type]
```

Go back to http://localhost:8080/jobs/, reload the page, and submit a new `fibonacci` calculation, using the same values as before. Go to `http://localhost:8080/jobs/2/`, and see if the result is correct this time (_hint: the fibonacci number of 5 is 5_).


How did this happen? Well, with Okteto, your changes were automatically applied as soon as you saved them, no commit, build, push or redeploy required 💪! 

## Step 5: Cleanup

Cancel the `okteto up` command by pressing `ctrl + c` and run the following command to remove the resources created by this guide: 

```console
$ okteto down -v
 ✓  Okteto Environment deactivated

```
 and finally delete the application by running:

```console
$ kubectl delete -f manifests
statefulset.apps "cache" deleted
service "cache" deleted
statefulset.apps "db" deleted
service "db" deleted
statefulset.apps "queue" deleted
service "queue" deleted
deployment.apps "web" deleted
service "web" deleted
deployment.apps "worker" deleted
```

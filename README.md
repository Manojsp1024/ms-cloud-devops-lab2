# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `ravi123/hello-app:v1`):

```
<paste here>

manojsp1024/hello-app:v1

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```
<paste here>
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
hello-app-868c7b7cb-5zjk2   1/1     Running   0          9m12s   10.244.2.9   desktop-worker    <none>           <none>
hello-app-868c7b7cb-64spl   1/1     Running   0          9m12s   10.244.1.7   desktop-worker2   <none>           <none>
hello-app-868c7b7cb-mbdv4   1/1     Running   0          9m18s   10.244.2.8   desktop-worker    <none>           <none>
hello-app-868c7b7cb-ts5nm   1/1     Running   0          9m17s   10.244.1.6   desktop-worker2   <none>           <none>


## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
<paste here>
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.31.222   172.21.0.7    8080:31558/TCP   16m

NAME              ADDRESSTYPE   PORTS   ENDPOINTS                                      AGE
hello-app-c6zjp   IPv4          80      10.244.1.7,10.244.2.9,10.244.2.8 + 1 more...   16m

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**
Initially, the Docker image was available only on my local machine. When I tried deploying hello-app:v1 directly on Kubernetes, the Pods entered the ImagePullBackOff state.

**Trap 2 — the selector typo.**

I intentionally changed the Service selector from app: hello-app to app: helo-app.

The EndpointSlice became empty because no Pods had the incorrect label. The Service could not find any backend Pods, and the application stopped responding. Correcting the selector restored connectivity.

**Trap 3 — the wrong targetPort.**

I changed the Service targetPort from 80 to 8080.

The EndpointSlice still contained the Pod IP addresses, but the destination port was incorrect. Nginx was listening on port 80, so requests to port 8080 failed. Changing the targetPort back to 80 restored the application.

**Trap 4 — the EXTERNAL-IP address.**

The LoadBalancer Service displayed an EXTERNAL-IP address, but accessing that address from my browser did not work.

This happened because the address belonged to Docker Desktop's private network and was not directly reachable from my Mac. Docker Desktop publishes the LoadBalancer port on localhost, so I accessed the application using http://localhost:8080 instead.

In a cloud environment, a public LoadBalancer EXTERNAL-IP can be accessed directly.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?

kubectl create deployment hello-app --image=manojsp1024/hello-app:v1 --replicas=3 --port=80 --dry-run=client -o yaml

kubectl create service loadbalancer hello-app --tcp=8080:80 --dry-run=client -o yaml

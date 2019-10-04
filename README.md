# inlets-operator

Get a Kubernetes LoadBalancer where you never thought it was possible.

inlets-operator provisions an [inlets](https://inlets.dev) VM or bare-metal host with an IaaS provider such as Packet, GCP, AWS or DigitalOcean, then runs the inlets server there. The inlets client runs within your cluster.

## Who is this for?

This solution is for users who want to gain incoming network access (ingress) to their private Kubernetes clusters running on their laptops, VMs, within a Docker container, on-premises, or behind NAT. The cost of the LoadBalancer with a IaaS like DigitalOcean is around 5 USD / mo, which is 10 USD cheaper than an AWS ELB or GCP LoadBalancer.

Whilst 5 USD is cheaper than a "Cloud Load Balancer", this tool is for users who cannot get incoming ingress, not for saving money on public cloud.

## Status

This version of the inlets-operator is a early proof-of-concept, but it builds upon inlets, which is stable and widely used.

Backlog:
- [x] Provision VMs/exit-nodes on public cloud
- [x] Provision to [Packet.com](https://packet.com)
- [x] Automatically update Service type LoadBalancer with a public IP
- [x] Tunnel `http` traffic
- [ ] Garbage collect hosts when CRD is deleted
- [ ] Provision to EC2
- [ ] Provision to GCP
- [ ] Provision to DigitalOcean

## Video demo

[![https://img.youtube.com/vi/LeKMSG7QFSk/0.jpg](https://img.youtube.com/vi/LeKMSG7QFSk/0.jpg)](https://www.youtube.com/watch?v=LeKMSG7QFSk&amp=&feature=youtu.be)

Watch me get a LoadBalancer with a public IP for my KinD cluster and Nginx which is running there.

## Try the operator

Run the operator - example with KinD:

Sign up to Packet.com and get an access key, save it in `~/packet-token`

```sh
kubectl apply ./aritifacts/crd.yaml

export PACKET_PROJECT_ID=""	# Populate from dashboard

go build && ./inlets-operator  --kubeconfig "$(kind get kubeconfig-path --name="kind")" --access-key=$(cat ~/packet-token) --project-id="${PACKET_PROJECT_ID}"
```

Example usage:

```sh
kubectl run nginx-1 --image=nginx --port=80 --restart=Never --restart=Always
kubectl run nginx-2 --image=nginx --port=80 --restart=Never --restart=Always

kubectl expose deployment nginx-1 --port=80 --type=LoadBalancer
kubectl expose deployment nginx-2 --port=80 --type=LoadBalancer

kubectl get svc

kubectl get tunnel nginx-tunnel-1 -o yaml

kubectl get svc

kubectl logs deploy/nginx-1-tunnel-client
```

Check the IP of the LoadBalancer and then access it via the Internet.

Example with OpenFaaS, make sure you give the port a name of `http`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: gateway
  namespace: openfaas
  labels:
    app: gateway
spec:
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
      nodePort: 31112
  selector:
    app: gateway
  type: LoadBalancer
  ```
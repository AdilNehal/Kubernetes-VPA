# Install the kube server metrics
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# If you face "Readiness probe failed: HTTP probe failed with statuscode: 500" issue with metric server pod then add the following:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls # add this field 

# Get the pods
kubectl get deployment metrics-server -n kube-system

# The pre-requisite to deploy the VPA is to clone the project provided on the below link:
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler

# Execute script to setup VPA
./vertical-pod-autoscaler/hack/vpa-up.sh

# Validate that 3 VPA pods are running in the kube-system namespace
kubectl get pods -n kube-system | grep -i vpa

# If you find the following issue "MountVolume.SetUp failed for volume "tls-certs" : secret "vpa-tls-certs" not found" then
vi ./vertical-pod-autoscaler/pkg/admission-controller/gencerts.sh
"/CN=${CN_BASE}_ca" -> "//CN=${CN_BASE}_ca"
./vertical-pod-autoscaler/hack/vpa-up.sh

kubectl apply -f vpa.yaml -f deployment.yaml

# After few minutes, pods will be evicted & new pods will be running of the vpa-check with updated resources by vpa




# Kubernetes user management

## Create a namespace for testing

```sh
NAMESPACE="finance"
USER="john"
GROUP="finance-dept"

kubectl create ns ${NAMESPACE}
```

## Create Certificate / Keypair for a user

```sh
# create private key for user
openssl genrsa -out ${USER}.key 2048

# create Certification Request
openssl req -new -key ${USER}.key -out ${USER}.csr -subj "/CN=$USER/O=${GROUP}"

# copy CA to sign cert request
scp root@servername:/etc/kubernetes/pki/ca.crt ca.crt
# or
kubectl cp kube-apiserver-docker-desktop://run/config/pki/ca.crt -n kube-system ca.crt


# copy private key
scp root@servername:/etc/kubernetes/pki/ca.key ca.key
# or
kubectl cp kube-apiserver-docker-desktop://run/config/pki/ca.key -n kube-system ca.key

# create certificate
openssl x509 -req -in ${USER}.key -CA ca.crt -CAkey ca.key -CAcreateserial -out ${USER}.crt -days 365
```

## Create kubeconfig file

```sh
# create the configfile
kubectl --kubeconfig ${USER}.kubeconfig set-cluster kubernetes --server https://${DNS_NAME}:6443 --certificate-authority=ca.crt

# add the user
kubectl --kubeconfig ${USER}.kubeconfig config set-credentials ${USER} --client-certificate=${PWD}/${USER}.crt --client-jey=${PWD}/${USER}.key

# set the context
kubectl --kubeconfig ${USER}.kubeconfig config set-context ${USER}-kubernetes --namespace ${NAMESPACE} --user ${USER}

# TODO: check current-context


```
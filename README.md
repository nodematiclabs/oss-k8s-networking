# OSS K8s Networking (Cert Manager and Nginx) Demo

## Installation

Set environment variables (replace with your own GCP project)
```bash
export GOOGLE_PROJECT=example
```

Create a sandbox cluster
```bash
gcloud container clusters create oss-networking-sandbox \
    --zone=us-central1-a \
    --workload-pool=$GOOGLE_PROJECT.svc.id.goog \
    --scopes https://www.googleapis.com/auth/cloud-platform
gcloud container clusters get-credentials oss-networking-sandbox --zone=us-central1-a
```

Create a service account with DNS Admin permissions for Cert Manager
```bash
gcloud iam service-accounts create cert-manager \
    --display-name="Cert Manager"
gcloud projects add-iam-policy-binding $GOOGLE_PROJECT \
    --member="serviceAccount:cert-manager@$GOOGLE_PROJECT.iam.gserviceaccount.com" \
    --role="roles/dns.admin"
gcloud projects add-iam-policy-binding $GOOGLE_PROJECT \
    --member="serviceAccount:$GOOGLE_PROJECT.svc.id.goog[default/cert-manager]" \
    --role="roles/iam.workloadIdentityUser"
gcloud iam service-accounts keys create cert-manager.json \
    --iam-account="cert-manager@$GOOGLE_PROJECT.iam.gserviceaccount.com"
export GOOGLE_APPLICATION_CREDENTIALS=$PWD/cert-manager.json
```

Install Cert Manager
```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --set installCRDs=true
kubectl annotate serviceaccount \
    cert-manager \
    "iam.gke.io/gcp-service-account=cert-manager@$GOOGLE_PROJECT.iam.gserviceaccount.com" \
    --overwrite
```

Modify the [cert-manager.yaml](cert-manager.yaml) to use your domain, then create Cert Manager certificates/configurations
```bash
kubectl apply -f cert-manager.yaml
```

Install Nginx Ingress Controller
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
```

Apply any test ingress configurations:
1. `kubectl apply -f cookies.yaml`
2. `kubectl apply -f hosts.yaml`
3. `kubectl apply -f http-https.yaml`
4. `kubectl apply -f query-parameters.yaml`
5. `kubectl apply -f regex.yaml`

Set a DNS A record for the ingress domains

(If testing the cookies ingress, run `document.cookie='Example=always'` in your browser console)
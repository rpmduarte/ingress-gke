# GKE Ingress External LB

## Description

This is an example for setting up GKE native external LB ingress, it will work for a customer named "abcd" and for a given application.
Learn more about Ingress on the main [Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/) documentation site.

## Pre-requisites
- **[1]**
Add the following annotations in the services you want to expose; see example below for each of the services present to be exposed.  

[Documentation](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-features)

**otcs-frontend service**
```yaml
cloud.google.com/backend-config: '{"ports": {"80": "abcd-backend"}}'
cloud.google.com/neg: '{"ingress": true}'
```

**otac-0 service**
```yaml
cloud.google.com/backend-config: '{"ports": {"8080": "abcd-ac-backend"}}'
cloud.google.com/neg: '{"ingress": true}'
```

**otds service**
```yaml
cloud.google.com/backend-config: '{"ports": {"80": "abcd-otds-backend"}}'
cloud.google.com/neg: '{"ingress": true}'
```

Let me explain the first annotation (cloud.google.com/backend-config); the port in the backend config needs to match the exposed service port, in the example below
the port we need to add to the annotation will be the "port: 80", you'll be able to find that in the service configuration yaml file as per example:
```yaml
  ports:
  - name: service1
    nodePort: 32041
    port: 80
    protocol: TCP
    targetPort: 8080
```
The second part of the annotation needs to match the name of the "kind: BackendConfig" that you'll have to create/customize for the application.
"abcd-otds-backend" in this case stands for customername-service-kind

- **[2]** 
An external IP in GCP already configured, the name will be required for the ingress file as an annotation.

- **[3]** 
Firewall rules for the health checks [Documentation](https://cloud.google.com/load-balancing/docs/health-checks#firewall_rules)  
The Ingress will try and create the rules automatically but will fail with lack of permissions (in the EMS world), as such
you'll have to have them pre-configured in the host project. 
At the time of writing this README, the network team has in place a broad firewall rule that allows the health check traffic.  
NOTE: This wasn't tested in a Private-VPC model.

- **[4]** 
Health checks will require an HTTP-200 reply from the application POD, please obtain a path from engineering that will return this code.

- **[5]**
A Certificate needs to be created and the crt and key added in base64 encoding to the secret.yml
(this will eventually be automated with Terraform)

- **[6]** 
The hosts and path types need to be supplied by Engineering

- **[7]**
An ssl security policy needs to exist in the project, this is created by default with Terraform for every ems project:
```terraform
resource "google_compute_ssl_policy" "ssl-policy" {
  name            = "tls-policy"
  profile         = "MODERN"
  min_tls_version = "TLS_1_2"
  depends_on = [
    google_project.customer_project
  ]
}
```
## Get started
Create a backend configuration for each service that includes the affinity type and health check.
Please keep in mind the health check path needs to return an HTTP-200 code.
_IMPORTANT_: the health check port needs to match the targetPort in the service and not the Service Port.
[Documentation](https://cloud.google.com/load-balancing/docs/health-checks)

Create a FrontendConfig file with a relevant name, use the sslPolicy name, configure the http to https redirection.  
[Documentation](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-features#configuring_ingress_features)

Create a secret file with the external cert details.
[Documentation](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets)

Create an Ingress file that maps to the frontendConfig name. All the annotations are explained in Google documentation.
Map the secret name that contains the certificate in under tls, as well as the hosts that are to be exposed.


## Deploy

Add the yaml files to be deployed to the kustomization.yaml  
Connect to a GKE cluster, pull the repo from git.
CD one folder back and deploy:
```bash
kubectl kustomize *foldername* | kubectl -n *namespaceToDeployTo* apply -f -
```

## Troubleshooting
The ingress health checks can take quite some time to show as green, if you want to double check if the 
health checks are coming up, go to, Compute Engine -> Network Endpoint Groups and find the NEG and look for the health check status at the bottom.


## Contributing
Feel free to branch out and create a merge request. This was a POC that will need to be automated.

## Changelog


# Get Involved

- **Contributing**: Pull requests are welcome!

## Issues


## License

Rui Duarte 2021

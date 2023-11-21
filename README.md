
============================================================================
Document Version: v1
Created By: Dharmang Makwana
Agenda: EKS Cluster ISTIO Implementation for end-end SSL
Targeted Audiance: Should be familier with Kubernetes and AWS ACM, EKS, LB
============================================================================

Pre-Requsite:
1) Working EKS Cluster with ALB Controller installed
2) ACM ARN with SSL certificate uploaded
3) Linux/Windows machine with OpenSSL installed

Step 1) Installation of ISTIO
		# curl -L https://istio.io/downloadIstio | sh -
		# cd istio-1.20.0
		# export PATH=$PWD/bin:$PATH

Step 2) Configuration of the ISTIO Profile
		# istioctl install \
		  --set profile=demo \
		  --set values.gateways.istio-ingressgateway.type=NodePort

Step 3) Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later (devops is the name of the namespace)
		# kubectl label namespace devops istio-injection=enabled

Step 4) Creation of Application
		4.1) Application Deployment and its Cluster Type service creation
			# kubectl apply -f helloworld.yml
			# kubectl apply -f helloworldsvc.yml
			# kubectl apply -f nginx.yml
			# kubectl apply -f nginxsvc.yml

		4.2) Creation of ALB Ingress G.W => Do change the ACM ARN & Host names accordingly
			# kubectl apply -f ingress.yml

		4.3) Generate self-signed certificates. We will use a key/pair to encrypt traffic from ALB to Istio Gateway.
			# openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
			  -keyout certs/key.pem -out certs/cert.pem -subj "/CN=yourdomain.com" \

		4.4) Generate Kubernetes secret containing key.pem and cert.pem. We will use it with Istio Gateway to implement traffic encryption.
			# kubectl create -n istio-system secret generic tls-secret \
			  --from-file=key=certs/key.pem \
			  --from-file=cert=certs/cert.pem

		4.5) Creation of Ingress G.W and Virtual Services
			# kubectl apply -f gw.yml
			# kubectl apply -f vs.yml

Step 5) Testing of the Application.
		Expected Result:
			5.1) Auto Redirection from http to https should work
			5.2) ALB to Ingress G.W communication is encrypted
			5.3) Ingress G.W to ISTIO Side card Proxy communication is encrypted

# Load Balancer controller

De controller installeert een ALB (Amazon Load Balancer) zodra een ingress wordt gestart. De ingress wordt geinstalleerd op de ALB.

Trivia:
- Maakt gebruik van service account dat is aangemaakt door eksctl in eks-demo-infra project
- De variabele AWS_ACCOUNT_ID is een secret. De substitutie wordt voorbereid in de kustomization in het eks-demo-infra project. Zie voorbeeld. De veriabele is een value in 'aws' secret. 

Voorbeeld ingress:

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: zzz
      annotations:
        # Ingress class geeft aan dat het een AWS ALB is
        kubernetes.io/ingress.class: "alb"

        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/target-type: ip

        # Default krijgt iedere ingress een eigen ALB (kost geld), alle ingresses met dezelfde group delen een ALB
        alb.ingress.kubernetes.io/group.name: zzz

        # IPv6, zie notities in README in eks-demo-infra
        alb.ingress.kubernetes.io/ip-address-type: ipv4
        alb.ingress.kubernetes.io/subnets: subnet-0fc025155115d3ca9, subnet-0f1f930cfb2b3909d  #, subnet-08991736d58ec2e49

        # SSL
        external-dns.alpha.kubernetes.io/hostname: dev.liberaalgeluid.nl, acc.liberaalgeluid.nl, test.liberaalgeluid.nl
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}, {"HTTP": 8080}, {"HTTPS": 8443}]'
        alb.ingress.kubernetes.io/ssl-redirect: '443'
        alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:${AWS_ACCOUNT_ID}:certificate/6776643f-a45b-47a9-97c7-f8999fb3ce61

        # Health Check Settings
        alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
        alb.ingress.kubernetes.io/healthcheck-port: traffic-port
        alb.ingress.kubernetes.io/healthcheck-path: /actuator/health
        alb.ingress.kubernetes.io/healthcheck-interval-seconds: '15'
        alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
        alb.ingress.kubernetes.io/success-codes: '200'
        alb.ingress.kubernetes.io/healthy-threshold-count: '2'
        alb.ingress.kubernetes.io/unhealthy-threshold-count: '2'
      namespace: zzz


Variabele substitutie

    ---
    apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
    kind: Kustomization
    metadata:
      name: eks-demo-app
      namespace: flux-system
    spec:
      postBuild:
        substituteFrom:
          - kind: Secret
            name: aws

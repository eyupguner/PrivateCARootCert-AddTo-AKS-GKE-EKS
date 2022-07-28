# PrivateCARootCert-AddTo-AKS-GKE-EKS
Many enterprises/people use self-signed certificates to protect apps or services. On kubernetes this can present a problem as docker/containerd requires that self-signed certificates be nested under the necesarry paths on every host. This restriction can be solved with a DaemonSet that copies the cacert file to the needed directory on each host. This deployment yaml for adding custom certification authority root certificate to cloud kubernetes clusters.

#Attention

Require 1.17+ Kubernetes Version

This deployment for ubuntu/debian base node pool with containerd. If you are using another os node pool like coreos, cos_containerd or docker engine you have to change following part of the yaml. 

#definevolume

      volumes:
      - name: host-ca
        hostPath:
          path: /usr/local/share/ca-certificates/

#mountvolume

         volumeMounts:
        - name: host-ca
          mountPath: /usr/local/share/ca-certificates/
          
#argument's-paths-and-command

        args:
          - -c
          - |
            cp /tmp/ecomyd-ca /usr/local/share/ca-certificates/ecomyd-ca.crt
            nsenter --mount=/proc/1/ns/mnt -- sh -c "update-ca-certificates && systemctl restart containerd"
            
#CAcertificateBase64Encode

A secret will be mounted into the DaemonSet as a file and then copied. This secret must include the base64 encoded contents of the root CA (pem format) used to sign the container registry certs.

    base64 -w 0 private-cacert.pem
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURrRENDQW5pZ0F3SUJBZ0lRSVlES1YvUU8zcDlPYVVUSWpXaFFlVEFOQmdrcWhraUc5dzBCQ.......

      apiVersion: v1
      kind: Secret
      metadata:
        name: private-cacert
        namespace: kube-system
      type: Opaque
      data:
        private-cacert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURrRENDQW5pZ0F3SUJBZ0lRSVlES1YvUU8zcDlPYVVUSWpXaFFlVEFOQmdrcWhraUc5dzBCQ.......

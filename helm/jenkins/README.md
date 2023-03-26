# Jenkins 
## helm 3 install documents 참고
[install](https://www.jenkins.io/doc/book/installing/kubernetes/)
[install with Helm3](htps:t//www.jenkins.io/doc/book/installing/kubernetes/#install-jenkins-with-helm-v3)

```
# jenkinsci/jenkins, jenkins/jenkins가 있는데 tag버전이 동일한거 보니 경로만 다른듯? 확실치 않음. 
> helm search repo -l jenkinsci/jenkins
> helm show values jenkinsci/jenkins > jenkins.values.yaml
> helm pull jenkinsci/jenkins --version 4.3.9 --untar

# Create a pv
> curl -LO https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-volume.yaml
# namespace -> devops-tools, worker node1에 셋팅되도록 설정하였음. 

> k describe pv/jenkins-pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                           STORAGECLASS   REASON   AGE
jenkins-pv   10Gi       RWO            Retain           Available   devops-tools/jenkins-pv-claim   jenkins-pv              67s

# worker node1 - ssh 
> sudo chown -R 1000:1000 /mnt/jenkins

> k get pvc,pv --namespace=devops-tools
NAME                                     STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS            AGE
persistentvolumeclaim/jenkins-pv-claim   Bound    jenkins-pv-volume   10Gi       RWO            local-storage-storage   119s
NAME                                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                           STORAGECLASS            REASON   AGE
persistentvolume/jenkins-pv-volume   10Gi       RWO            Retain           Bound    devops-tools/jenkins-pv-claim   jenkins-local-storage            119s


# Create a service account 
> curl -LO https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-sa.yaml

> k get clusterrole
> k get ClusterRoleBinding

# Install Jenkins 
> curl -LO https://raw.githubusercontent.com/jenkinsci/helm-charts/main/charts/jenkins/values.yaml

# serviceAccount: create: false로 수정 

controller.serviceType=NordPort 
persistence.existingClaim=jenkins-pv-claim
serviceAccount.create=false
serviceAccount.name=jenkins-admin
controller.resources.requests.cpu=500m
controller.resources.requests.memory=512Mi 

> h install jenkins jenkinsci/jenkins --version 4.3.9 -f ./jenkins.values.yaml --namespace=devops-tools --dry-run

> kubectl get all -l app.kubernetes.io/name=jenkins

> k describe statefulset.apps/jenkins --namespace=devops-tools


NOTES:
1. Get your 'admin' user password by running:
  kubectl exec --namespace devops-tools -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  export NODE_PORT=$(kubectl get --namespace devops-tools -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
  export NODE_IP=$(kubectl get nodes --namespace devops-tools -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

3. Login with the password from step 1 and the username: admin
4. Configure security realm and authorization strategy
5. Use Jenkins Configuration as Code by specifying configScripts in your values.yaml file, see documentation: http://$NODE_IP:$NODE_PORT/configuration-as-code and examples: https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos

```
1. NOTES
 - admin pwd: gvmxJGX9gyjDrZvLro0kFD
 - Jenkins URL - http://192.168.56.10:30701 
    1. virtual-box port-forwarding 설정 
        - 60016(clinet - port) : 30701(vmware - nodePort)
    2. 접속 
        - http://localhost:60015/login?from=%2F
 - 





## 레퍼런스 
https://arisu1000.tistory.com/27848
https://velog.io/@aylee5/EKS-Helm%EC%9C%BC%EB%A1%9C-Jenkins-%EB%B0%B0%ED%8F%AC-MasterSlave-%EA%B5%AC%EC%A1%B0-with-Persistent-VolumeEBS
https://m.blog.naver.com/alice_k106/221360324682
https://stackoverflow.com/questions/63490278/kubernetes-persistent-volume-hostpath-vs-local-and-data-persistence
# 쿠버네티스 컨테이너 이슈 대응

</br>

# 개요

2023년 4월 3일부터 쿠버네티스 프로젝트의 레지스트리가 기존 [k8s.gcr.io](http://k8s.gcr.io) 레지스트리에서 새로운 레지스트리인 [registry.k8s.io](http://registry.k8s.io) 레지스트리로 변경됨에 따라 [k8s.gcr.io](http://k8s.gec.io) 레지스트리를 참조하는 이미지에 대한 지원이 중단되었습니다. 이에 따라 기존 이미지들에 대해 배포가 중단되거나 임의로 제거될 수 있기때문에 서비스 중인 클러스터가 사용중인 모든 이미지에 대해 검사를 실시하고 이슈가 있는 이미지 레지스트리를 새로운 커뮤니티 레지스트로 변경하고자 합니다.

</br>

- 이슈 전문
    
    <aside>
    💡 안녕하세요
    
    2023년 4월 3일부터 Kubernetes 프로젝트는 다음 블로그 [1] 에서 발표한 바와 같이 커뮤니티 소유 이미지 (커뮤니티 이미지라고도 함) 를 이전 이미지 레지스트리 `k8s.gcr.io`에 게시하는 것을 중단할 예정이며 이에 따라 `k8s.gcr.io` 레지스트리가 정지됩니다. Kubernetes 및 관련 하위 프로젝트에 대한 추가 이미지는 레지스트리로 푸시되지 않습니다. 필요하다고 사료될 경우, Kubernetes 프로젝트는 기존 레지스트리에서 기존 커뮤니티 이미지를 제거할 권리를 보유합니다. 기본적으로 EKS 클러스터는 커뮤니티 소유 레지스트리에서 호스팅되는 이미지에 의존하지 않습니다. 애플리케이션이 레거시 레지스트리에서 커뮤니티 소유 이미지를 사용하도록 구성된 경우 가용성 문제가 발생할 수 있습니다. 애플리케이션과 해당 종속성을 업데이트하여 새 이미지 레지스트리인 `registry.k8s.io`에서 이미지를 가져와 문제를 방지할 수 있습니다. 새 레지스트리에는 이전 레지스트리에서 가져온 모든 커뮤니티 이미지와 이미지 태그가 들어 있습니다. 
    
    <무엇이 문제일까요?>
    2022년 11월 28일, Kubernetes 프로젝트는 `k8s.gcr.io` 레지스트리에서 커뮤니티 이미지를 새로운 커뮤니티 소유 레지스트리인 `registry.k8s.io` [2] 로 옮길 것이라고 발표했습니다. 이 새 레지스트리는 Kubernetes 컨테이너 이미지를 위한 콘텐츠 전송 네트워크 (CDN) 처럼 작동하여 여러 클라우드 제공업체 및 지역에 부하를 분산합니다. 이 변경으로 인해 단일 엔티티에 대한 프로젝트의 의존도가 낮아지고 사용자에게 더 빠른 다운로드 환경이 제공됩니다. Kubernetes 프로젝트가 기본값을 새 이미지 레지스트리로 변경했지만, 이전 레지스트리를 계속 지원하였습니다. 하지만, 2023년 4월 3일부터 이전 `k8s.gcr.io` 레지스트리가 동결됩니다. Kubernetes 프로젝트는 커뮤니티 이미지를 이전 레지스트리에 게시하는 것을 중단합니다. 레지스트리에 있는 기존 커뮤니티 이미지는 향후 제거될 수 있습니다. 이전 `k8s.gcr.io` 레지스트리에서 호스팅되는 이미지에 의존하여 배포가 이뤄지는 경우, 애플리케이션에 가용성 위험이 발생할 수 있습니다.
    
    <영향을 받는지 어떻게 확인할 수 있나요?>
    애플리케이션이 레거시 레지스트리에서 사용하는 커뮤니티 이미지를 식별하는 단계별 방법은 이미지 레지스트리 리디렉션 설명서 [3] 를 참고하시기 바랍니다.
    
    또한 새 kubectl 플러그인 커뮤니티 이미지를 실행하여 Amazon EKS 클러스터에서 실행 중인 리소스 중 이전 이미지 레지스트리에서 커뮤니티 이미지를 가져오는 리소스를 식별하실 수 있습니다. 커뮤니티 이미지 플러그인 설치 및 실행 지침은 해당 페이지 [3] 에서 확인할 수 있습니다. 
    
    또는 kubectl을 사용하여 이전 이미지 레지스트리에서 커뮤니티 이미지를 사용하는 클러스터에서 실행 중인 파드(pod)를 식별하여 파드에서 사용하는 이미지를 나열하실 수 있습니다. kubectl 명령은 해당 페이지 [4] 에서 확인하실 수 있습니다.
    
    <문제를 어떻게 해결할 수 있나요?>
    이전 이미지 레지스트리에서 커뮤니티 이미지를 가져오는 애플리케이션을 식별한 후 파드, 배포 매니페스트 및 helm 차트와 같은 리소스에 대한 정의를 업데이트하시기 바랍니다. 리소스 정의에서 컨테이너 이미지의 이미지 레지스트리를 `k8s.gcr.io`에서 `registry.k8s.io`로 변경하시기 바랍니다. 이렇게 하면 이후의 모든 이미지 가져오기 요청이 새 이미지 레지스트리에 전달됩니다. 커뮤니티에서는 현재 `registry.k8s.io`로 자동 리디렉션을 설정했지만, 향후 클러스터 불안정성을 방지하려면 커뮤니티의 권장 사항 [5] 에 따라 새 레지스트리를 사용하도록 리소스 정의를 업데이트하는 것을 권장 드립니다.
    
    AWS 블로그 [6], EKS 뉴스레터 [7] 및 AWS Container from the Couch 동영상 [8] 을 통해 이러한변경 사항에 대응하실 수 있도록 지속적으로 업데이트와 지침을 게시하고 있습니다. 추가질문이나 우려 사항이 있는 경우 AWS Support [9] 에 문의하시기 바랍니다.
    
    [1]  https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/#timeline-of-the-changes 
      
    [2]  https://kubernetes.io/blog/2022/11/28/registry-k8s-io-faster-cheaper-ga/
      
    [3]  https://kubernetes.io/blog/2023/03/10/image-registry-redirect/
      
    [4]  https://github.com/kubernetes-sigs/community-images#kubectl-community-images
      
    [5]  https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/#what-s-next
      
    [6]  https://aws.amazon.com/blogs/containers/changes-to-the-kubernetes-container-image-registry/
      
    [7]  https://eks.news/
      
    [8]  https://www.youtube.com/shorts/5RvrkLPImGQ
      
    [9]  https://aws.amazon.com/support
    
    </aside>
    

</br>
      
# 검사

이슈 레지스트리는 두가지 방법으로 체크가 가능합니다.

첫번째 방법으로는 터미널에서 다음의 명령을 실행하는 것입니다.

```jsx
$ kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```

이 명령은 현재 접속중인 클러스터의 모든 네임스페이스 파드 컨테이너 이미지들에 대해 리스트를 출력합니다.  

![1](https://user-images.githubusercontent.com/124660559/228406026-15547e1d-35af-491d-8303-d6556f7512da.png)

리스트의 내용을 참조하여 [k8s.gcr.io](http://k8s.gcr.io) 레지스트리를 사용하는 이미지를 확인할 수 있습니다. 제가 접속중인 클러스터에서는 External-dns 이미지가 이슈 레지스트리를 사용하고 있는 것으로 확인되었습니다.

만약 클러스터에 사용하는 이미지가 많거나 이슈 레지스트리 이미지가 많아서 좀더 시각적으로 체크하고 싶다면 두번째 방법을 추천합니다. Kubectl Krew 매니저에서 제공하는 Community-images 플러그인을 사용하는 방법입니다.

 

먼저 Krew 매니저가 설치되어있지 않다면 다음 명령어를 통해 설치해줍니다.

```jsx
$ (
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
) 
```

Krew 설치후엔 쉘 환경변수에 설치한 플러그인 매니저를 등록해줍니다.

```jsx
$ export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

등록 후에는 Community-images 플러그인을 설치합니다.

```jsx
$ kubectl krew install community-images
```

등록과 설치 과정을 마친 후에는 다음 명령어를 통해 클러스터 이미지를 검사할 수 있습니다.

```jsx
$ kubectl community-images
```
<img width="526" alt="2" src="https://user-images.githubusercontent.com/124660559/228406116-d0c9bbf1-60ca-4bbf-8d9b-fe3d8ff9b549.png">


첫번째 방법과 마찬가지로 external-dns 이미지에 대한 이슈를 확인할 수 있습니다.

      
</br>
  
# 업데이트

검사된 이미지에 대해서 레지스트리를 업데이트해주는 작업은 여러가지 방법이 있습니다. 

가장 많이 쓰이는 쿠버네티스 패키지 매니저 헬름은 이미 대부분의 이미지를 새로운 [registry.k8s.io](http://registry.k8s.io) 레지스트리로 업데이트되었기때문에 헬름 레포지토리를 업데이트한 뒤 재설치를 해주는 과정을 거치면 됩니다.

```jsx
// helm 레포지토리 업데이트
$ helm repo update
```

레포지토리를 업데이트한 이후 헬름을 통해 이슈 이미지를 업그레이드 및 재생성할 수 있습니다.

```jsx
$ helm upgrade <RELEASE_NAME> <CHART_NAME> --namespace <NAMESPACE> --recreate-pods
```

<img width="577" alt="3" src="https://user-images.githubusercontent.com/124660559/228406268-477ba5a9-b427-4805-919e-996458b67cbc.png">


재생성시 Image tag를 확인해보면 이슈가 해결된 새로운 레지스트리로 변경되었음을 확인할 수 있습니다.

이 방법은 레지스트리를 업데이트하기 위해 리소스를 재설치하는 과정을 거치기때문에 의존성을 가지거나 config등을 지니는 경우 복잡한 문제가 발생할 수 있기때문에 해당 리소스의 이미지를 직접 변경해주는 방법을 사용할 수 있습니다.

```jsx
$ kubectl edit <RESOURCE_TYPE> <RESOURCE_NAME>
```

<img width="484" alt="4" src="https://user-images.githubusercontent.com/124660559/228406283-1d3b160f-7bf7-4faa-83e6-348697cebd59.png">

Containers 의 Image를 직접 [k8s.gcr.io](http://k8s.gcr.io) 에서 [registry.k8s.io](http://registry.k8s.io) 로 수정 후 저장해줍니다. 쿠버네티스 커뮤니티에 따르면 대부분의 이미지를 

변경 이후 다음 명령어를 통해 디플로이먼트를 재시작하면 됩니다.

```jsx
$ kubectl rollout restart deployment/<DEPLOYMENT_NAME>
```

<img width="440" alt="5" src="https://user-images.githubusercontent.com/124660559/228406305-60b56460-8be4-44f1-b195-e4cc7a07c40d.png">

Community-images를 통해서도 정상적인 레지스트리로 변경되었음을 확인할 수 있습니다.

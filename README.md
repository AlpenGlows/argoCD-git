# Yeni akışın sırası (GitOps’lu düzen)

1. **Dev → GitHub’a push**
    
    `hello world 1` → `2` değişikliğini commitledin.
    
2. **(CI) GitHub → Jenkins tetiklenir**
    
    *Evet, CI hâlâ webhook’la tetiklenir.*
    
    - Jenkins build + test yapar.
    - Docker image üretir ve **Docker Registry (v2)**’ye **push** eder (örn. `myapp:fa85650`).
3. **(Bridge) Argo CD Image Updater devreye girer**
    - Registry’de yeni **image tag**’i gördüğünde, **Git’teki** deployment repo’suna (ops repo) **PR/commit** açar:
        
        `values.yaml` içindeki `image.tag` alanını `fa85650` ile **günceller**.
        
4. **(CD) Argo CD Git’i izler ve senkronize eder**
    - Argo CD, ops repo’daki bu commit’i görür.
    - İlgili **Helm chart + values.yaml** ile **Kubernetes**’e **rollout** yapar (ilgili **namespace**’te).
5. **Kubernetes çalıştırır**
    - Yeni Pod’lar ayağa kalkar, Service/Ingress/PVC vs. uygulanır.
    - Uygulama yeni sürümle canlıdır.

```
Dev → Git push
   │
   ├─► (Webhook) Jenkins (CI): build + test + image push → Registry
   │
   ├─► Argo CD Image Updater: values.yaml'da image.tag'ı günceller (Git PR/commit)
   │
   └─► Argo CD: Git'teki chart+values'a göre Kubernetes'e deploy (CD)
                      │
                      └─► K8s rollout → yeni sürüm çalışıyor

```

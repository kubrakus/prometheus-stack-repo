# Prometheus Stack Repository

Bu repo, Kubernetes üzerinde Prometheus tabanlı gözlemlenebilirlik (observability) altyapısını kurmak ve yönetmek için hazırlanmıştır.  
Helm chart'ları, alert kuralları, ingress tanımları ve "app-of-app" yaklaşımıyla merkezi bir yapı sunar.

## 📂 Dizin Yapısı

- **prometheus-stack/**  
  Prometheus, Grafana ve ilgili bileşenleri deploy etmek için Helm chart ve değer dosyalarını içerir.

- **alerts/**  
  Alertmanager kuralları ve özelleştirilmiş uyarı tanımlarını içerir.

- **monitoring-ingress/**  
  Monitoring servislerine (örn. Grafana, Alertmanager, Prometheus) erişim için Ingress kaynaklarını içerir.

- **app-of-app/**  
  ArgoCD "App of Apps" yaklaşımını kullanarak tüm monitoring stack'in yönetimini sağlar.

## 🚀 Ön Koşullar

- Kubernetes cluster (v1.22+ önerilir)  
- [Helm](https://helm.sh/) (v3+)  
- Opsiyonel: [ArgoCD](https://argo-cd.readthedocs.io/) (app-of-app için)

---

### 1. Kurulum Yöntemi

Prometheus Stack, manuel Helm komutları ile değil, **ArgoCD Application CRD** manifest’leri kullanılarak kurulmuştur.  
Bu yaklaşım, versiyon kontrol, tekrarlanabilir kurulum ve merkezi yönetim avantajı sağlar.

- `prometheus-community/kube-prometheus-stack` Helm chart’ı temel alınmıştır.
- İlgili Helm değerleri (`values.yaml`) repository’de versiyon kontrol altında tutulmuştur.
- Alert kuralları ayrı manifest dosyaları halinde oluşturulmuş ve Prometheus tarafından otomatik olarak yüklenmesi sağlanmıştır.
- İzleme bileşenlerine (Prometheus, Grafana, Alertmanager) dışarıdan erişim için **Ingress** tanımı yapılmıştır.

> **Not:** Bu kurulum, case’deki "node ve servis metriklerini izleme" ile "3 adet kritik alert tanımlama" gereksinimini karşılar.

📂 Açıklamalar

app-of-app/ → ArgoCD’de "Application of Applications" yapısını sağlayan manifest’ler.

prometheus-stack/ → Helm chart values.yaml dosyası ve kurulum parametreleri.

alerts/ → 3 adet kritik alert kuralı (Node, Pod, API Server).

monitoring-ingress/ → Prometheus, Grafana ve Alertmanager’a erişim için Ingress tanımı.

---

### 2. Kurulum Adımları

***Adım 1 — CRD’lerin Yüklenmesi***

Prometheus Operator’un çalışabilmesi için gerekli CustomResourceDefinition’lar (`promentheus-crd-application.yaml`) deploy edildi.

***Adım 2 — Prometheus Stack Kurulumu***

Prometheus-stack-application.yaml ArgoCD Application manifest’i uygulanarak Prometheus Stack Helm chart’ı yüklendi.
Bu chart ile birlikte:

- Prometheus
- Grafana
- Alertmanager
- Node Exporter
- Kube State Metrics

bileşenleri otomatik kuruldu.

***Adım 3 — Alert Tanımlarının Eklenmesi***

3 adet kritik alert kuralı ayrı dosyalar halinde oluşturuldu:

**Node Not Ready →** 1 dakika boyunca node NotReady durumunda kalırsa tetiklenir.

Dosya: `alerts/node-not-ready.yaml`

**Pod Alerts →** Tüm pod’lar için genel sağlık kontrolleri.
Örneğin CrashLoopBackOff veya Pending durumları 1 dakikadan uzun sürerse alarm verir.

Dosya: `alerts/pod-alerts.yaml`

**API Server Down →** Kube API server erişilemezse 1 dakika içinde alarm verir.

Dosya: `alerts/apiserver-down.yaml`

***Adım 4 — İzleme Servislerine Erişim***

`monitoring-ingress.yaml` ile:

- Grafana → `grafana.kubikolog.com`
- Prometheus → `prometheus.kubikolog.com`
- Alertmanager → `alertmanager.kubikolog.com`
adresleri üzerinden HTTPS ile erişim sağlanmıştır.

Bu yapı sayesinde:

Kubernetes cluster’ındaki tüm kritik bileşenler izlenmekte,

Anlık problemler Alertmanager ile yakalanmakta,

Prometheus & Grafana arayüzlerinden geçmiş metrikler incelenebilmekte,

Tüm kurulum ve yapılandırma GitOps prensipleri ile ArgoCD üzerinden yönetilmektedir.

**Not:** Oluşturulan custom prometheus ruleların arayüzde görünür olabilmesi için aşağıdaki label alert yamllarına eklenmiştir.

```yaml  
labels:
    release: prometheus-stack
```

🛠️ Geliştirme ve Katkı

1. Fork yapın

2. Branch oluşturun (git checkout -b feature/foo)

3. Değişikliklerinizi commit edin

4. Pull Request açın 🎉

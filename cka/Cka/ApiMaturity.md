# API Maturity

## Content

### ❓ Describe the three-phase graduation process that new Kubernetes API resources go through from initial introduction to production readiness, including the key characteristics and risks associated with alpha API resources, how beta resources differ from alpha resources, and what commitments Kubernetes makes regarding GA resources.
Kubernetes has a well-documented three-phase graduation process for new API resources:

**Phase 1: Alpha**
Alpha resources are experimental and you should consider them hairy and scary. You should expect them to have bugs, drop features without warning, and change a lot when they move into beta. This is why most clusters disable them by default. Alpha resources are served with API versions like `v1alpha1`, `v1alpha2`, etc.

**Phase 2: Beta**
Beta resources are considered pre-release and should be very close to what the developers expect the final GA release to look like. However, it's normal to expect minor changes when promoted to GA. Most clusters enable beta APIs by default, and you'll occasionally see beta resources in production environments (though this is not a recommendation). Beta resources have API versions like `v1beta1`, `v1beta2`, etc.

**Phase 3: Generally Available (GA/Stable)**
GA resources are considered production-ready, and Kubernetes has a strong long-term commitment to them. When deprecated, Kubernetes continues to serve and support them for 12 months or three releases, whichever is longest. However, Kubernetes will only deprecate an existing stable resource after a newer stable version is available.

| API version                 | Track          |
| --------------------------- | -------------- |
| Alpha                       | Experimental   |
| Beta                        | Pre-release    |
| GA (Generally Available)    | Stable         |

### ❓ Describe the three-stage maturity model for Kubernetes API resources, the characteristics of each stage, and what is the deprecation policy guarantee for GA resources in Kubernetes and how long does it provide support?
All new resources enter the API as alpha, progress through beta, and hopefully graduate to GA. Alpha resources are considered experimental, and, therefore, subject to change and usually disabled on most clusters. Beta resources are considered pre-release and, therefore, more reliable and won't change much when they graduate to GA. Most clusters enable beta resources by default, but you should be cautious about using them in production. GA resources are considered stable and production-ready. Kubernetes has a strong commitment to GA resources and backs this up with a well-defined deprecation policy guaranteeing support for at least 12 months, or three versions, after the deprecation announcement.


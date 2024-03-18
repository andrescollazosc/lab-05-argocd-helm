# FROM argoproj/argocd:v1.8.7

# ARG SOPS_VERSION="v3.8.1"
# ARG HELM_SECRETS_VERSION="v4.6.0"
# ARG HELM_GCS_VERSION="0.3.7"
# ARG SOPS_PGP_FP="56A6229CEAF2C1FA855C3C90B52D05A203060B58"

# ENV SOPS_PGP_FP=${SOPS_PGP_FP}

# USER root  
# COPY helm-wrapper.sh /usr/local/bin/
# RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9
# RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6ED0E7B82643E131
# RUN apt-get update && \
#     apt-get install -y \
#     curl \
#     gpg && \
#     apt-get clean && \
#     rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
#     curl -o /usr/local/bin/sops -L https://github.com/mozilla/sops/releases/download/${SOPS_VERSION}/sops-${SOPS_VERSION}.linux && \
#     chmod +x /usr/local/bin/sops && \
#     cd /usr/local/bin && \
#     mv helm helm.bin && \
#     mv helm2 helm2.bin && \
#     mv helm-wrapper.sh helm && \
#     ln helm helm2 && \
#     chmod +x helm helm2

# # helm secrets plugin should be installed as user argocd or it won't be found
# USER argocd
# RUN /usr/local/bin/helm.bin plugin install https://github.com/jkroepke/helm-secrets --version ${HELM_SECRETS_VERSION}
# RUN /usr/local/bin/helm.bin plugin install https://github.com/hayorov/helm-gcs.git --version ${HELM_GCS_VERSION}
# ENV HELM_PLUGINS="/home/argocd/.local/share/helm/plugins/"

ARG ARGOCD_VERSION="v2.6.2"
FROM argoproj/argocd:$ARGOCD_VERSION
ARG SOPS_VERSION="3.8.1"
ARG VALS_VERSION="0.24.0"
ARG HELM_SECRETS_VERSION="4.6.0"
ARG KUBECTL_VERSION="1.26.1"
# vals or sops
ENV HELM_SECRETS_BACKEND="vals" \
    HELM_SECRETS_HELM_PATH=/usr/local/bin/helm \
    HELM_PLUGINS="/home/argocd/.local/share/helm/plugins/" \
    HELM_SECRETS_VALUES_ALLOW_SYMLINKS=false \
    HELM_SECRETS_VALUES_ALLOW_ABSOLUTE_PATH=false \
    HELM_SECRETS_VALUES_ALLOW_PATH_TRAVERSAL=false \
    HELM_SECRETS_WRAPPER_ENABLED=false

# Optionally, set default gpg key for sops files
# ENV HELM_SECRETS_LOAD_GPG_KEYS=/path/to/gpg.key

USER root
RUN apt-get update && \
    apt-get install -y \
      curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN curl -fsSL https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl \
    -o /usr/local/bin/kubectl && chmod +x /usr/local/bin/kubectl

# sops backend installation (optional)
RUN curl -fsSL https://github.com/getsops/sops/releases/download/v${SOPS_VERSION}/sops-v${SOPS_VERSION}.linux.amd64 \
    -o /usr/local/bin/sops && chmod +x /usr/local/bin/sops

# vals backend installation (optional)
RUN curl -fsSL https://github.com/helmfile/vals/releases/download/v${VALS_VERSION}/vals_${VALS_VERSION}_linux_amd64.tar.gz \
    | tar xzf - -C /usr/local/bin/ vals \
    && chmod +x /usr/local/bin/vals

RUN ln -sf "$(helm env HELM_PLUGINS)/helm-secrets/scripts/wrapper/helm.sh" /usr/local/sbin/helm

USER argocd

RUN helm plugin install --version ${HELM_SECRETS_VERSION} https://github.com/jkroepke/helm-secrets
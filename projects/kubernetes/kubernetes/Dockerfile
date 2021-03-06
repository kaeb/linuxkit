FROM linuxkit/alpine:79987c65c66700171c073151c1d3f0372597bec2 AS build

ENV kubernetes_version v1.7.6
ENV cni_version        v0.6.0

ENV kube_release_artefacts "https://dl.k8s.io/${kubernetes_version}/bin/linux/amd64"

RUN apk add -U --no-cache \
  curl \
  && true

RUN mkdir -p /out/etc/apk && cp -r /etc/apk/* /out/etc/apk/
#coreutils needed for du -B for disk image checks made by kubelet
# example: $ du -s -B 1 /var/lib/kubelet/pods/...
#          du: unrecognized option: B
RUN apk add --no-cache --initdb -p /out \
    alpine-baselayout \
    busybox \
    ca-certificates \
    coreutils \
    curl \
    ebtables \
    ethtool \
    iproute2 \
    iptables \
    libc6-compat \
    musl \
    socat \
    util-linux \
    && true
# Remove apk residuals. We have a read-only rootfs, so apk is of no use.
RUN rm -rf /out/etc/apk /out/lib/apk /out/var/cache

RUN rmdir /out/var/run && ln -nfs /run /out/var/run

RUN curl -fSL -o /out/root/cni.tgz https://github.com/containernetworking/plugins/releases/download/${cni_version}/cni-plugins-amd64-${cni_version}.tgz
RUN curl -fSL -o /out/usr/bin/kubelet https://dl.k8s.io/${kubernetes_version}/bin/linux/amd64/kubelet && chmod 0755 /out/usr/bin/kubelet
RUN curl -fSL -o /out/usr/bin/kubeadm https://dl.k8s.io/${kubernetes_version}/bin/linux/amd64/kubeadm && chmod 0755 /out/usr/bin/kubeadm
RUN curl -fSL -o /out/usr/bin/kubectl https://dl.k8s.io/${kubernetes_version}/bin/linux/amd64/kubectl && chmod 0755 /out/usr/bin/kubectl

ADD kubelet.sh /out/usr/bin/kubelet.sh
ADD kubeadm-init.sh /kubeadm-init.sh
RUN sed -e "s/@KUBERNETES_VERSION@/${kubernetes_version}/g" </kubeadm-init.sh >/out/usr/bin/kubeadm-init.sh && chmod +x /out/usr/bin/kubeadm-init.sh

FROM scratch
WORKDIR /
ENTRYPOINT ["/usr/bin/kubelet.sh"]
COPY --from=build /out /
ENV KUBECONFIG "/etc/kubernetes/admin.conf"
LABEL org.mobyproject.config='{"binds": ["/dev:/dev", "/etc/resolv.conf:/etc/resolv.conf", "/run:/run:rshared,rbind", "/var:/var:rshared,rbind", "/var/lib/kubeadm:/etc/kubernetes", "/etc/kubelet.conf:/etc/kubelet.conf", "/etc/kubeadm:/etc/kubeadm"], "mounts": [{"type": "cgroup", "options": ["rw","nosuid","noexec","nodev","relatime"]}], "capabilities": ["all"], "rootfsPropagation": "shared", "pid": "host", "runtime": {"mkdir": ["/var/lib/kubeadm", "/var/lib/cni/etc", "/var/lib/cni/opt"], "mounts": [{"type": "bind", "source": "/var/lib/cni/opt", "destination": "/opt/cni", "options": ["rw", "bind"]}, {"type": "bind", "source": "/var/lib/cni/etc", "destination": "/etc/cni", "options": ["rw", "bind"]}]}}'

# Public Key Directory for Image Verification

Place your public key (`cosign.pub`) generated via `cosign generate-key-pair` into this directory:

Path: `/etc/pki/containers/cosign.pub`

This allows `rpm-ostree` / `bootc` and Podman to verify signatures on signed container images deployed with `ostree-image-signed:docker://ghcr.io/<YOUR_GITHUB_USER>/bazzite-bifrost:latest`.

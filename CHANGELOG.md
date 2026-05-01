# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] - 2025-05-01

### Added
- Initial release of sp-engine
- Multistage Dockerfile with `slim`, `final` and `dev` targets
- Dockerfile.uv – faster builds using uv instead of pip
- Requirements split across `base`, `netconf`, `restconf`, `ai`, `dev`, `testing`, `full`
- Ansible collections for IOS, IOS-XR, NX-OS, JunOS via `collections.yml`
- GitHub Actions workflow – automated build & push to Docker Hub
- MIT License
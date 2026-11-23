# vm-nvidia — the NVIDIA GPU **passthrough VM** base box

The guest contract, and the VM sibling of
[opencharly/pod-cdi](https://github.com/opencharly/pod-cdi).

## Why this is a different image from pod-cdi

A VM with PCI passthrough **owns its kernel**. The guest loads its own NVIDIA module, so it
must carry its own driver user-space: the libraries, the Vulkan ICD files, and the container
toolkit that configures them. Nothing is injected from the host — the host's driver serves
the host, not the guest.

A **pod** is the exact inverse: the kernel module is host-global, so the host injects the
driver user-space per container through the CDI spec. A container image that bakes a driver
user-space shadows the host's and is wrong.

| | vm-nvidia (this repo) | pod-cdi |
|---|---|---|
| substrate | VM with GPU passthrough | pod / container |
| driver source | **baked into the image** | host, injected via CDI |
| layer | `layer-nvidia` | `layer-cdi` |
| `nvidia-ctk` in image | **yes** — nothing else configures the guest | no |
| `nvidia-utils` / `LD_LIBRARY_PATH` | **yes** | no |

## Composition

```yaml
vm-nvidia:
  candy:
    base: cachyos.cachyos
    candy:
      - '@github.com/opencharly/layer-agent-forwarding:v2026.274.0608'
      # ORDER IS THE CONTRACT: layer-cuda's strip runs FIRST, then the guest driver installs.
      - '@github.com/opencharly/layer-cuda:v2026.326.0028'
      - '@github.com/opencharly/layer-nvidia:v2026.326.0913'
```

`layer-nvidia`'s build checks assert `/usr/bin/nvidia-ctk` exists. That assertion is
**correct here** and was the source of a long confusion when the same layer backed pod
images: see opencharly/pod-cdi for the container-side contract.

## Never compose `layer-cdi` here

A VM that does not bake its driver has no driver. `layer-cdi` asserts that no driver
user-space is present — the opposite of what a passthrough guest needs.

# AGENTS.md — vm-nvidia

The org rulebook at the umbrella (`opencharly/opencharly` `AGENTS.md`) applies in full.
This file adds only what is specific to this repo.

## This repo is the VM side of a two-repo boundary

- **`vm-nvidia` (here)** — the boxes it contains are VMs with GPU passthrough. The driver
  user-space is baked into the image because the guest owns its kernel module. Composes
  `layer-nvidia`.
- **`pod-cdi`** — the boxes it contains are pods. The driver comes from the host via CDI,
  and its image must bake no driver user-space. Composes `layer-cdi`.

Never compose `layer-cdi` here: a VM that does not bake its driver has no driver.
`layer-nvidia`'s checks asserting `/usr/bin/nvidia-ctk` are correct in this repo and wrong
in any container image — do not "fix" them by removing them; use `pod-cdi` for pods.

## Pins

Pin a **tag** — a merged ref. A branch or SHA pin does not resolve for `@github` candy
refs (opencharly/charly#1201).

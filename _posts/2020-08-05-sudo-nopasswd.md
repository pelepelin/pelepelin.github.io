---
layout: post
title: Sudo without a password prompt
---

I often install new Linux virtual machines where I'm the only interactive user and so I see no sense in entering password every several minutes. Unfortunately, `sudoers` syntax is not at all intuitive, so every time I need to google for the same lines to be added.

- Run `sudo visudo`.
- Update the `%sudo` line to be:
  ```
  %sudo   ALL=(ALL:ALL) NOPASSWD: ALL
  ```

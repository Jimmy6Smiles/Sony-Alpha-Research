# References and attribution

This research builds on earlier public work. Results for those projects' target cameras are not treated as hardware acceptance for ILCE-1M2.

- [Sony-PMCA-RE — ma1co](https://github.com/ma1co/Sony-PMCA-RE): public Sony service/protocol research used as a reference for local authentication and transport analysis.
- [fwtool.py — ma1co](https://github.com/ma1co/fwtool.py): Sony firmware-container analysis tooling.
- [Sony-firmware-unpacker — q19032](https://github.com/q19032/Sony-firmware-unpacker): complementary unpacking implementation used during this firmware investigation.
- [Sony-ZVE-10-Camtool — nolan2628](https://github.com/nolan2628/Sony-ZVE-10-Camtool): service/Jiritsu research on another camera; useful context, not an ILCE-1M2 installer.
- [a7c2-service-preload — YuuLuo](https://github.com/YuuLuo/a7c2-service-preload): related service/preload research referenced during initial investigation.
- [Magic Lantern](https://www.magiclantern.fm/): architectural inspiration for separating a small startup mechanism from card-hosted extensions. Its Canon-specific boot mechanism is not evidence of a Sony-compatible boot-card format.

Firmware source context: [Sony firmware support page used for the analyzed package](https://service.sony.com.cn/download/firm/00003133.html). Package identity is recorded separately in [firmware-identities.json](../evidence/firmware-identities.json).

The ILCE-1M2-specific observations, validation boundaries and failure summaries in this repository come from the experiments described here. Links acknowledge upstream work without distributing its code or implying endorsement. Sony firmware, marks, and upstream projects retain their respective ownership and license terms.
